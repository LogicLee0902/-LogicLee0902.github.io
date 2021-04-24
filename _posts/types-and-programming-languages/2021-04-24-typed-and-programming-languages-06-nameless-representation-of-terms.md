---
layout: "post"
title: "「TAPL」 06 Nameless Representation of Terms"
subtitle: "匿名表示"
author: "roife"
date: 2021-04-24

tags: ["Types and Programming Languages@Books@Series", "PKU - 编程语言的设计原理@Courses@Series", "程序语言理论@Tags@Tags", "类型系统@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# Variable Representations

1. Represent variables symbolically, with variable renaming mechanism to avoid capture
2. Represent variables symbolically, with bound variables and free variables are all different (**Barendregt convention**)
3. Some “canonical” representation of variables and terms that does not require renaming
4. Avoid substitution by mechanisms such as **explicit substitutions**
5. Avoid variables (**Combinatory Logic**)

下面讲的是第三种。

# Terms and Contexts

De Bruijn 的做法是用自然数来表示变量，其中 $k$ 表示从内到外第 $k$ 级深度的 λ。这样的项被称为 De Bruijn terms，变量被称为 De Bruijn indices。例如：

$$
\begin{aligned}
    \mathtt{fix} &= \lambda f. (\lambda x. f\ (\lambda y. x\ x\ y))\ (\lambda x. f\ (\lambda y. x\ x\ y));\\
    &= \lambda.(\lambda. 1\ (\lambda. (1\ 1)\ 0))(\lambda. 1\ (\lambda. (1\ 1)\ 0));
\end{aligned}
$$

> **Definitions**: Terms
>
> Let $\mathcal{T}$ be the smallest family of sets $\{\mathcal{T}_0, \mathcal{T}_1, \mathcal{T}_2, \dots\}$ such that
> 1. $k \in \mathcal{T}_n$ whenever $0 \le k < n$
> 2. if $t_1 \in \mathcal{T}_n$ and $n > 0$, then $\lambda. t_1 \in \mathcal{T}_{n-1}$
> 3. if $t_1 \in \mathcal{T}_n$ and $t_2 \in \mathcal{T}_n$, then $(t_1, t_2) \in \mathcal{T}_{n-1}$

其中，$T_n$ 集合中的项被称为 **$n$-terms**，即包含了不超过 $n$ 个自由变量。

对于 closed terms，那么显然它属于所有的 $\mathcal{T}_n$，而且其 de Bruijn representation 是唯一的。

## Naming context

对于包含自由变量的 terms，其中的自由变量需要用 naming context 表示。

> **Definition**
>
> Suppose $x_0$ through $x_n$ are variable names from $\mathcal{V}$ . The naming context $\Gamma = x_n, x_{n−1}, \dots, x_1, x_0$ assigns to each $x_i$ the de Bruijn index $i$. Note that the rightmost variable in the sequence is given the index $0$; this matches the way we count λ binders — from right to left — when converting a named term to nameless form. We write $dom(Γ)$ for the set $\{x_n, \dots ,x_0\}$ of variable names mentioned in $\Gamma$.

例如对于 naming context $\Gamma = x \rightarrow 4; y \rightarrow 3; z \rightarrow 2; a \rightarrow 1; b \rightarrow 0$，term $\lambda w. \lambda y. w$ 可以表示成 $\lambda . \lambda . 4\ 0$。

即假设最大的深度为 $m$（bound variables 最大为 $m$），那么自由变量即为 $m$ 加上 naming context 中的值 $x$：$m+x$。

# Shifting and Substitution

为了实现 substitution，需要一个叫 shifting 的操作来改变自由变量的编码。

例如对 $[x \mapsto s](\lambda y. x), s = z\ (\lambda w.w)$ 做 substitution 的时候，会变成 $\lambda y. z\ (\lambda w.w)$，此时 λ 的深度多了一层，即所有自由变量的编码都要增加 $1$。

Shifting 函数会用一个 cutoff 参数来控制哪个变量需要被 shift。

> **Definition**: Shifting
>
> The $d$-place shift of a term $t$ above cutoff $c$, written $\uparrow^d_c (t)$, is defined as follows:
>
> $$
> \begin{alignat*}{2}
> &\uparrow^d_c(k) &&=
>     \begin{cases}
>         k & \text{if $k < c$} \\
>         k+d & \text{if $k \ge c$}
>     \end{cases}\\
> &\uparrow^d_c(\lambda. t_1) &&= \lambda. \uparrow^d_{c+1} (t_1) \\
> &\uparrow^d_c(\lambda. t_1\ t_2) &&= \uparrow^d_c(\lambda. t_1)\ \uparrow^d_c(\lambda. t_2)
> \end{alignat*}
> $$
>
> $\uparrow^d_0 (t)$ 可以记作 $\uparrow^d (t)$

> **Definition**: Substitution
>
> The substitution of a term $s$ for variable number $j$ in a term $t$, written $[j \mapsto s]t$, is defined as follows:
>
> $$
> \begin{aligned}
>     &[j \mapsto s]t &&=
>         \begin{cases}
>             s & \text{if $k = j$} \\
>             k & \text{otherwise}
>         \end{cases}\\
>     &[j \mapsto s](\lambda. t_1) &&= \lambda. [j+1 \mapsto \uparrow^1 (s)] (t) \\
>     &[j \mapsto s](\lambda t_1\ t_2) &&= ([j \mapsto s]t_1\ [j \mapsto s]t_2)
> \end{aligned}
> $$

