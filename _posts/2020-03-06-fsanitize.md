---
layout: post
title: fsanitize
categories: 服务端
description: 内存泄露排查
keywords: fsanitize
---

## 内存检测

本次的工作任务是：调查SceneServer内存持续上涨的原因；

目前已分析出如下结论

1. 已知装载了 “-fsanitize=address” ，这个参数需要额外添加libasan库才会正常运行；添加此参数，内存必增长，关闭次参数，内存不会增长
2. 编译器采用gcc；已调查gcc7.3，gcc8.3，gcc9.2都具有次问题，暂时排除gcc编译器的问题
3. 项目中使用的第三方库：centos-release-scl-rh.noarch；devtoolset-8；sol3；protobuf可能存在问题，正在排查
4. 尝试采用clang进行编译，sol最少支持clang 3.9x ； gcc 7.x版本;
5. cmake报错关于root的，是环境变量的问题，于是卸载了cmake 3.10，重新源码安装了个
6. 安装clang makeinstall完并没有装到bin下，需要符号连接 ln -s /opt/clang_install/llvm-build/bin/clang++ /usr/bin/clang++
7. clang编译失败 ==！
8. 继续尝试，首先不安装3.9，直接安装最新版clang ： 
9. 之前clang总是报错c++11一些库找不到，是因为头文件与库文件目录还是之前通过yum安装的老的

```
# 连接生成的文件放在是后面的路径
ln -s /usr/local/include/c++/9.1.0   /usr/include/c++/9.1.0
ln -s /usr/local/lib/gcc/x86_64-pc-linux-gnu/9.1.0  /usr/lib/gcc/x86_64-pc-linux-gnu/9.1.0
```

## 内存泄露分析

[https://nebula-graph.io/cn/posts/introduction-to-google-memory-detect-tool-addresssanitizer/](https://nebula-graph.io/cn/posts/introduction-to-google-memory-detect-tool-addresssanitizer/)

AddressSanitizer 的使用注意事项

1. AddressSanitizer 在发现内存访问违规时，应用程序并不会自动崩溃。这是由于在使用模糊测试工具时，它们通常都是通过检查返回码来检测这种错误。当然，我们也可以在模糊测试进行之前通过将环境变量 ASAN_OPTIONS 修改成如下形式来迫使软件崩溃：

export ASAN_OPTIONS='abort_on_error=1'/

2. AddressSanitizer 需要相当大的虚拟内存（大约 20 TB），不用担心，这个只是虚拟内存，你仍可以使用你的应用程序。但像 american fuzzy lop 这样的模糊测试工具就会对模糊化的软件使用内存进行限制，不过你仍可以通过禁用内存限制来解决该问题。唯一需要注意的就是，这会带来一些风险：测试样本可能会导致应用程序分配大量的内存进而导致系统不稳定或者其他应用程序崩溃。因此在进行一些重要的模糊测试时，不要去尝试在同一个系统上禁用内存限制。