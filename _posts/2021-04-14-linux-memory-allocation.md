---
layout: post
title:  linux内存分配策略与内存池
categories: c++
description: linux内存分配策略与内存池
keywords: c++, memory
typora-copy-images-to: ..\images\posts\c++
---

### linux地址空间

![1618403561046](..\images\posts\c++\1618403561046.png)

以应用最广的80X86体系结构来说，内存地址分为三种:

- 逻辑地址(Logical Address)也称相对地址
  - 包含在机器语言指令中用来指定一个操作数或一条指令的地址
  - 编译器编译程序生成代码段和数据段，代码段与数据段每条数据都会有自己的逻辑地址
  - 程序内部使用的都是逻辑地址(段内偏移量)
- 线性地址(Linear Address)也称虚拟地址
  - 是逻辑地址到物理地址变换之间的中间层
  - 在分段部件中逻辑地址是段中的偏移地址，然后加上基地址就是线性地址
  - 是一个32位无符号整数，可以用来表示高达4GB的地址，线性地址通常用十六进制数字表示，值得范围从0x00000000到0xfffffff）程序代码会产生逻辑地址，通过逻辑地址变换就可以生成一个线性地址
  - 如果启用了分页机制，那么线性地址可以再经过变换以产生一个物理地址。如果没有启用分页机制，那么线性地址直接就是物理地址；段基址+逻辑地址=线性地址
- 物理地址(Physical Address)
  - CPU地址总线传来的地址，由硬件电路控制（现在这些硬件是可编程的了）其具体含义。物理地址中很大一部分是留给内存条中的内存的，但也常被映射到其他存储器上（如显存、BIOS等）。在没有使用虚拟存储器的机器上，虚拟地址被直接送到内存总线上，使具有相同地址的物理存储器被读写；而在使用了虚拟存储器的情况下，虚拟地址不是被直接送到内存地址总线上，而是送到存储器管理单元MMU，把虚拟地址映射为物理地址

由于线性地址是连续，内存中可能没有这么大的一块连续空间。为了解决这个问题，CPU采用了分页的内存管理机制，每页4KB；通过分页机制，将线性地址转换为物理地址;

程序第一次分配的内存是虚拟内存，没有分配物理内存。第一次访问已分配的虚拟地址空间的时候，发生缺页中断，操作系统负责分配物理内存，然后建立虚拟内存和物理内存之间的映射关系

------

### 系统调用函数

malloc与free内部本质是调用一下一些列系统调用，用于用户态与内核态交互；其中brk用于分配小于128k内存，mmap用于分配大于128k内存；可通过 mallopt(M_MMAP_THRESHOLD, ) 来修改这个临界值

#### 内存分配原理

![1618404237503](..\images\posts\c++\1618404237503.png)

-  brk：将数据段(.data)的最高地址指针 _edata往高地址推(释放内存就是往低地址移动)
-  mmap：在进程的虚拟地址空间中(堆和栈中间，称为文件映射区域的地方)，找一块空闲的虚拟内存

brk和mmap的问题：

- brk和mmap都是系统调用，每次的调用的成本很高
- 释放内存时直接归还给了操作系统，实际是可以重新利用的
- brk最近申请的内存没有释放的话，之前申请的内存是无法释放的，内存碎片的产生的根源就是因为这



标准C库中，提供了malloc /free 函数分配释放内存，这两个函数底层是由 brk，mmap，munmap这些系统调用实现的

- malloc**小于128k**的内存，使用brk分配内存，将_edata往高地址推(只分配虚拟空间，不对应物理内存(因此没有初始化)，第一次读/写数据时，引起内核缺页中断，内核才分配对应的物理内存，然后虚拟地址空间建立映射关系)
- malloc**大于128k**的内存，使用mmap分配内存，在堆和栈之间找一块空闲内存分配(对应独立内存，而且初始化为0)


------



brk与sbrk不会直接释放内存，因为只有一个指针，而是在发现最高地址空闲内存超过128K，才会进行内存紧缩；堆内内存碎片不会直接释放，会导致疑似"内存泄露"问题，但是mmap分配的内存可以会通过munmap进行free ，会真正释放，但是是有缺点的：

