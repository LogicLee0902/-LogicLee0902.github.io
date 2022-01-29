---
layout:     post
title:      "「Head First Java」 01 Class & Obejct"
subtitle:   Java 基础语法
date:       2022-01-24
author:     Leo

header-img: img/post-java.jpg
catalog: true
tags: ["Java@Languages@Tags", "Head First Java@Books@Series"]
lang: zh
header-style: text
katex: true
---

> 鉴于之前学过一点，就跳过一些基础的java， javac的编译执行过程，以及一些基本的声明过程了
>
> 加上其与C/C++也有相似的地方，重点在一些Java特别的地方

# 补充一些可注意的事项

* jdk11后可以直接`java Program_Name.java` 代替先`java`再`javac`

* java在循环时不能像C/C++一样用大于0的数当作True，integer和boolean并不相同，要明确声明一个boolean变量

* 输出`print`与`printIn`的区别在于，`print`输出后的后续输出还会在同一行，而`printIn`输出后自动换行。

* 输入依靠命令行参数，**注意**：java命令并不是第一个参数

  e.g.:

  ```shell
  $ java Example -s check
  #arg[0] "-s"
  #arg[1] "check" 
  ```

# 类与对象

运用对象时一般需要两个类，一个是要被操作于对象的类，另一个用来测试该类的类（带有main()，并在期中创立对象）

```java
// 1. write class
class Dog {
 int size;   
 String breed;
 String name;
 void bark() {
     System.out.printIn("Ruff！Ruff!");
 }
}
// 2. write class for testing
class DogTestDrive {
    public static void main (String[] args) {
//3. form a new object and test the method 
        Dog d = new dog();
        d.size = 40;
        d.bark();
    }
}
```







