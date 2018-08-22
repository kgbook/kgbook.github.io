---
layout: post
title: gdb 调试时 "No such file or directory" 的问题
date: 2018-08-22
tags: [gdb]
---

## 问题场景 ##

该问题在交叉编译，板端运行调试时是个普遍的现象！但gdb 提示源文件找不到，可是我们编译调试版本的时候加了`-g -O0 -rdynamic`等选项了啊！

`list` 或 `next` 等命令不能正常使用，很影响程序的调试！

CMakeLists.txt 相关规则：
```cmake
project(koala)
set(KOALA_DEBUG 1)

if (KOALA_DEBUG)
    set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -g -O0 -rdynamic")
    set(CMAKE_VERBOSE_MAKEFILE ON)
else()
    set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -O2")
    add_definitions(-DNDEBUG)
endif ()
```

## 问题描述 ##

有如下断点信息,
```shell
Breakpoint 1, WpaSupplicant::start (this=0x299840) at /home/kgbook/koala/src/WpaSupplicant.cpp:117
```

当运行到该断点，运行 `list` 命令，
```shell
117     /home/kgbook/koala/src/WpaSupplicant.cpp: No such file or directory.
```

## 解决方案 ##
1. 拷贝该源码代码到编译路径，如板端不存在该路径，则 `mkdir -p` 创建该路径。
2. 修改 CMakeLists.txt 规则 或 Makefile 规则， 增加 `g++` 编译器选项 `-ggdb`，编译时才会生成 gdb 调试所需要的调试信息。

>-ggdb
Produce debugging information for use by GDB. This means to use the most expressive format available (DWARF 2, stabs, or the native format if neither of those are supported), including GDB extensions if at all possible. 

Makefile 修改如下：
```makefile
CXX_FLAGS += -ggdb
```

CMakeLists.txt 修改如下：
```
set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -ggdb")
```

注：
如果 编译器为 gcc 则修改 CFLAGS。

## 参考文献 ##
1. [Options for Debugging Your Program or GCC](https://gcc.gnu.org/onlinedocs/gcc-4.9.4/gcc/Debugging-Options.html#Debugging-Options)