- 进程向 OS 申请和释放地址空间的接口 sbrk/mmap/munmap 都是系统调用，频繁调用系统调用都比较消耗系统资源的
- mmap 申请的内存被 munmap 后，重新申请会产生更多的缺页中断。例如使用 mmap 分配 1M 空间，第一次调用产生了大量缺页中断 (1M/4K 次 ) ，当munmap 后再次分配 1M 空间，会再次产生大量缺页中断。缺页中断是内核行为，会导致内核态CPU消耗较大
- 如果使用 mmap 分配小内存，会导致地址空间的分片更多，内核的管理负担更大。同时堆是一个连续空间，并且堆内碎片由于没有归还 OS ，如果可重用碎片，再次访问该内存很可能不需产生任何系统调用和缺页中断，这将大大降低 CPU 的消耗

所以无法直接用mmap进行内存的分配，下面分别介绍内存系统调用函数

#### brk与sbrk

根据man手册查看函数原型：map brk

```cpp
#include <unistd.h>
int brk(void *addr);
void *sbrk(intptr_t increment);
```

brk和sbrk会改变 program break的位置(heap的结束位置)，当heap为空时，sbrk(0)即为堆初始地址；

- brk通过传递的addr来重新设置program break，成功则返回0，否则返回-1；brk指向高地址就是申请内存，指向低地址就是释放内存
- sbrk用来增加heap，增加的大小通过参数increment决定，返回增加大小前的heap的program break，如果increment为0则返回program break

在程序运行时，第一次分配的brk地址，也就是堆heap首地址是随机分配的，并不是BSS segment的结束地址，证明代码：

```cpp
#include <stdio.h>
#include <unistd.h>

int bss_end;
int main()
{
  printf("bss end: %p\n", &bss_end);
  void *tret = sbrk(0);
  if (tret != (void *)-1)
  {
    printf("heap start: %p\n", tret);
  }
  return 0;
}
```

每次heap start 输出都不相同，但是   bss end输出都是相同的

![1618404720557](..\images\posts\c++\1618404720557.png)

------



当使用malloc 和 free 函数分配和释放内存时，能提高程序性能，不是每次内存分配都调用brk或sbrk函数，而是重用前面空闲的内存空间；因为在malloc与free内部实现了类似线程池的结构，而不是直接调用系统调用

brk和sbrk分配的堆空间类似于缓冲池，每次malloc从缓冲池获得内存，如果缓冲池不够了，再调用brk或sbrk扩充缓冲池，直到达到缓冲池大小的上限，free则将应用程序使用的内存空间归还给缓冲池

测试malloc是否含有缓冲池，缓冲池大小扩充：

```cpp
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>


int main(void)
{
  void *tret;
  char *pmem;


  tret = sbrk(0);
  if (tret != (void *)-1)
          printf ("heap start: %p\n", tret);


  pmem = (char *)malloc(64);  //分配内存
  if (pmem == NULL) {
          perror("malloc");
          exit (EXIT_FAILURE);
  }
  printf ("pmem:%p\n", pmem);
  tret = sbrk(0);
  printf (" heap end:%p\n", tret);
  if (tret != (void *)-1)
          printf ("heap size on each load: %p\n", (char *)tret - pmem);
    free(pmem);
    return 0;
}
```

![1618405038608](..\images\posts\c++\1618405038608.png)

调用malloc(64)后，缓冲区大小从0变成 0x20ff0，改成malloc(1)结果一样，只要内存分配不超过0x20ff0大小，缓冲池都是默认扩充0x20ff0大小；说明调用malloc的时候会分配一块大内存作为缓冲池；

malloc一次分配的内存超过了0x20ff8，malloc不再从堆中分配空间，而是使用mmap()这个系统调用从映射区寻找可用的内存空间

------

brk性能探索：系统调用要从用户态到内核态切换效率低，计算进行0x5fffff次brk内存分配操作，耗时22s

