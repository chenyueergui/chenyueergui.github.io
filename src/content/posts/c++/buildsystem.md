---
title: how to build a c++ system
published: 2025-05-26
description: 'how to build c++ system'
image: ''
tags: [c++, make, cmake]
category: 'c++'
draft: false 
lang: 'zh_CN'
---

# 如何使用工具正确构建多C++文件


## 直接使用g++/clang/c++命令


多文件c++ 的结构如下所示

```markdown
foo.cpp bar.cpp main.cpp print.hpp
```

头文件中只放入函数声明(`function declaration`)，源文件放入函数定义(`function definition`)

> [!NOTE] 
> 头文件中要使用`#pragma once` 来确保头文件只会被包含一次

实现步骤
+ compile correspond source files to object files
+ Bind the object files into static/dynamic libraries
+ Link the libraries to executables

具体命令

> Compile correspond source files to object files

```shell
c++ -std=c++17 -c foo.cpp -I . -o foo_static.o
c++ -std=c++17 -c -fPIC foo.cpp -I . -o foo_dynamic.o
```

> [!TIP] 
> `-I` 指定的是头文件的位置在哪里

> Pack objects into libraries

```shell
# static 
ar rcs libfoo.a foo_static.o
# dynamic 
c++ --shared foo_dynamic.o -o libfoo.so
```

> link the libraries to our binary

```shell
c++ -std=c++17 main.cpp -L . -I . -lfoo -o main
```

> [!TIP]
> `-L`指定的是folder to the library search path
> `-lfoo` Link to the library file libfoo.a or libfoo.so

> [!WARNING]
> `-l` must be after all the `.cpp` and `.o` files

## 使用make 

c++ 多文件结构
```
main.cpp, blah.cpp, blah.hpp
```

make 的简单结构
```
target: dependencies
    command
    command
    command 
    command
```

[非常棒的Makefile 教程](https://makefiletutorial.com/#getting-started)

对于这部分的c++多文件结构 下面是对应的Makefile
```shell
all: main
main: main.cpp libblah.a
    c++ -std=c++17 main.cpp -I . -L . -lblah -o main
libblah.a: blah.o
    ar rcs libblah.a blah.o
blah.o: blah.cpp
    c++ -std=c++17 -c blah.cpp -I . -o blah.o
clean: 
    rm -f blah.o libblah.a main
```

## 使用cmake (recommended)

cmake 本身`不是构建系统` 是构建生成系统 为Make 等其他构建系统提供支持
cmake 主要的功能有
+ 提供对于宿主机的支持，你不需要考虑你的电脑到底是windows macos 还是 linux
+ 配置变量for your build
+ 创建正确的编译命令(如指明依赖， 确定正确的编译flags)

### minimal cmake project
CMakeLists.txt
```shell
cmake_minimum_required(VERSION  3.16..3.24)
project(our_project 
    VERSION 0.0.1
    DESCRIPTION "out first project"
    LANGUAGES CXX
        )
```

### 如何设置变量
```shell
set(FOO "foo" CACHE STRING "Description")
message(STATUS "Value of Foo: ${FOO}")
```
上述命令会将 FOO 的值写入cmakecache.txt中

如何修改
+ 直接修改CMakeCache.txt 中的FOO 的取值
+ `cmake -S . -B build -D FOO="cmd_value"`

> [!NOTE] -D 参数改变的只是 cache 中的FOO 取值，不可以改变内部和局部变量

如何直接将FOO 存入内部 而不是存入缓存中
```shell
set(FOO "foo" CACHE INTERNAL "Description")
```

如何设置局部变量
```shell
set(FOO "foo")
```

这样会覆盖所有的cache 中的`FOO`

c++ 文件结构




---