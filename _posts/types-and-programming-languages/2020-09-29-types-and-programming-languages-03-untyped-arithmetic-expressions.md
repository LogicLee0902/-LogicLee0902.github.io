---
layout: "post"
title: "「TAPL」 03 Untyped Arithmetic Expressions"
subtitle: "开始"
author: "roife"
date: 2020-10-04

tags: ["Types and Programming Languages@Books@Series", "PKU - 编程语言的设计原理@Courses@Series", "程序语言理论@Tags@Tags", "类型系统@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
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

t 是一个 meta-variable (相当于一个 placeholder).

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

## Proposition: $\mathcal{T} = S$

由于 $\mathcal{T}$ 是满足 inductively 定义条件的最小集合, 所以只要证明 $S$ 也是满足 inductively 定义的最小集合即可. 即证明:
- $S$ 满足 inductively 定义的条件
- 任何满足 inductively 定义的集合都包含 $S$ (即 $S$ 是满足条件的最小集合)

第一点证明显然. 对于第二点证明, 可以先证 $S_i \subset S'$ (归纳).

# Induction on terms

根据 $\mathcal{T} = S$ , 可以递归定义出一些关于集合 $S$ 的函数, 也可以对于 terms 的命题进行递归证明.

## Consts(t)

$Consts(\mathtt{t})$: $\mathtt{t}$ 中出现的常量的集合.

$$Consts(\mathtt{true}) = \{\mathtt{true}\}$$

$$Consts(\mathtt{false}) = \{\mathtt{false}\}$$

$$Consts(\mathtt{0}) = \{\mathtt{0}\}$$

$$Consts(\mathtt{succ}\ t_1)  = Consts(\mathtt{t_1}) + 1$$

$$Consts(\mathtt{pred}\ t_1)  = Consts(\mathtt{t_1}) + 1$$

$$Consts(\mathtt{iszero}\ t_1)  = Consts(\mathtt{t_1}) + 1$$

$$Consts(\mathtt{if}\ t_1\ \mathtt{then}\ t_2\ \mathtt{else}\ t_3) = Consts(t_1) \cup Consts(t_2) \cup Consts(t_3)$$

## Size(t)

$Size(\mathtt{t})$: $\mathtt{t}$ 的大小, 可以看作是语法树中的节点个数

$$Size(\mathtt{true}) = 1$$

$$Size(\mathtt{false}) = 1$$

$$Size(\mathtt{0}) = 1$$

$$Size(\mathtt{succ}\ t_1)  = Size(\mathtt{true}) + 1$$

$$Size(\mathtt{pred}\ t_1)  = Size(\mathtt{true}) + 1$$

$$Size(\mathtt{iszero}\ t_1)  = Size(\mathtt{true}) + 1$$

$$Size(\mathtt{if}\ t_1\ \mathtt{then}\ t_2\ \mathtt{else}\ t_3) = Size(t_1) + Size(t_2) + Size(t_3) + 1$$

## depth(t)

$depth(\mathtt{t})$: 语法树的深度

$$depth(\mathtt{true}) = 1$$

$$depth(\mathtt{false}) = 1$$

$$depth(\mathtt{0}) = 1$$

$$depth(\mathtt{succ}\ t_1)  = depth(\mathtt{true}) + 1$$

$$depth(\mathtt{pred}\ t_1)  = depth(\mathtt{true}) + 1$$

$$depth(\mathtt{iszero}\ t_1)  = depth(\mathtt{true}) + 1$$

$$depth(\mathtt{if}\ t_1\ \mathtt{then}\ t_2\ \mathtt{else}\ t_3) = \max{depth(t_1) + depth(t_2) + depth(t_3)} + 1$$

## Proposition: $\mid Consts(t) \mid \leq size(t)$

分为 3 个 cases
- $t$ 是  constants
- $t$ 是 $\mathtt{succ}\ t_1$ 或 $\mathtt{pred}\ t_1$ 或 $\mathtt{iszero}\ t_1$
- $t$ 是 $\mathtt{if}\ t_1\ \mathtt{then}\ t_2\ \mathtt{else}\ t_3$

## Theorem: Structural Induction

If, for each term $\mathtt{s}$,
given $P(\mathtt{r})$ for all immediate subterms $\mathtt{r}$ of $\mathtt{s}$
we can show $P(\mathtt{s})$,
then $P(\mathtt{s})$ holds for all $s$.

此处也可以换为证明对于所有的 $depth(\mathtt{r}) \le depth(\mathtt{s})$ 或 $size(\mathtt{r}) \le size(\mathtt{s}) $ 进行归纳.

# Semantic Styles

- Syntax (语法): 程序的结构
- Semantic (语义): 程序的值

## Operational semantics

Operational Semantics 用一个 abstract machine 来定义程序的语义, machine 的每一个状态都是一个 term, 状态之间用 transition function (要么对 $t$ 进行 simplification, 要么终止程序) 进行转移. machine 一开始的状态是 $t$, 终止状态就是 $t$ 的值.

通常一种语言会有多种 operational semantics, 一种是贴近程序员的, 另一种是贴近解释器或编译器的. 证明这两种语义相同相当于证明语言实现的正确性.

## Denotational semantics

Denotational Semantics 首先确定了一个 semantics domains, 然后定义一个 interpretation function 将 term 映射到 semantics domains. (domain theory)

Denotational Semantics 好处在于可以突出语言的核心概念, 并且 semantics domain 可以导出许多 laws, 比如用来确定两个程序是否相同.

## Axiomatic semantics

以上两种 semantics 都是先定义语言的行为, 然后导出一些 laws. 而 Axiomatic Semantics 将 laws 本身作为程序的定义. Terms 本身即证明. (Hoare Logic)

