---
layout: "post"
title: "「BUAA-OO-Lab」 Pre - 03 正则表达式与应用"
subtitle: "应用正则表达式匹配邮件"
author: "roife"
date: 2021-02-20
tags: ["BUAA - 面向对象设计与构造@Courses@Series", "北航@Tags@Tags", "面向对象@Tags@Tags", "Java@Languages@Tags", "正则表达式@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

Pre-03 总共有 6 个 Tasks 迭代开发，都和正则表达式有关。类似于 Pre-02 开发一个简单的邮件信息系统。

- Task 1～4：应用正则表达式提取所需文本
- Task 5～6：封装操作，和数据结构结合

一开始 AK 的代码太烂了，所以后面针对部分题目重新写了一边。

# Task 1

## 题目

邮件信息共有四种格式：
1. `username@domain-yyyy-mm-dd`
2. `username@domain-yyyy-mm-dd-hh`
3. `username@domain-yyyy-mm-dd-hh:mimi`
4. `username@domain-yyyy-mm-dd-hh:mimi:ss`

- `y` 代表一位年份数字，`m` 代表一位月份数字，`d` 代表一位日期数字，`h` 代表一位小时数字，`mi` 代表一位分钟数字，`s` 代表一位秒数数字
- `username` 为只包含**大小写字母、`-`** 的长度不为零的字符串，**大小写不敏感**。
- `domain` 为只包含**大小写字母、数字、`.`** 的长度不为零的字符串，**大小写敏感**。
- `domain` 中第一个 '.' 字符（不含）之前的字符串 （保证域名含有 '.' 字符且域名类别不为空） 为 **类别**。如（`@buaa.edu.cn` 类别为 `buaa`）

现在输入一行邮件信息，包含了多个邮件地址。邮件地址间用空格、逗号或逗号+空格分割。要提取出所有的邮件地址，输出数量。

## 分析

第一个题目可以不用正则表达式，直接读入一行字符串，然后 `replace` 和 `split` 就好了。

```java
String[] splitedStr = s.replace(", ", " ")
    .replace(",", " ")
    .split(" ");
```

# Task 2

Task 2 开始建议使用正则表达式。

## 题目

输入格式和 Task 1 相同，但是数据改为多行，要求提取出所有邮件地址和时间，将邮件地址按照用户名小写的字典序排序输出。

## 分析

直接上正则表达式可以了。

在架构上我使用了一个 `MailFactory` 简单工厂类来生成需要的 `Mail` 对象。

记得 `matcher` 一定要 `find()` 一次才能访问 `group()`！

```java
private static final String patternUsername = "(?<username>[A-Za-z-]+)";
private static final String patternDomain = "(?<domain>[A-Za-z0-9.]+)";
private static final String patternYyyyMmDd =
    "(?<yyyy>[\\d]{4})-(?<mm>[\\d]{2})-(?<dd>[\\d]{2})";
private static final String patternHhMimiSsMaybe =
    "((?<hh>[\\d]{2}):)?((?<mimi>[\\d]{2}):)?(?<ss>[\\d]{2})?"; // 贪心
private static final String pattern = patternUsername + "@" + patternDomain +
    "-" + patternYyyyMmDd + "(-)?" + patternHhMimiSsMaybe;
public static final Pattern regex = Pattern.compile(pattern);
```

值得注意的是，这里排序输出可以用 stream 轻松完成。

```java
mailList.stream()
    .sorted(Comparator.comparing(Mail::getUsername))
    .map(Mail::display)
    .forEach(System.out::println);
```

# Task 3

在上一个 Task 上增加对邮件进行分析的功能。

## 题目

输入分为三部分：
- 邮件信息部分（若干行），形式与 Task2 的输入相同
- `END_OF_INFORMATION`（一行），表示邮件信息部分结束
- 查询指令序列（若干行），每条指令一行，输入一个查询指令和 `username`，然后查询邮件的信息。若 `username` 不存在，输出 `no username exists `。如果 `username` 存在，但相应的时间信息不存在，输出 `null`。

## 分析

没什么特殊的，就是加上 `getXxx()` 的函数，然后根据输入输出对应的信息就可以了。

# Task 4

略独立的一个 Task。

## 题目

定义 5 类用户名（括号内表示重复字数）。
- A类：
  - 用户名中存在⼀个子串为 `a(x) + b(y) + a(z) + c(i)`
  - $x \in [2, 3]$，$y \in [2,4]$，$z \in [2, 4]$，$i \in [2, 3]$
- B类：
  - 用户名中存在⼀个子串为 `a(x) + ba(y) + bc(z)`
  - $x \in [2, 3]$，$y \in [0,100000000]$，$z \in [2, 4]$
- C类
  - 用户名中存在⼀个子串 s 满足 `lower(s) = a(x) + ba(y) + bc(z)`
  - $x \in [2, 3]$，$y \in [0,100000000]$，$z \in [2, 4]$
- D类：
  - 该用户名存在⼀个前缀为 `a(x) + b(y) + c(z)`
  - $x \in [0, 3]$，$y \in [1,1000000]$，$z \in [2, 3]$
  - 且该用户名存在⼀个后缀 s 满足 `lower(s) = b(i) + a(j) + c(k)`
  - $i \in [1, 2]$，$j \in [1,2]$，$k \in [0, 3]$
- E类:
  - 该用户名存在⼀个子序列 `a(x) + b(y) + c(z) + b(i) + c(j)`
  - $x \in [1, 3]$，$y \in [2, 10000]$，$z \in [1, 2]$，$i \in [1, 3]$，$j \in [2, 10000]$

输入不超过 100 个字符。

在 Task 3 的基础上支持查询邮件的类型，一种邮件可以对应多种类型。

## 分析

这个题目显然不能暴力做，那样的话生成的正则表达式就太大了。

我们发现输入不超过 100 个字符，那么单个字符匹配不超过 100 次，双字符匹配不超过 50 次，这样就可以简化正则表达式了。

而对于 E 题的子序列，我们用贪心的算法，因为不要求连续，所以只要每种字符找最少次数就可以了。

出于封装的考虑，我们可以将这个题目的操作封装成一个单独的类 `UsernameAnalysis`。每种类别对应一种函数，一次调用判断即可。

```java
String patternA = "a{2,3}b{2,4}a{2,4}c{2,3}";
String patternB = "a{2,3}(ba){0,50}(bc){2,4}";
String patternC = "a{2,3}(ba){0,50}(bc){2,4}";
String patternDprefix = "^(a){0,3}b{1,100}c{2,3}";
String patternDsuffix = "b{1,2}a{1,2}c{0,3}$";
String patternE = "a(.*)b(.*)b(.*)c(.*)b(.*)c(.*)c";
```

这里要注意的一点是匹配要用 `find()` 而不是 `matches()`，后者是判断整个串能否匹配。

# Task 5

从这一题开始考察封装能力。

## 题目

输入格式和之前一样，现在询问部分共有两种操作：
+ 删除指令：不输出
  + **删除username对应的邮件**，指令：`del username`
  + **删除指定日期所有邮件**，指令：`del all yyyy-mm-dd `

+ 查询指令
  + **查询username对应邮件的日期**，指令：`qutime username`，若不存在则输出 `no username exists`。
  + **查询某日期所有邮件**，指令：`qutime all yyyy-mm-dd`，若不存在则输出 `no email exists`。

## 分析

用一个 Hashmap 封装即可。

我用 Hashmap 存储键对 `Username : Mail`，然后考虑到操作指令是独立的，所以将其封装在新的类 `Mailbox` 中。

其中值得注意的是，`Mailbox` 可以写得非常 Functional。

```java
public boolean deleteByUsername(String username) {
    return mailbox.entrySet()
        .removeIf(entry -> entry.getKey().equals(username));
}

public boolean deleteByDate(String date) {
    return mailbox.entrySet()
        .removeIf(entry -> entry.getValue().getDate().equals(date));
}

public Mail queryByUsername(String username) {
    return mailbox.get(username);
}

public List<Mail> queryByDate(String date) {
    return mailbox.values().stream()
        .filter(mail -> mail.getDate().equals(date))
        .sorted(Comparator.comparing(Mail::getUsername))
        .collect(Collectors.toList());
}
```

顺便有个毒瘤的坑：你判断是按照 `username` 还是日期操作的时候，不能判断第二个参数是不是 `all`，因为可能存在 `username.equals("all")`！必须要判断 `split()` 后的长度。

# Task 6

可以考察之前做得做得怎么样，如果之前写得不好的话，这里要大规模重构。

## 问题

现在改变输入方式

1. `dd-mm-yyyy-username@domain-place`
2. `hh-dd-mm-yyyy-username@domain-place`
3. `mimi:hh-dd-mm-yyyy-username@domain-place`
4. `ss:mimi:hh-dd-mm-yyyy-username@domain-place`

其中 `place` 由英文字母组成，对大小写敏感。其他的不变。

对于询问，要求将邮件按照地点归类。并且每个询问附加一个 `place` 参数，表示在 `place` 的集合中询问。如 `qutime username place`。

如果这个 `place` 没有邮件，就返回 `no place exists`。

## 分析

首先对于新增的参数，只要稍微改变一下正则表达式就好了。

```java
private static final String patternUsername = "(?<username>[A-Za-z-]+)";
private static final String patternDomain = "(?<domain>[A-Za-z0-9.]+)";
private static final String patternPlace = "(?<place>[A-Za-z]+)";
private static final String patternYyyyMmDd =
    "(?<dd>[\\d]{2})-(?<mm>[\\d]{2})-(?<yyyy>[\\d]{4})";
private static final String patternHhMimiSsMaybe =
    "((((?<ss>[\\d]{2}):)?(?<mimi>[\\d]{2}):)?(?<hh>[\\d]{2}))?";
private static final String pattern = patternHhMimiSsMaybe + "(-)?" + patternYyyyMmDd + "-" +
    patternUsername + "@" + patternDomain + "-" + patternPlace;
public static final Pattern regex = Pattern.compile(pattern);
```

然后就是容器操作的问题。

按照题目要求，我们要在原来的容器外面再套一个 map。这里建议创建一个新的类 `MailboxInPlace`，因为本来操作就比较复杂。

而且这里也有一个坑，题目要求如果某个 `place` 下面没有邮件，就要输出错误提示。这个不仅要检查 `containsKey()`，还要检查具体的值下面有没有邮件，所以条件为：

```java
public boolean isPlaceEmpty(String place) {
    return !hashMap.containsKey(place) || hashMap.get(place).isEmpty();
}
```
