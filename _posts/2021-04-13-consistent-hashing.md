---
layout: post
title:  一致性哈希
categories: 数据结构与算法
description: 一致性哈希
keywords: hash
---

如何将一个与进程相关的id映射到进程上：

1. 进程之间移动文件描述符；<移动后的句柄值也许会变化>如果进程改变了，调用 unix domain sokcer（unix域套接字），可以将文件描述符发送给其他进程
2. 哈希表( bikeid % N(pids) )单车连接到哪个进程是不好控制的如果进程挂掉了，或者进程数量变化了tcp的动态均衡方案
3. 一致性哈希负载均衡式的，需要足够离散命中比较固定