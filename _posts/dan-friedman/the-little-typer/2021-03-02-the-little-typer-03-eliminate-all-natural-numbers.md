---
layout: "post"
title: "「The Little Typer」 03 Eliminate All Natural Numbers!"
subtitle: ""
author: "roife"
date: 2021-03-02

tags: ["The Little Typer@Books@Series", "程序语言理论@Tags@Tags", "函数式编程@Tags@Tags", "Dependent Type@Tags@Tags", "Dan Friedman@Series@Series", "Pie@Languages@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
---

# Total Functions

全函数就是指对于任意定义域中的值，都可以给出结果的函数。

> **Total Function**
>
> A function that always assigns a value to every possible argument is called a total function.
>
> **备注**：Pie 中所有函数都是全函数。而这使得 Pie 中子表达式的求值顺序是无关紧要的。因为如果不是全函数，那么不同的求值顺序会导致

# `iter-Nat`

`iter-Nat` 是一个函数，它判断一个 Nat 是否是 `zero`。如果不是，递归迭代 Nat。

使用格式如下：

```lisp
(iter-Nat target
  base
  step)
```

如果 `target is zero`，那么返回 `base`；否则如果是 `(add1 n)`，则返回 `(step (step (... (step zero))))`。可以发现 `step` 是一个 λ 表达式。

`iter-Nat` 也是一个**模式匹配**，但是他可以对自然数进行递归。

> **The Law of `iter-Nat`**
>
> If `target` is a `Nat`, `base` is an `X`, and `step` is an
>
> ```lisp
> (→ X X)
> ```
>
> then
>
> ```lisp
> (iter-Nat target
>   base
>   step)
> ```
>
> is an `X`.

> **The First Commandment of `iter-Nat`**
>
> If
>
> ```lisp
> (iter-Nat zero
>   base
>   step)
> ```
>
> is an `X`, then it is the same `X` as `base`.

> **The Second Commandment of `iter-Nat`**
>
> If
>
> ```lisp
> (iter-Nat (add1 n)
>   base
>   step)
> ```
>
> is an `X`, then it is the same `X` as
>
> ```lisp
> (step
>   (iter-Nat n
>     base
>     step))
> ```
>
> **注解**：为什么前面说 Recursion is not an option，这里又允许迭代呢？是因为这里 `iter-Nat` 的迭代方式是可以确定会 terminate 的，所以加入不影响使用。（其实是开了一个洞）`iter-Nat` 实际上是设定了终值和每次递归的计算方式，最多递归 n 次。

## `+`

通过 `iter-Nat` 我们就可以定义加法了：

```lisp
(claim step-+
  (→ Nat
    Nat))

(define step-+
  (λ (n)
    (add1 n)))

(claim +
  (→ Nat Nat
    Nat))

(define +
  (λ (n j)
    (iter-nat n
      j
      step-+)))
```

# `rec-Nat`

`rec-Nat` 结合了 `which-Nat` 和 `iter-Nat` 二者。

primitive recursion

```lisp

```