```cpp
#include <stdio.h>
#include <unistd.h>
#include <time.h>


int main(void)
{
  clock_t time = clock();
  void *begin = sbrk(0);
  for ( unsigned int i = 0; i < 0x5fffff; ++i)
  {
    void *mem = sbrk(1024);
    int * p = (int*)mem;
    brk(begin);
  }
  printf("consume time:%Lf\n",(long double) (clock() - time)/1000/1000);
}
```

![1618405175012](..\images\posts\c++\1618405175012.png)

mmap是系统调用，效率也很低；所以尽量不要使用系统调用来分配内存；而是使用效率更高的malloc与free的c标准库函数

------

### glic

glibc通过管理break与mmap申请的内存，提供了malloc和free的接口；相比于brk与mmap，在使用时会申请时一次性申请大块内存

内存块采用chunk管理，并且将大小相似的chunk用链表管理，一个链表被称为一个bin，下面是chunk结构体

![1618405349619](..\images\posts\c++\1618405349619.png)

结构：

- 前一块内存大小
- 这块内存大小

  - 给一个地址，就能释放掉那块内存，因为glic的缘故，根据首地址能找到对应的这块内存的大小
  - 三个bit域
- 用户能使用的内存：User data start here下一块内存的大小


总共是128个bin，前64个bin里，相邻的bin内的chunk大小相差8字节，称为small bin，后面的是large bin，large bin里的chunk按先大小，再最近使用的顺序排列

![1618405419357](..\images\posts\c++\1618405419357.png)

- 128bin。每个bins都维护了大小相近的双向链表的chunk，chunk就是分割的内存块
- index表示的是每个trunk内可供用户使用的内存大小
- main area：主内存分配区，左边是小内存分配区


------

glibc 内存分配流程

![1618405474572](..\images\posts\c++\1618405474572.png)

- 获取分配区的锁，防止多线程冲突

- 计算出需要分配的内存的chunk实际大小

- 判断chunk的大小，如果小于max_fast（64b），则取fast bins上去查询是否有适合的chunk，如果有则分配结束

- chunk大小是否小于512B，如果是，则从small bins上去查找chunk，如果有合适的，则分配结束

- 继续从 unsorted bins上查找。如果unsorted bins上只有一个chunk并且大于待分配的chunk，则进行切割，并且剩余的chunk继续扔回unsorted bins；如果unsorted bins上有大小和待分配chunk相等的，则返回，并从unsorted bins删除；如果unsorted bins中的某一chunk大小 属于small bins的范围，则放入small bins的头部；如果unsorted bins中的某一chunk大小 属于large bins的范围，则找到合适的位置放入

- 从large bins中查找，找到链表头后，反向遍历此链表，直到找到第一个大小 大于待分配的chunk，然后进行切割，如果有余下的，则放入unsorted bin中去，分配则结束

- 如果搜索fast bins和bins都没有找到合适的chunk，那么就需要操作top chunk来进行分配了（top chunk相当于分配区的剩余内存空间）。判断top chunk大小是否满足所需chunk的大小，如果是，则从top chunk中分出一块来

- 如果top chunk也不能满足需求，则需要扩大top chunk。主分区上，如果分配的内存小于分配阀值（默认128k），则直接使用brk()分配一块内存；如果分配的内存大于分配阀值，则需要mmap来分配；非主分区上，则直接使用mmap来分配一块内存。通过mmap分配的内存，就会放入mmap chunk上，mmap chunk上的内存会直接回收给操作系统。

  ------

glic利用了内存池提高了内存的利用率，但是也有不足：

- 后分配的内存先释放,因为 ptmalloc 收缩内存是从 top chunk 开始,如果与 top chunk 相邻的 chunk 不能释放, top chunk 以下的 chunk 都无法释放。与相邻的trunk内存达到128k就会释放(类似于brk的释放策略)
- 多线程锁开销大， 需要避免多线程频繁分配释放(tcmalloc有进行优化，开辟了线程内存)
- 内存从thread的分配区中分配， 内存不能从一个分配区移动到另一个分配区， 就是说如果多线程使用内存不均衡，容易导致内存的浪费。 比如说线程1使用了300M内存，完成任务后glibc没有释放给操作系统，线程2开始创建了一个新的arena， 但是线程1的300M却不能用了，既不能释放，也不能给其他线程使用
- 每个chunk至少8字节的开销很大
- 不定期分配长生命周期的内存容易造成内存碎片，不利于回收。 64位系统最好分配32M以上内存，这是使用mmap的阈值


