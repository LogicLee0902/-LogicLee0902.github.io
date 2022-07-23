---
layout:     post
title:      "「CMakeList」How to write CMakeList.txt"
subtitle:  CMakeList教程
date:       2022-05-17
author:     Leo
header-img: ""
catalog: true
tags: ["C/C++@Languages@Tags"]
lang: zh
katex: true
---

&emsp;这是一个要填好久的坑了，背景主要在于将C/C++的ide从Dev-C++的过渡到了CLion，然后发现了巨大的不同，特别是CLion中有关.c, .cpp, .h等各类文件的链接极度依赖于CMakeList，因此记录一下相关的知识点。

# CMakeList 学习


## 背景

&emsp;CMakeList.txt是CMake的指导说明书吧，CMake是跨平台编译工具，比make更高级一些。其编译的主要工作是生成CMakeLists.txt文件，然后根据该文件生成Makefile，最后调用make来生成可执行程序或者动态库。所以基本步骤就只有两步：（1）cmake生成CMakeLists.txt文件；（2）make执行编译工作。

而CLion中我们关注于如何编写CMakeList即可

## 基础配置

### 设置项目版本

可以通过project配置项目信息
`project(PROJECT_NAME VERSION 1.0.0 LANGUAGES C CXX)`

VERSION对应的版本号应该实际为`main.minor.patch.tweak`,方便在代码运行的时候了解当前版本号，，并结合configure命令可以得到版本头文件，但感觉CLion中对此方面没有做要求，可以省略跳过。

### 制定语言版本

可以用set进行定制，如下

```Cmake
set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 11)
```

就是说，在编译C代码的时候，使用的是C99标准，编译C++的时候使用C++11标准。

这里设置的变量都是CMAKE_开头，这类变量都是CMake的内置变量，正是通过修改这些变量的值来配置CMake构建的行为。
CMake的内置变量一般有三种

* `_CMAKE`
*  `_CMAKE`
*  下划线开头加上CMake命令的名称的变量名

### 配置编译选项

通过`add_compile_options`进行配置，其同时对多个编译器有用。 通过设置变量CMAKE_C_FLAGS可以配置c编译器的编译选项； 而设置变量CMAKE_CXX_FLAGS可配置针对c++编译器的编译选项。其中编译选项就是之前写在dev编译选项里面的选项，比如`-Wall`、`-Wextra`等等。O2优化也可以在里面配置

```Cmake
add_compile_options(-Wall -Wextra -pedantic -Werror)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pipe -std=c99")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pipe -std=c++11")
```

### 配置编译类型

`CMAKE_BUILD_TYPE`变量，也是那么几个：`Debug`, `Release`, `RelWithDebInfo`、`MinSizeRel`等

如果设置编译类型为Debug，那么对于c编译器，CMake会检查是否有针对此编译类型的编译选项CMAKE_C_FLAGS_DEBUG，如果有，则将它的配置内容加到CMAKE_C_FLAGS中。

可以针对不同的编译类型设置不同的编译选项，比如对于Debug版本，开启调试信息，不进行代码优化：

```CMake
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g -O0")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g -O0")
```

对于Release版本，不包含调试信息，O2优化：

```CMake
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O2")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2")
```

这样不同的版本会略有区别

>补充 ： 几种编译类型的说明
>* Debug：调试版本，包含调试信息，不进行代码优化，用于调试。
>* Release：发布版本，不包含调试信息，进行代码优化，用于发布。因此对于release版本增加断点是不会停止的

## 编译配置

必要的框架一般如下

```Cmake

project(xxx)                                          #必须

add_subdirectory(子文件夹名称)                         #父目录必须，子目录不必

add_library(库文件名称 STATIC 文件)                    #通常子目录(二选一)
add_executable(可执行文件名称 文件)                     #通常父目录(二选一)

include_directories(路径)                              #必须
link_directories(路径)                                 #必须

target_link_libraries(库文件名称/可执行文件名称 链接的库文件名称)       #必须

```

### 添加头文件文件夹

通过`include_directories`进行添加，可以添加多个文件夹，比如添加`include`文件夹，`include/`文件夹下的头文件，这样就可以在CMake中使用头文件了。

```Cmake
include_directories(include)
```

用作例子的项目[cmake-template](https://gitee.com/RealCoolEngineer/cmake-template)，结构如下

![20220718161503](https://s2.loli.net/2022/07/18/suDqr3VUTHKXkwe.png)

 
项目的构建任务为：

* 将math目录编译成静态库，命名为math
* 编译main.c为可执行文件demo，依赖math静态库
* 编译test目录下的测试程序，可以通过命令执行所有的测试
* 支持通过命令将编译产物安装及打包

> .cc 是 c++在linux下的后缀名 等同于 .cpp

