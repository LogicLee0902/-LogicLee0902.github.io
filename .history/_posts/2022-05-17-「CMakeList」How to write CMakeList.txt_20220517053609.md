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

## 基础配置