------

### tcmalloc内存池

解决glic的以上弊端

tcmalloc在内存分配上，对于小内存分配

![1618405726766](..\images\posts\c++\1618405726766.png)

- 不同尺寸的内存申请，使用freelist的方式来管理；与glic方式类似根据内存大小，设置内存链表
- 把内存的尺寸分成了很多种类别(class)，每一种类别用一个链表管理了多个内存块。这些尺寸以8byte进行递增（1, 2, 4, 6, 8为倍数），如果不足8byte就进行向上“取整”，比如如果分配7个字节则会分配8字节，如果是52字节则会分配64字节


对于大内存分配：


![1618405756287](..\images\posts\c++\1618405756287.png)  

- 引入了span的概念，span也就是组织了多个连续的内存页(4k)，记录起始页和页的数量

![1618405767897](..\images\posts\c++\1618405767897.png)

- span的结构
- 1Page 就是4k大小，然后在1Page中找到其中一个span，在span中也存在内存链表，4字节，8字节，12字节... 找到合适的内存块

![1618405789733](..\images\posts\c++\1618405789733.png)

------

全局分配和线程分配：

![1618405803970](..\images\posts\c++\1618405803970.png)

内存申请:

- 每个线程都一个线程局部的 ThreadCache，按照不同的规格，维护了对象的链表（因为这个ThreadCache只为这个线程服务，所以在分配内存时不用加锁
- 如果ThreadCache 的对象不够了，就从 CentralCache 进行批量分配(分配时需要加锁，因为时多线程共享的内存区域)
- 如果 CentralCache 依然没有，就从PageHeap申请Span
- 如果 PageHeap没有合适的 Page，就只能从操作系统申请了

内存释放：

- 在释放内存的时候，ThreadCache依然遵循批量释放的策略，对象积累到一定程度就释放给 CentralCache
- CentralCache发现一个 Span的内存完全释放了，就可以把这个 Span 归还给 PageHeap
- PageHeap发现一批连续的Page都释放了，就可以归还给操作系统

------

### 内存池

采用谷歌的tcmalloc或是jemalloc也有缺陷

回顾内存分配区别：

- glibc一次性问内核申请一大块内存，然后划分成很多的小块，这样用户需要使用内存的时候，就直接找到合适的内存给用户，如果释放内存的时候，也是直接由glibc去回收，这样就避免频繁得调用系统调用问内核要内存
- tcmalloc，它按照线程划分内存块，解决多线程之间对内存申请和释放的竞争问题，提高了效率
- tamalloc为了要满足各种尺寸申请和释放，所以算法复杂，所以这里就有了内存池技术的必要性

glibc，tcmalloc或者jemalloc 内存分配器缺陷：

- 它为了要满足各种尺寸申请和释放，所以算法复杂
- 但实际大部分的程序需要一种常规化的内存分配算法(结构体类型可能就几种，不需要支撑各种各样的内存大小) 
- 这样的设计应用在实际问题时并不是一个高效的算法
- 复杂的算法性能没有用户态的内存池高

内存池更需要一种：

- 太通用化的设计，实际上是所有方式都平庸；太特殊化的设计， 也会减弱对其他情况满足的情况；任何一个算法很难普遍化的适用任何一个工程，还不损失性能
-  需要根据业务的特点，设计独有的内存分配器(池)
- .工程的角度，可以防止程序因为内存错误（不当使用）而造成的崩溃或者其它各种各样的问题

------

#### 如何设计内存池

满足条件：

- 内存空间的大小是否可以调整
  - 根据产品特点，指定1024是使空间换时间
- 是否支持线程安全

假设一种最简单的内存池实现方案，服务器消息结构体不会超过1k字节，对消息管理的时候，设计环形队列，一次性申请一大块内存，不是通过malloc申请的，而是直接申请一大块的buff数组，char megcache[1024*1024] = {0}，分成1024块的环形队列；在网络消息传递时，不使用new而是创建char[]，进行内存拷贝，在栈上创建防止内存碎片产生

