---
layout:     post
title:      "「Java multithreaded and design pattern 02」Immutable pattern"
subtitle:   不可变模式
date:       2022-04-10
author:     Leo
header-img: ""
catalog: true
tags: ["Java@Languages@Tags", "图解Java多线程设计模式@Books@Series"]
lang: zh
header-style: text
katex: true
---

多个线程访问也不会出问题，可以方便提高性能

#  粗略介绍

---

比较明显的特征是，对于类的属性只有getter没有setter，任何属性只能在构造函数**初始化**时赋值。这样其中的getter就无需声明为`synchronized`
同时还有一
