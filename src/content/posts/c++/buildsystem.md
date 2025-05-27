---
title: how to build a c++ system
published: 2025-05-26
description: 'how to build c++ system'
image: ''
tags: [cplusplus, make, cmake]
category: 'cplusplus'
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

[极佳的cmake 入门教程](https://cliutils.gitlab.io/modern-cmake/README.html)

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

> [!TIP]
> 如何运行某个cmake project `cmake -S . -B build`
> `cmake --build build -j 12`

### 如何设置变量
```shell
set(FOO "foo" CACHE STRING "Description")
message(STATUS "Value of Foo: ${FOO}")
```
上述命令会将 FOO 的值写入cmakecache.txt中

如何修改
+ 直接修改CMakeCache.txt 中的FOO 的取值
+ `cmake -S . -B build -D FOO="cmd_value"`

> [!NOTE] -D 参数改变的只是 cache string 中的FOO 取值，不可以改变内部(cache internal)和局部变量(local), local 会覆盖所有在cmakecache.txt 中相同名称的变量

如何直接将FOO 存入内部 而不是存入缓存中
```shell
set(FOO "foo" CACHE INTERNAL "Description")
```

如何设置局部变量
```shell
set(FOO "foo")
```
这样会覆盖所有的cache 中的`FOO`

### 如何设置cmake build type

```shell
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE INTERNAL "Build type")
endif()
message(STATUS "CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")
```


### 如何添加库
```shell
# 添加项目配置/ 只含头文件的库
add_library(cxx_setup INTERFACE)
# 添加其余库
add_library(tools tools.cpp)
```



### 如何添加运行库
```shell
add_executable(print_hello print_hello.cpp)
```

### 使用含有target 的命令
所有含有target 的命令都一定要指明传播性，interface 为所有子辈可见，public为当前与所有子辈可见，private 表示仅自己可见

1. 配置项目

```shell
add_library(cxx_setup INTERFACE)
target_compile_options(cxx_setup INTERFACE -Wall -Wpedantic -Wextra)
target_compile_features(cxx_setup INTERFACE cxx_std_17)
# 指明头文件位置
target_include_directories(cxx_setup INTERFACE ${PROJECT_SOURCE_DIR})
```
2. 连接库
```shell
add_executable(print_hello print_hello.cpp)
# 指明依赖库
target_link_libraries(print_hello PRIVATE tools cxx_setup)
```

3. 为库配置头文件与编译选项
```shell
add_library(tools tools.cpp)
target_link_libraries(tools PUBLIC cxx_setup)
```

4. google test

+ 开启测试选项
```shell
# Enable testing for this project
include(CTest)
# Add subdirectories with code
add_subdirectory(external)
```

+ 在测试文件夹中注册
```shell
# BUILD_TESTING variable is created by include(CTest)
# It is set to ON by default
if (BUILD_TESTING)
    add_executable(my_test my_test.cpp)
    target_link_libraries(my_test PRIVATE GTest::gtest_main)

    include(GoogleTest)
    # Finds all the Google tests associated with the executable
    gtest_discover_tests(my_test)
endif()
```

+ 使用googletest
```shell
// Must include the gtest header to use the testing library
#include <gtest/gtest.h>

namespace {
  // We will test this dummy function but you can test
  // any function from any library that you write too.
  int GetMeaningOfLife() {  return 42; }
}

// All tests must live within TEST* blocks
// Inside of the TEST block is a standard C++ scope
// TestTopic defines a topic of our test, e.g. NameOfFunctionTest
// TrivialEquality represents the name of this particular test
// It should be descriptive and readable to the user
// TEST is a macro, i.e., preprocessor replaces it with some code
TEST(TestTopic, TrivialEquality) {
  // We can test for equality, inequality etc.
  // If the equality does not hold, the test fails.
  // EXPECT_* are macros, i.e., also replaced by the preprocessor.
  EXPECT_EQ(GetMeaningOfLife(), 42);
}

TEST(TestTopic, MoreEqualityTests) {
  // ASSERT_* is similar to EXPECT_* but stops the execution
  // of the test if fails.
  // EXPECT_* continues execution on failure too.
  ASSERT_EQ(GetMeaningOfLife(), 0) << "Oh no, a mistake!";
  EXPECT_FLOAT_EQ(23.23F, 23.23F);
}
```

+ run google test
```shell
cmake -S . -B build
cmake --build build -j 12
GTEST_COLOR=1 ctest --test-dir build --output-on-failure -j12
# Or if you have an older CMake version (<3.20)
cd build && GTEST_COLOR=1 ctest --output-on-failure -j12
```

+ 如何动态维护google test
```shell
cd my_project
git submodule add https://github.com/google/googletest.git external/googletest
git submodule update --init --recursive
```

+ 自动更新脚本 cmake/updateSubmodule.cmake
```shell
# Adapted from https://cliutils.gitlab.io/modern-cmake/chapters/projects/submodule.html
find_package(Git QUIET)
if(GIT_FOUND)
    option(UPDATE_SUBMODULES "Check submodules during build" ON)
    if(NOT UPDATE_SUBMODULES)
        return()
    endif()
    execute_process(COMMAND ${GIT_EXECUTABLE} submodule
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                    OUTPUT_VARIABLE EXISTING_SUBMODULES
                    RESULT_VARIABLE RETURN_CODE
                    OUTPUT_STRIP_TRAILING_WHITESPACE)
    message(STATUS "Updating git submodules:\n${EXISTING_SUBMODULES}")
    execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                    RESULT_VARIABLE RETURN_CODE)
    if(NOT RETURN_CODE EQUAL "0")
        message(WARNING "Cannot update submodules. Git command failed with ${RETURN_CODE}")
        return()
    endif()
    message(STATUS "Git submodules updated successfully")
endif()
```

---