------

参考互联网各种有名的内存池设计方案：

#### boost::pool的设计



![1618406667161](..\images\posts\c++\1618406667161.png)![1618406663755](..\images\posts\c++\1618406663755.png)

- block：内存区，将内存区划分为多个chunk，多个chunk用链表串起来，可以动态增长
- chunk：大小相同
- 内存释放的时候，会放在block链表的最后，后面都是空闲的，所有申请的时候效率很高

------

#### ZeroMQ内存池

内存池特点：

- 尽量少的申请内存，尽量使用即将释放的内存
- 一个读线程，一个写线程，读写之间使用无锁解决互斥
- 批量写入，预取机制，提高效率

内存队列(内存池)：yqueue_t

```cpp
//Individual memory chunk to hold N elements 存储N个元素的单个内存块
struct chunk_t
{
    T values [N];
    chunk_t *prev;
    chunk_t *next;
}
```

这个结构体存储了一个双向列表，结构如下：

![1618406761919](..\images\posts\c++\1618406761919.png)

```cpp
//如果队列为空，Back position可能指向无效的内存
//begin和end位置总是有效的。Begin位置被队列读取器独占访问(front/pop)，而back和end位置被队列写入器独占访问(back/push)
 chunk_t *begin_chunk; //控制读指针
int begin_pos;  //真正读的位置
chunk_t *back_chunk; //控制写指针
int back_pos; //真正写的位置
chunk_t *end_chunk; //控制申请内存指针
int end_pos;   	

atomic_ptr_t<chunk_t> spare_chunk; //原子操作；内存不够时判断 spare_chunk 是否为空，如果不是空的就链接到end_chunk，并将 spare_chunk 置空，如果不够，继续向操作系统申请；当链表begin_thunk读到next_chunk的时候，会将内存交给spare_chunk，并且begin_chunk指向 next结点；如果第二个chunk也释放掉的时候，那么第一次释放的chunk会真正归还操作系统；存在结构体里的数据只能读一次，每读一次，指针会移动一次
```

对yqueue_t 继续封装：ypip_t

- 支持批量写入，预取机制，提高效率

```cpp
//  Allocation-efficient queue to store pipe items.
//  Front of the queue points to the first prefetched item, back of
//  the pipe points to last un-flushed item. Front is used only by
//  reader thread, while back is used only by writer thread.
yqueue_t <T, N> queue;   /* 内存池 */
//  Points to the first un-flushed item. This variable is used exclusively by writer thread.
T *w; //控制写
//  Points to the first un-prefetched item. This variable is used exclusively by reader thread.
T *r; //控制读
//  Points to the first item to be flushed in the future.
T *f; // flush
//  The single point of contention between writer and reader thread.
//  Points past the last flushed item. If it is NULL, reader is asleep.
//  This pointer should be always accessed using atomic operations.
atomic_ptr_t <T> c; //原子操作指针
```

Ypipe_t的初始化：

```cpp
inline ypipe_t ()
{
    //  Insert terminator element into the queue.
    queue.push ();
    //  Let all the pointers to point to the terminator.
    //  (unless pipe is dead, in which case c is set to NULL).
    r = w = f = &queue.back ();
    c.set (&queue.back ());
}
```

![1618406841749](..\images\posts\c++\1618406841749.png)

Ypipe_t的预读机制：

- 被一个线程写，被另一个线程读
- 通过f指针移动通知最后写好的位置

```cpp
inline bool read (T *value_)
{
    //  Try to prefetch a value.
    if (!check_read ())
        return false;
    //  There was at least one value prefetched.
    //  Return it to the caller.
    *value_ = queue.front ();
    queue.pop ();
    return true;
}
```

check_read()函数：

```
inline bool check_read ()
{
    //  Was the value prefetched already? If so, return.
    if (&queue.front () != r && r)
         return true;
    r = c.cas (&queue.front (), NULL);
    if (&queue.front () == r || !r)
        return false;
    return true;
}
```

![1618406875393](..\images\posts\c++\1618406875393.png)