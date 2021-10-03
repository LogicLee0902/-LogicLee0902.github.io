---
layout: "post"
title: "「TAPL」 13 References"
subtitle: "Reference Type/Mutable Types/Pointer"
author: "roife"
date: 2021-10-03

tags: ["Types and Programming Languages@Books@Series", "PKU - 编程语言的设计原理@Courses@Series", "程序语言理论@Tags@Tags", "类型系统@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

本章讨论的是 STLC + Unit + References 的类型系统。

注意本章中描述的 references 不是指 C++ 中的引用类型，而是指一个存了值的 cell，其中 cell 的值可读可写，所以两次读出来的值可能不同。即 references 可以理解为可变类型。

# Introduction

在 ML 系语言中变量分为两种，一种是拥有值的常量，另一种是拥有 cell 的变量。

对于前者而言可以直接将其用作运算 `x + 5`，但是不能向其赋值。对于后者而言可以直接向其赋值 `y := 10`，但是不能将其用于运算。如果要将其用作运算就必须进行 dereference，即 `!y + 5`。

在 C/Java 当中可以将所有的变量看作是 references，其和 ML 系语言的区别有亮点：
1. C/Java 中 references 的 deref 过程是 implicit 的，即使用 cell 中的值不需要显式解引用（但是可以把 C 语言中的指针看成是 References）
2. C/Java 中的变量允许 `null` 值存在，所以实际上是一种 `Option<Ref<T>>` 类型

## Basics

References 有三种基本操作：allocation，的dereferencing 和 assignment。

### allocation

$$
\begin{aligned}
r &= \operatorname{\mathtt{ref}}\ 5; \\
r &: \operatorname{\mathtt{Ref}}\ \operatorname{\mathtt{Nat}} \\
\end{aligned}
$$

### Dereferencing

$$
!r : \operatorname{\mathtt{Nat}}
$$

### assignment

$$
r := 7;
$$

注意 assignments 的返回值为 `unit`。

## Side Effects and Sequencing

由于 assignments 的值为 `unit`，则可以利用 sequencing 写，使得语句顺序执行：

$$
(r := \operatorname{\mathtt{succ}}(!r); r := \operatorname{\mathtt{succ}}(!r); !r);
$$

## References and Aliasing

两个 references 可以指向同一个 cell，此时对其中一个的修改会影响到另一个，并且二者都称为是这个 cell 的 **aliases**。


## Shared State

Aliases 使得静态分析变得更困难。比如一种特殊的情况：

$$
(r := 1; r := !s);
$$

在大多数情况可以认为前者是冗余的，可以被删去；然而如果 $r$ 和 $s$ 指向的是同一个 cell，反而后者是冗余的。

Aliases 可以让程序的各个部分共享状态并进行“沟通”，即 `shared state`，这个可以用来实现“对象”的效果。

## References to Compound Types

References 结合函数可以用来实现一个（低效的）数组：

$$
\operatorname{\mathtt{NatArray}} = \operatorname{\mathtt{Ref}}\ (\operatorname{\mathtt{Nat}} \rightarrow \operatorname{\mathtt{Nat}});
$$

$$
\begin{aligned}
\operatorname{\mathtt{newarray}} &= \lambda \_ : \operatorname{\mathtt{Unit}}. \operatorname{\mathtt{ref}}\ (\lambda n : \operatorname{\mathtt{Nat}}. 0); \\
\operatorname{\mathtt{newarray}} &: \operatorname{\mathtt{Unit}} \rightarrow \operatorname{\mathtt{NatArray}}
\end{aligned}
$$

$$
\begin{aligned}
\operatorname{\mathtt{lookup}} &= \lambda a : \operatorname{\mathtt{NatArray}}. \lambda n : \operatorname{\mathtt{Nat}}. (!a)\ n; \\
\operatorname{\mathtt{lookup}} &: \operatorname{\mathtt{NatArray}} \rightarrow \operatorname{\mathtt{Nat}} \rightarrow \operatorname{\mathtt{Nat}}
\end{aligned}
$$

$$
\begin{aligned}
\operatorname{\mathtt{update}} &= \lambda a : \operatorname{\mathtt{NatArray}}. \lambda m : \operatorname{\mathtt{Nat}}. \lambda v : \operatorname{\mathtt{Nat}}. \\
& \qquad \operatorname{\mathtt{let}}\ oldf = (!a)\ \operatorname{\mathtt{in}} \\
& \qquad \quad a := (\lambda n : \operatorname{\mathtt{Nat}}. \operatorname{\mathtt{if}}\ \operatorname{\mathtt{equal}}\ m\ n\ \operatorname{\mathtt{then}}\ v\ \operatorname{\mathtt{else}}\ oldf\ n); \\
\operatorname{\mathtt{update}} &: \operatorname{\mathtt{NatArray}} \rightarrow \operatorname{\mathtt{Nat}} \rightarrow \operatorname{\mathtt{Nat}} \rightarrow \operatorname{\mathtt{Unit}}
\end{aligned}
$$

## Garbage Collection

垃圾回收对应了 deallocation 的过程。在很多现代的语言中都采用了垃圾回收，因为手动回收内存很难实现 type safety，容易造成 dangling reference 等问题。

# Typing

$$
\dfrac {
    \Gamma \vdash t_1 : T_1
} {
    \Gamma \vdash \operatorname{\mathtt{ref}}\ t_1 : \operatorname{\mathtt{Ref}}\ T_1
} \tag{T-Ref}
$$

$$
\dfrac {
    \Gamma \vdash t_1 : \operatorname{\mathtt{Ref}}\ T_1
} {
    \Gamma \vdash !t_1 : T_1
} \tag{T-Deref}
$$

$$
\dfrac {
    \Gamma \vdash t_1 : \operatorname{\mathtt{Ref}}\ T_1 \quad \Gamma \vdash t_2 : T_1
} {
    \Gamma \vdash t_1 := t_2 : \operatorname{\mathtt{Unit}}
} \tag{T-Assign}
$$

# Evaluation
