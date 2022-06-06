---
layout:     post
title:      "「Java multithreaded and design pattern 02」Immutable pattern"
subtitle:   不可变模式
date:       2022-04-10
author:     Leo
header-img: "img/post-multithread.jpg"
catalog: true
tags: ["Java@Languages@Tags", "图解Java多线程设计模式@Books@Series"]
lang: zh
katex: true
---

多个线程访问也不会出问题，可以方便提高性能

# 粗略介绍

---

比较明显的特征是，对于类的属性只有getter没有setter，任何属性只能在构造函数**初始化**时赋值。这样其中的getter就无需声明为`synchronized`

同时会有一些常用方法，但其实并不是Immutable的必要条件

1. 将共享的类型设置为final，这样无法创建此类的子类，便可以防止子类修改其字段值的一种措施
2. 将类的字段设置为private和final。这时字段一旦被赋值就不会再被修改

# 角色

---

## immutable

创建后字段就不改变
其优点在于，**不需要`synchronized`进行保护**

### 类图如下

![20220517135948](https://s2.loli.net/2022/05/17/WJtcsbdIrKDBVjh.png)


### 线程图
![20220418003702](https://s2.loli.net/2022/04/18/kiftAOZC6wd9oRS.png)

