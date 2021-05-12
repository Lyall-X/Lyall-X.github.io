---
layout: post
title: 网络攻防
categories: 运维
description: 网络攻防
keywords: 网络攻防，安全
---

## 网络攻防简介

学习目标：安全防护；渗透攻击；企业中真实项目, 深度报文检测

---

负载均衡：

* 软件负载均衡：nginx
* 硬件负载均衡：F5负载均衡器就是硬件网络性能优化设备
  * 同于交换机、路由器这些网络基础设备，而是建立在现有网络结构上用来增加网络带宽和吞吐量的的硬件设备

/etc/services：包含了服务名和端口号之间的映射，系统的端口号的范围为0–65535

* 0 不使用
* 1--1023 系统保留，只能由root用户使用
* 1024---4999 由客户端程序自由分配
* 5000---65535 由服务器端程序自由分配

网络攻防环境：

* 靶机：包含系统和应用程序漏洞，并作为攻击目标的主机，metasploitable，现在服务器漏洞少，所以用这个
* 攻击机：用于发起网络攻击的主机，Kali Linux
* 侵检测分析系统：
  * IDS: Intrusion Detection Systems, 入侵检测系统：部署在靶机
  * IPS: Intrusion Prevention System, 入侵防御系统：部署在网关

* 网络连接：交换机、网关等

信息安全需具备：

* 机密性：控制对数据的r的访问权限
* 完整性： 控制对数据的w的访问权限；哈希函数
* 可用性：保证系统能够对可授权用户的正常访问；磁盘raid，多网卡bind；dos攻击的是这个
* 可追溯性：提供审计，事后追踪

------

## Linux 基本安全防护技术

* 自主访问控制（许可位、ACL）
* 文件属性
* PAM技术
* 能力机制

---

命令补充：

* whoami：有效用户ID相关联的用户名
* id：* 查看用户的id；root用户id是0
* useradd：
  * -m ： 一起创建用户的家目录
  * -s： 指定用户的登录shell类型， -s /bin/bash
  * useradd -m -s /bin/bash itcast

* userdel：-r： 用户主目录以及用户主目录下文件一起删除
* passwd：修改用户口令
  * /etc/shadow：记录用户密码
  * /etc/passwd：记录用户相关信息

---

### 访问权限（许可位）

* 文件权限（访问许可位）
  * 属主、属组、“其他”

* 文件权限（许可位）表示方式
  * r、w、x、-来表示文件的访问权限
  * 对于普通文件来说
    * r: 就是读的权限
    * w: 就是修改文件的权限
    * x: 就是执行权限

  * 对于目录文件来说
    * r: 读取目录下的目录项,必须要先有x权限, 如ls命令
    * w: 在目录下创建文件, 删除文件, 重命名文件, 移动目录中的文件
    * x: 进入到目录的权限, 如cd ls

* 变更文件的访问权限：chmod
* 粘着位sticky

**Set UID**

当s这个标志出现在文件所有者的x权限上时，此时就被称为Set UID，简程SUID；

主要是为了临时获取所有者执行权限；例如passwd，su，sudo等命令

* SUID权限仅对
* 执行者对于该程序需要具有x的
* 本权限仅在执行该程序的过程中有效：在root下如果删除了su进程，则不具备suid权限；ps -ef | grep su 查看
* 执行者将具有该程序拥有者的权限

/etc/shadow, 同组用户和其他人是没有写权限的，但是passwd命令可以将密码写入shadow中；普通用户没有权限写入shadow但是可以修改密码，就是因为suid功能

这个SUID只能运行在二进制的程序上（系统中的一些命令），不能用在脚本上，同样也不能放到目录上，放上也是无效的

**Set GID**

s出现在gourp的x权限上，是SGID

* SGID对二进制程序有用
* 程序执行者对于该程序来说需具备x的权限
* SGID主要用在目录上

**Sticky Bit**

出现在other的x权限上；

SBIT（Sticky Bit）目前只针对目录有效，对于目录的作用是：当用户在该目录下建立文件或目录时，仅有自己与 root才有权力删除

最具有代表的就是/tmp目录，任何人都可以在/tmp内增加、修改文件（因为权限全是rwx），但仅有该文件/目录建立者与 root能够删除自己的目录或文件

SBIT对文件不起作用

**SUID/SGID/SBIT权限设置**

和rwx权限一样，s、t也有两种设置方法：

* 文字法 ：SUID: u+s ，SGID: g+s，SBIT: o+t
* 数字法：将原来的三位数扩展为四位数即可，SUID为4，SGID为2，SBIT为1，把它们放在权限数字的最开头。例如设置SUID，可以写成4777，设置SGID可以写成2777

---

**访问控制列表 -ACL**

ACL：存储在文件扩展属性中的一组访问控制规则， (利用文件的扩展属性保存额外的访问控制权限）

ACL：访问控制列表

* 设置: setfacl -m u:username:rwx filename
* 查看: getfacl filename
* 清除: setfacl -x u:username filename

在设置了ACL和mask之后, 某个用户对文件的访问权限顺序:

* 若文件的属主是该用户, 则按照文件属主的权限进行控制
* 若不是文件属主, 则看是不是同一个组的, 若是同一组的则按照组的权限进行控制
* 若不是同一组, 则看ACL和mask, 两个权限结合进行控制, 如: mask 为r, 而ACK为rw, 则最后的

  注意: mask表示对ACK权限做一个权限限制, ACK不能超过mask设置的权限.

    setfacl  -m m::r -m u:itcast:rwx ./file1 mask设置了r权限

---

### 文件属性

* 在特定的文件系统中支持的，对文件、文件夹等文件额外施加的一些访问控制
* ext4：第四代文件系统

![c9015d7177f32419988fd0ef87a8811b.png](\images\posts\operations\c9015d7177f32419988fd0ef87a8811b.png)

* chattr 命令；root执行
  * -R 递归地修改文件夹和子文件夹的属性
  * -V chattr命令会输出带有版本信息的冗余信息
  * -f 忽略大部分错误信息

* 属性符号：
  * '+' 符号用来为文件和文件夹设置属性，
  * '-' 符号用来移除或者取消属性
  * '=' 使它们成为文件有的唯一属性。

* chattr <options> <attributes> <file or Directory >
* lsattr <File or Directory>

例如：设置

* sudo chattr +i filename  ---->表示文件不能被删除, 不能被修改(包括root用户在内)
* sudo chattr +a filename  ---->表示文件不能够直接覆盖, 只能在末尾追加

查看：

* sudo lsattr filename
* sudo lsattr -l filename：查看具体设置含义

---

**PAM技术**

PAM：Pluggable Authentication Modules , 可插拔的鉴权模块， sun提出的一种鉴权机制

在/etc/pam.d文件夹下，可以看到调用模块

---

**特权（能力）机制**

能力是一种特权， 如果把root用户的超级特权分作N份， 每一份可以作为一个能力子集

------

## 网络嗅探及协议分析技术

网络嗅探：工具包括tcpdump（命令行）、wireshark（图形界面）

---

**tcpdump**

用于捕获网络报文，并输出报文内容的工具；命令行嗅探（抓包）工具。

