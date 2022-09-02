---
layout:     post
title:      "Learn Compile Technology 01"
subtitle:  编译的初学习
date:       2022-0p8-31
author:     Leo
header-img: "img/post-compile.jpg"
catalog: true
tags: ["Compile@Tags@Tags", "编译技术@Courses@Series"]
lang: zh
katex: true
---

本文主要学习内容来源自[Let's Building A Simple Interpreter](https://ruslanspivak.com/lsbasi-part1/)

# Learning Compile Technology 01

很喜欢教程中的一句话

>***if you don’t know how compilers work, then you don’t know how computers work. If you’re not 100% sure whether you know how compilers work, then you don’t know how they work.***  —— Steve Yegge

## Definition of the compiler and interpreter

简单来说，解释器或者编器的目标就是为了将由高级语言写成的源程序转换成别的形式，而别的形式是可以使得机器理解并运行流转。其中，编译器和解释器的区别也在此诞生。当转换器机器语言时，其为编译器，当其可以不需要转换为机器语言而奖其执行时，解释器。

![image.png](https://pic7.58cdn.com.cn/nowater/webim/big/n_v27a4b1e23c7954e3ca6fc0b8fd1d9accb.png)