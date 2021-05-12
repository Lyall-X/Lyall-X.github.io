---
layout: post
title: 性能优化
categories: 服务端
description: 性能优化
keywords: 性能
---

## Linux性能优化

### 平均负载：

平均活跃进程数，命令 uptime

* 可运行状态进程
* 不可中断状态进程

Pressure Stall Information：命令 head /proc/pressure/*

* 10s, 1m, 5m 硬件资源短缺百分比
* Kernel >= 4.20

---

### 内存性能优化

内存池

* 缺页异常
* 多线程锁
* 内存碎片

内存工作原理

* 虚拟内存空间
* malloc = brk + mmap

快速定位磁盘I/O性能问题

网络性能优化

linux支持多少并发连接

解决time_wait

------

## 性能优化步骤

1. 先看CPU是否满载，满载就优化CPU
2. 没有满载就找流控点，通过流控点调整时延和通量

### 分析流程

1. 调度没有充分展开，比如你只有一个线程，而你其实有16个核，这样就算其他核闲着，你也不能怎么样。这是需要想办法把业务hash展开到多个核上处理
2. 配套队列的线程有IO空洞，要通过异步设计把空洞填掉，或者通过在这个队列上使用多个线程把空洞修掉
3. 如果CPU占用率已经占满了，观察CPU的时间是否花在业务进程上，如果不是，分析产生这种问题的原因。Linux的perf工具常常可以提供良好的分析
4. 如果CPU的时间确实都已经在处理业务的，剩下的问题就是看CPU执行系统是否被充分利用

### 关于流控

* 流控应该出现在队列链的最前面，而不是在队列的中间。因为即使你在中间丢包了，前面几个队列已经浪费CPU时间在这些无效的包上了，这样的流控很低效。所以，中间的队列，只要可能，一般不设置自身的流控
* 在有流控的系统上，需要注意一种很常见的陷阱：就是队列过长，导致最新的包被排到很后才处理，等完成整个系统的处理的时候，这个包已经被请求方判断超时了。这种情况会导致大量的失效包。所以，流控的开始时间要首先于每个包的超时时间

------

## ftrace

### trace-cmd追踪进程堆栈

官网：[https://lwn.net/Articles/410200/](https://lwn.net/Articles/410200/)

知乎教程：[https://zhuanlan.zhihu.com/p/22130013](https://zhuanlan.zhihu.com/p/22130013)

[https://zhuanlan.zhihu.com/p/33267453](https://zhuanlan.zhihu.com/p/33267453)

---

安装使用：[https://ichrisking.github.io/2018/03/08/FlameGraph/](https://ichrisking.github.io/2018/03/08/FlameGraph/)

官网：[http://www.brendangregg.com/FlameGraphs/memoryflamegraphs.html](http://www.brendangregg.com/FlameGraphs/memoryflamegraphs.html)

### 火焰图分析gateServer性能

* 火焰图中的每一个方框是一个函数，方框的长度，代表了它的执行时间，所以越宽的函数，执行越久
* 火焰图的楼层每高一层，就是更深一级的函数被调用，最顶层的函数，是叶子函数

perf record -e GateServer:malloc -F 99 -p 31361 -g -o in-fb.data -- sleep 60

perf record -F 99 -p 30646 -g -o in-fb.data -- sleep 60

perf record -e kmem:kmalloc -p 5859 -g -o in-fb.data -- sleep 60

### 升级内核：bcc性能工具

使用文档：[https://iblogs.top/2019/09/18/bcc%E5%B7%A5%E5%85%B7%E5%8C%85%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8/](https://iblogs.top/2019/09/18/bcc%E5%B7%A5%E5%85%B7%E5%8C%85%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8/)

pprof使用：

[https://blog.csdn.net/weixin_41644391/article/details/81193518](https://blog.csdn.net/weixin_41644391/article/details/81193518)