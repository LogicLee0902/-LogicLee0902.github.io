---
layout: "post"
title: "「BUAA-OO-Lab」 Pre - 01 & 02 工具链安装 & 熟悉 Java"
subtitle: "开发图书馆管理系统"
author: "roife"
date: 2021-02-17
tags: ["BUAA - 面向对象设计与构造@Courses@Series", "北航@Tags@Tags", "面向对象@Tags@Tags", "Java@Languages@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# 工具链介绍

- Git
- JDK 1.8.0
  - Arch 上面自带了 Java 的版本管理
- IDEA
  - Checkstyle-IDEA：下载最新版的 plugins，然后在 settings 里面可以切换使用 checkstyle 的版本
  - MetricsReloaded
  - Statistic
  - .ignore
  - Markdown support
- Markdown 工具：推荐 Typora，我直接用 VSCode 了

# Pre - 02 图书管理系统

Pre 02 主要是熟悉面向对象和 Java 语法用的，分为 5 个 Tasks：
- Task 1～3：熟悉 Classes 和 OOP
- Task 4：引入 Exceptions
- Task 5：综合完成更复杂的操作

总的来说就是 ~~一天速成 Java ：从入门到入土~~。

## Task 1

### 题目

编写一个类 `Bookset`，拥有 `name`/`price`/`num` 三个 private field，封装成 `getXXX` 和 `setXXX` 的形式。

| type  | attribute |            意义             |         输出         |
| :---: | :-------: | :-------------------------: | :------------------: |
| 1/2/3 |    无     | 查询 Bookset 的名字/单价/数量 | 相应的名字/单价/数量 |
|   4   |  `name`   |   Bookset 名字更改为`name`   |          无          |
|   5   |  `price`  |  Bookset 价格更改为`price`   |          无          |
|   6   |   `num`   |   Bookset 数量更改为`num`    |          无          |
|   7   |    无     |       查询 Bookset 总价       |  一个浮点数表示总价  |

### 分析

第一个 Tasks 非常友好，没啥坑，主要用来熟悉 Git 的使用和评测机的用法。

主要提交的时候只要提交 `src` 文件夹就好了，所以我们可以只在 `src` 下面进行 `git init`。另外记得写好 `.gitignore`。

另一件事情就是代码风格检查。最好在每一次提交前对每一个文件用 `check style` 检查一遍，这样交上去就不用担心格式的问题了。

## Task 2

### 题目

在 Task 1 的基础上封装一个 `Bookshelf` 类，内部是一个 `Bookset` 的容器。同时主程序里面有一个 `Bookshelf` 的类，代表我们有多个书架，然后要实现以下操作：

| type |          attribute           |                             意义                             |                        输出                        |
| :--: | :--------------------------: | :----------------------------------------------------------: | :------------------------------------------------: |
|  1   |             `i`              |                 查询`i`号书架单价最高书单价                  |                     一个浮点数                     |
|  2   |             `i`              |                   查询`i`号书架丛书的总价                    |                     一个浮点数                     |
|  3   | `i name price num` | 向`i`号书架加入一套书 |                         无                         |
|  4   |           `i name`           |               由`i`号书架移出书名为`name`的书                | 一个整数，表示该书架的剩余书籍总册数 |

**Warnings** 输入数据范围为 Double/Long，而且中途运算可能会溢出，所有要用 `BigInteger`/`BigDecimal`。

还有就是记得操作 4 有输出。

### 分析

~~略毒瘤，主要坑点在大数类~~

首先，主要把 Task 1 里面和 `num` 相关的操作**全部改成** `long`，包括数据类型，以及读入要用 `scanner.nextLong()`。

然后我们分析一下哪几个地方要用大数类：
- `Bookshelf.getMaxPrice`
- `Bookshelf.getTotalPrice`
- `Bookshelf.removeBookset`
- **`Bookset.getTotalPrice`**：这个最毒瘤，不能忘记改

另一个坑点就是，注意使用 `BigDecimal.valueOf(d)` 来将浮点数转换为大整数，直接用 `BigDecimal(d)` 会有误差（**小心**）。

这里的容器推荐用 `HashMap`。

## Task 3

### 题目

这一个 Task 主要熟悉继承和多态。

首先题目给出了一个 `Bookset` 的子类的继承关系，然后编写新的操作。

书籍类型 | 属性
-|-
`Other` | 与基础类 `Bookset` 相同
`OtherA` | 包括 `Bookset` 全部属性，新添加最小阅读年龄 `age`
`Novel` | 包括 `OtherA` 全部属性，新添加完结与否标志 `finish`
`Poetry` | 包括 `OtherA` 全部属性，新添加作者 `author`
`OtherS` | 包括 `Bookset` 全部属性，新添加出版年份 `year`
`Math` | 包括 `OtherS` 全部属性，新添加使用年级
`Computer` | 包括 `OtherS` 全部属性，新添加专业类型

| type |                  attribute                  |                             意义                             |                        输出                        |
| :--: | :-----------------------------------------: | :----------------------------------------------------------: | :------------------------------------------------: |
|  1   |                  `i name`                   |            查询`i`号书架上书名为`name`的丛书信息             | 丛书的全部属性，各项间以空格分隔（与输入顺序一致） |
|  2   |                     `i`                     |                 查询`i`号书架的有多少种丛书                  |                      一个整数                      |
|  3   |                     `i`                     |                   查询`i`号书架的书籍总数                    |                      一个整数                      |
|  4   | `i type name price num var1 var2` | 向`i`号书架加入一套书 |                         无                         |
|  5   |                  `i name`                   |              由`i`号书架移出书名为`name`的丛书               |    一个整数，表示该书架的剩余书籍总数    |

建议熟悉**工厂方法**。

### 分析

#### 架构

这个 Task 开始正式进入架构设计了，我来讲讲我的架构（其实挺直观的），根据这个架构分析每个类继承的属性和新增的属性，然后对应封装即可。

```
Bookshelf (abstract)
 |- Other
 |- Art (abstract)
     |- OtherA
     |- Novel
     |- Poetry
 |- Science (abstract)
     |- OtherS
     |- Computer
     |- Math
```

其中 `getInformation` 要在每一个子类中重载（除了 `OtherX`，因为它们直接就是抽象类的实现）。

#### 工厂方法

这里不具体展开说工厂方法，只是简单介绍一下概念。

通常来说我们用 `new ClassName` 来创建一个对象，但是在这个 Task 中，我们的类非常多，需要要一个很大的 `switch` 或 `if` 语句来根据要求创建对象，而这放在代码里是很丑陋的，于是我们想到能不能将这个 `switch`/`if` 语句单独封装成一个类。我们传入想要的对象的类型，以及创建需要的参数，然后它返回我们需要的对象（当然类型是 `Bookshelf`，因为是多态）。

这就像一个工厂一样，我们告诉工厂要生产什么，并且提供需要的原料（构造器参数），然后工厂就把它生产出来返回给我们。

具体来说，是这样写的（个人写法，轻拍）：

```java
public class BooksetFactory {
    public static Bookset getBookset(String type, Scanner scanner) {
        String name = scanner.next();
        double price = scanner.nextDouble();
        long num = scanner.nextLong();

        switch (type) {
            case "Other":
                return new Other(name, price, num, "Other");
            case "OtherA":  {
                long age = scanner.nextLong();
                return new OtherA(name, price, num, age, "OtherA");
            }
            case "Novel":     // ...
            case "Poetry":    // ...
            case "OtherS":    // ...
            case "Math":      // ...
            case "Computer":  // ...
            default: {
                return null;
            }
        }
    }
```

这里一个比较巧妙的点是，我们传入的参数是 `Scanner`。因为不同的类型所需要的参数不一样，所以我们传入 `Scanner`，让工厂按需读取。

> 怎么有种共产主义的感觉

## Task 4

### 题目

增加异常处理。输入和 Task 3 一样，但是对于每个操作需要增加异常处理信息。

| type  |               异常                |             输出              |
| :---: | :-------------------------------: | :---------------------------: |
|   1   | `i`号非空书架上不存在书名为`name`的书 | Oh, no! We don't have `name`. |
| 1 2 3 |         `i`号书架为空书架         |    Oh, no! This is empty.     |
|   4   | `i`号书架上已存在书名为`name`的书 |   Oh, no! The `name` exist.   |
|   5   | `i`号书架上不存在书名为`name`的书 |   mei you wo zhen mei you.    |

### 分析

注意 "Oh, no! We don't have `name`." 这个地方的 `.` 和空格！~~毒瘤~~

然后就是讲讲异常类咋写。

这里用第三个异常举一个例子，继承 `Exception`：

```java
package exceptions;

public class BooksetExistedException extends Exception {
    public BooksetExistedException(String name) {
        super("Oh, no! The " + name + " exist.");
    }
}
```

照葫芦画瓢写出另外三个即可，可以把这四个放在 `src/exceptions` 文件夹下，因为他们都属于 `exceptions` 包。

在需要用到的文件里先 `import exceptions.BooksetExistedException;` 就可以了。

建议先判断是否会发生异常，然后再处理其他工作。

需要注意的是，操作 5 要返回剩余的书。这里你不能直接在 `removeBookset` 里面调用 `getTotalNum`。因为后者抛出了一个 `EmptyShelfException` 异常，但是前者是不会抛出这个异常的，前者的异常是 `BooksetNotExistedException`。而如果你在前者使用 `getTotalNum` 的地方加上 `try...catch`，你会发现 `catch` 里面没什么好写的，因为本来这里就不会抛出异常，但是这个和代码风格检查又冲突了。所以我推荐这里将 `getTotalNum` 中计算总数的部分提取出一个新方法 `private BigInteger getTotalNumHelper()`，然后两个函数分别先处理完各自的异常，然后调用这个新方法返回结果。

这里还有一个大坑点，就是某一行可能有多余的参数，如 `2 i name` 里面 `name` 是多余的。建议每次处理完一行直接换行，例如：

```java
try {
    bookshelves.get(i).addNewBookset(bookset);
} catch (BooksetExistedException e) {
    System.out.println(e.getMessage());
} finally { // 想想为啥要放在 finally 里面
    scanner.nextLine();
}
```

## Task 5

### 题目

增加**比较函数**。

现在可以将两个书架进行合并，合并的方式为复制两个书架，然后将内部的丛书合并，并将合并后的书架作为新书架（原来的不会消失）。合并丛书时将两个相同名字的丛书的数量相加作为新丛书。注意，如果两套丛书**名字**相同，但是除了**数量**外有其他不同的属性，那么就抛出异常。


| type | attribute |            意义            |            输出            |
| :--: | :-------: | :------------------------: | :------------------------: |
|  6   |   `i j`   | 对`i`号书架和`j`号书架求并 | 一个整数，表示新书架的编号 |

| type |                异常                |       输出       |
| :--: | :--------------------------------: | :--------------: |
|  6   | `i`号书架与`j`号书架上的书发生冲突 | Oh, no. We fail! |

### 分析

我们先分解题目。首先合并两个书架直接用 `HashMap`。~~具体的做法只能说懂得都懂，不懂的我也不好多说，这事牵扯太多，说了对你我都没有好处。~~

重点在于两个问题：`clone() & equals()`。

首先谈谈第一个。在 Java 中，`Object` 的 `clone()` 方法默认是 `protected` 的，所以我们要为每一个类重载 `public XXX clone()` 方法。所幸的是，这里的类的成员只有两种：基本类型和 String。而后者是不可变对象，所以我们直接使用默认 `super.clone()` 就好了，不需要自己进行 Deep Copy。而这个可以由 IntelliJ 自动生成。

然后是 `equals()`。类似于上一个，这个也可以由 IntelliJ 自动生成。

具体为什么这两个函数这么特殊，参见我的另一篇博文 `「Core Java」 03 Inheritance`。

注意 IntelliJ 自动生成方法的一个细节，Double 比较用 `Double.compare(bookset.price, price) == 0`，可以不用手写 EPS 了。

值得注意的是，这里的 `clone()` 和 `equals()` 在这个地方也可以用工厂模式写。