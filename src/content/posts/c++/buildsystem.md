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

未完待续...
---