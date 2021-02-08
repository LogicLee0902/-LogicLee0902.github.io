---
layout: "post"
title: "「Core Java」 03 Interfaces, Lambda Expressions, and Inner Classes（Unfinished）"
subtitle: "接口，lambda 表达式和内部类"
author: "roife"
date: 2021-01-28

tags: ["Java@L", "Core Java@B", "未完成@T"]
language: zh-CN
catalog: true
header-image: ""
header-style: text
katex: true
---

# 接口

Java 中的接口有点像 Swift 的 protocol 或者 C++ 的纯虚基类，用 `interface` 声明。

```java
public interface Comparable<T> {
   int compareTo(T other);
}
```

接口中的方法默认是 `public` 的。

