---
layout: "post"
title: "「The Little Typer」 06 Precisely How Many?"
subtitle: ""
author: "roife"
date: 2021-03-20

tags: ["The Little Typer@Books@Series", "程序语言理论@Tags@Tags", "函数式编程@Tags@Tags", "Dependent Type@Tags@Tags", "Dan Friedman@Series@Series", "Pie@Languages@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
---

# Vec

`(Vec E k)` 表示一个长度为 `k` 的 `(List E)`。
- `(Vec E zero)` 的 constructor 为 `vecnil`
- `(Vec E (add1 k))` 的 constructor `vec::`
  - 当 `e` 为 `E` 类型，而且 `es` 为 `(Vec E k)` 类型时，`(vec:: e es)` 的类型为 `(Vec E (add1 k))`。
  - 如果一个表达式的类型为 `(Vec E (add1 k))`，则这个列表一定至少有一个 entry

> **The Law of `Vec`**
>
> If `E` is a type and `k` is a Nat,
> then `(Vec E k)` is a type.

> **The Law of `vecnil`**
>
> `vecnil` is a `(Vec E zero)`.

> **The Law of `vec::`**
>
> If `e` is an `E` and `es` is a `(Vec E k)`,
> then `(vec:: e es)` is a `(Vec E (add1 k))`.

