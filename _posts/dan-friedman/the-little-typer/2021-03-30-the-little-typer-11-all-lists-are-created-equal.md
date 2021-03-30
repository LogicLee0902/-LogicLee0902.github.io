---
layout: "post"
title: "「The Little Typer」 11 All lists are created equals"
subtitle: ""
author: "roife"
date: 2021-03-30

tags: ["The Little Typer@Books@Series", "程序语言理论@Tags@Tags", "函数式编程@Tags@Tags", "Dependent Type@Tags@Tags", "Dan Friedman@Series@Series", "Pie@Languages@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# `ind-Vec`

一个 `ind-Vec` 表达式为：

```lisp
(ind-Vec n es
  mot
  base
  step)
```

相比于 `ind-List`，`ind-Vec` 的每个参数都多了一个 Nat `k` 表示数量。

`mot` 的类型为：

```lisp
(Π ((k Nat))
  (→ (Vec E k)
    U))
```

可以发现 `mot` 的参数中没有 `E` 而有 `k`，因为对于一个 `Vec`，`E` 是确定不变的，而 `k` 有可能会改变。确定不变的称为 parameters，而有可能会改变的称为 indices。

> A family of types whose argument is an index is sometimes called “an indexed family.

`base` 的类型为 `(mot zero vecnil)`，`step` 的类型为：

```lisp
(Π ((k Nat)
    (h E)
    (t (Vec E k)))
  (→ (mot k t)
    (mot (add1 k) (vec:: h t))))
```

> **The Law of `ind-Vec`**
>
> If `n` is a Nat, `target` is a `(VecE n)`, `mot` is a
>
> ```lisp
> (Π ((k Nat))
>   (→ (Vec E k)
>     U))
> ```
>
> `base` is a
>
> ```lisp
> (mot zero vecnil)
> ```
>
> and `step` is a
>
> ```lisp
> (Π ((k Nat)
>     (h E)
>     (t (Vec E k)))
>   (→ (mot k t)
>     (mot (add1 k) (vec:: h t))))
> ```
>
> then
>
> ```lisp
> (ind-Vec n target
>   mot
>   base
>   step)
> ```
>
> is a `(mot n target)`