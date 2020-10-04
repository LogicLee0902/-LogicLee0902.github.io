---
layout: "post"
title: "「TAPL」 03 Untyped Arithmetic Expressions"
subtitle: "开始"
author: "roife"
date: 2020-10-04

tags: ["B「Types and Programming Languages」", "L「OCaml」", "笔记", "C「PKU - Design Principles of Programming Languages」", "笔记"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
mathjax: true
---

# Introduction

先用类似于 BNF 的 grammar 定义一个简单的语言:

[Terms, Grammarly]

$$
\begin{aligned}
t \Coloneqq & & (terms) \\
    & \mathtt{true} & (constant\ true) \\
    & \mathtt{false} & (constant\ false) \\
    & \mathtt{if}\ t\ \mathtt{then}\ t\ \mathtt{else}\ t & (conditions) \\
    & \mathtt{0} & (constant\ 0) \\
    & \mathtt{succ}\ t & (successor) \\
    & \mathtt{pred}\ t & (predecessor) \\
    & \mathtt{iszero}\ t & (zero\ test)
\end{aligned}
$$

t 是一个 metavariable (相当于一个 placeholder).

这种语言的程序是由这种语法生成的 term, 如 `iszero (pred (succ 0)) → true`.

# Syntax

## [Terms, Inductively]

The set of terms is the **smallest set $\mathcal{T}$**

1. $\\{ \mathtt{true}, \mathtt{false}, \mathtt{0} \\} \subseteq \mathcal{T}$;
2. if $t_1 \in \mathcal{T}$, then $\\{\mathtt{succ}\ t_1, \mathtt{pred}\ t_1, \mathtt{iszero}\ t_1\\} \subseteq \mathcal{T}$;
3. if $t_1 \in \mathcal{T}, t_2 \in \mathcal{T}, t_3 \in \mathcal{T}$, then $\mathtt{if}\ t_1\ \mathtt{then}\ t_2\ \mathtt{else}\ t_3 \in \mathcal{T}$.

## [Terms, by Inference Rules]

The set of terms is defined by the following rules:

$$\mathtt{true} \in \mathcal{T}$$

$$\mathtt{false} \in \mathcal{T}$$

$$\mathtt{0} \in \mathcal{T}$$

$$\frac{t_1 \in \mathcal{T}}{\mathtt{succ}\ t_1 \in \mathcal{T}}$$

$$\frac{t_1 \in \mathcal{T}}{\mathtt{pred}\ t_1 \in \mathcal{T}}$$

$$\frac{t_1 \in \mathcal{T}}{\mathtt{iszero}\ t_1 \in \mathcal{T}}$$

$$\frac{t_1 \in \mathcal{T}, t_2 \in \mathcal{T}, t_3 \in \mathcal{T}}{\mathtt{if}\ t_1\ \mathtt{then}\ t_2\ \mathtt{else}\ t_3 \in \mathcal{T}}$$

## [Terms, Concretely]

For each natural number $i$, define a set $S_i$ as follows:

$$S_0 = \emptyset$$

$$
\begin{aligned}
S_{i+1} = & \{\mathtt{true}, \mathtt{false}, \mathtt{0}\} \\
& \cup \{\mathtt{succ}\ t_1, \mathtt{pred}\ t_1, \mathtt{iszero}\ t_1 \mid t_1 \in S_i\} \\
& \cup \{\mathtt{if}\ t_1\ \mathtt{then}\ t_2\ \mathtt{else}\ t_3 \mid t_1, t_2, t_3 \in S_i\}
\end{aligned}
$$

Finally, let

$$S = \bigcup_i S_i$$
