---
layout: "post"
title: "「TAPL」 14 Exceptions"
subtitle: "异常"
author: "roife"
date: 2021-10-05

tags: ["Types and Programming Languages@Books@Series", "PKU - 编程语言的设计原理@Courses@Series", "程序语言理论@Tags@Tags", "类型系统@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

程序在运行过程中会发生各种异常情况，此时可以通过 `Optional` 类型去处理，但是这种方式要求每个 caller 都要参与到错误的处理中。而使用异常机制则可以直接将 exceptions 交给 exceptions handler 处理（此时程序的 control flow 发生变化），或者直接终止程序。

本章基于 STLC 和一些扩展类型，并且分成三层：遇到异常直接终止程序、将异常移交 handler 处理以及将异常信息传递给 handler。

# Raising Exceptions

![14-1 Errors](/img/in-post/post-tapl/14-1-errors.png)

首先定义 `error`，其 evaluation rules 使得程序只要一遇到 `error` 就会退出。

这里只把 `error` 定义为 term 而不是 value，这样消除了规则调用的二义性。如果 `error` 是 value 的话，那么对于 $(\lambda x : \operatorname{\mathtt{Nat}}. 0)\ \operatorname{\mathtt{error}}$ 既可以调用 `E-AppAbs` 又可以调用 `E-AppErr2`。同样 `E-AppErr2` 也要求左侧为 $v_2$，防止有二义性。

`T-Error` 表明 `error` 可以是任何类型。例如在 $(\lambda x : \operatorname{\mathtt{Bool}}. x)(\operatorname{\mathtt{error}}\ \operatorname{\mathtt{true}})$ 中 `error` 的类型为 $\operatorname{\mathtt{Bool}} \rightarrow \operatorname{\mathtt{Bool}}$。但是 `T-Error` 这个打破了“所有 `term` 都只有一个类型”的规则，这一点会在后面解决（subtyping 或 parametric polymorphism）。

注意，这里的 `error` 不能是某个具体的类型，必须是 $T$，否则会违反 preservation 性质。例如在 $(\lambda x : \operatorname{\mathtt{Nat}}. x) ((\lambda y : \operatorname{\mathtt{Bool}}. 5) (\operatorname{\mathtt{error}}\ \operatorname{\mathtt{as}}\ \operatorname{\mathtt{Bool}}));$ 中，进行外部函数的规约时就会发生错误。

在加了 exceptions 类型后，preservation property 和原来一样（即 evaluation 不改变类型），但是 progress property 会发生变化（因为加入了 `error`，结果不一定再是 value）。

> **Theorem** Progress
>
> Suppose $t$ is a closed, well-typed normal form. Then either $t$ is a value or $t=\operatorname{\mathtt{error}}$.

# Handling Exceptions

![14-2 Error Handling](/img/in-post/post-tapl/14-2-error-handling.png)

`error` 求值的过程中会 unwinding call stack 并直接返回。Call stack 由 **activation records*** 组成，其中每个 record 都是一个函数调用。Exceptions 传播的过程就是 activation records 从栈中弹出的过程。

在 `error` 向上传播的过程中可以用 `error handler` 来捕获异常。此时的 exceptions 在传播时，会不断弹出 call stack 中的 activation records，直到遇到最近的 error handler，然后执行 handler。

这里用 $\operatorname{\mathtt{try}}\ t_1\ \operatorname{\mathtt{with}}\ t_2$ 来表示 error handler，一旦遇到 `error` 就执行 $t_2$。显然 $t_1$ 和 $t_2$ 的类型必须相同。

# Exceptions Carrying Values

![14-3 Exceptions carrying values](/img/in-post/post-tapl/14-3-exceptions-carrying-values.png)

发生异常时
