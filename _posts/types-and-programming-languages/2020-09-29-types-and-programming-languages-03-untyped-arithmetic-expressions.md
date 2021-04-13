---
layout: "post"
title: "「TAPL」 03 Untyped Arithmetic Expressions"
subtitle: "无类型算术表达式"
author: "roife"
date: 2020-10-04

tags: ["Types and Programming Languages@Books@Series", "PKU - 编程语言的设计原理@Courses@Series", "程序语言理论@Tags@Tags", "函数式编程@Tags@Tags", "类型系统@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# Introduction

用类似于 BNF 的 grammar 定义一个简单的语言：

## [Terms, Grammarly]

$$
\begin{aligned}
t \Coloneqq & & (\text{terms}) \\
    & \mathtt{true} & (\text{constant}\ true) \\
    & \mathtt{false} & (\text{constant}\ false) \\
    & \mathtt{if}\ t\ \mathtt{then}\ t\ \mathtt{else}\ t & (\text{conditions}) \\
    & \mathtt{0} & (\text{constant}\ 0) \\
    & \mathtt{succ}\ t & (\text{successor}) \\
    & \mathtt{pred}\ t & (\text{predecessor}) \\
    & \mathtt{iszero}\ t & (\text{zero}\ test)
\end{aligned}
$$

t 是一个 meta-variable（相当于一个 placeholder）。

这里定义的语法可能会生成一些 nonsensical 的程序，例如：`succ 0`，后面需要通过类型系统排除。

这里没有提到括号的用法，但是为了看起来清楚一点，写的时候会加上括号，如 `iszero (pred (succ 0)) → true`。

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

> 关于 Influence Rules，需要注意的是
> 1. 如果一个 rule 没有 premises，那么被称为 axioms
> 2. 由于这里的 premises 和 conclusion 里面有 metavariables，所以严谨地说它们是 rule schemas

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

> 可以证明 $S_i \subseteq S_{i+1}$。

## Proposition: $\mathcal{T} = S$

由于 $\mathcal{T}$ 是满足 inductively 定义条件的最小集合，所以只要证明 $S$ 也是满足 inductively 定义的最小集合即可。即证明：
- $S$ 满足 inductively 定义的条件
- 任何满足 inductively 定义的集合都包含 $S$ (即 $S$ 是满足条件的最小集合)

第一点证明显然。对于第二点证明，可以先证 $S_i \subset S'$ (归纳)。

# Induction on terms

根据 $\mathcal{T} = S$，可以递归定义出一些关于集合 $S$ 的函数，也可以对于 terms 的命题进行递归证明。

## $\operatorname{Consts}(t)$

$\operatorname{Consts}(\mathtt{t})$: $\mathtt{t}$ 中出现的常量的集合.

$$\operatorname{Consts}(\mathtt{true}) = \{\mathtt{true}\}$$

$$\operatorname{Consts}(\mathtt{false}) = \{\mathtt{false}\}$$

$$\operatorname{Consts}(\mathtt{0}) = \{\mathtt{0}\}$$

$$\operatorname{Consts}(\mathtt{succ}\ t_1)  = \operatorname{Consts}(\mathtt{t_1})$$

$$\operatorname{Consts}(\mathtt{pred}\ t_1)  = \operatorname{Consts}(\mathtt{t_1})$$

$$\operatorname{Consts}(\mathtt{iszero}\ t_1)  = \operatorname{Consts}(\mathtt{t_1})$$

$$\operatorname{Consts}(\mathtt{if}\ t_1\ \mathtt{then}\ t_2\ \mathtt{else}\ t_3) = \operatorname{Consts}(t_1) \cup \operatorname{Consts}(t_2) \cup \operatorname{Consts}(t_3)$$

## $\operatorname{Size}(t)$

$\operatorname{Size}(\mathtt{t})$: $\mathtt{t}$ 的大小, 可以看作是语法树中的节点个数

$$\operatorname{Size}(\mathtt{true}) = 1$$

$$\operatorname{Size}(\mathtt{false}) = 1$$

$$\operatorname{Size}(\mathtt{0}) = 1$$

$$\operatorname{Size}(\mathtt{succ}\ t_1)  = \operatorname{Size}(\mathtt{true}) + 1$$

$$\operatorname{Size}(\mathtt{pred}\ t_1)  = \operatorname{Size}(\mathtt{true}) + 1$$

$$\operatorname{Size}(\mathtt{iszero}\ t_1)  = \operatorname{Size}(\mathtt{true}) + 1$$

$$\operatorname{Size}(\mathtt{if}\ t_1\ \mathtt{then}\ t_2\ \mathtt{else}\ t_3) = \operatorname{Size}(t_1) + \operatorname{Size}(t_2) + \operatorname{Size}(t_3) + 1$$

## $\operatorname{depth}(t)$

$\operatorname{depth}(\mathtt{t})$: 语法树的深度

$$\operatorname{depth}(\mathtt{true}) = 1$$

$$\operatorname{depth}(\mathtt{false}) = 1$$

$$\operatorname{depth}(\mathtt{0}) = 1$$

$$\operatorname{depth}(\mathtt{succ}\ t_1)  = \operatorname{depth}(\mathtt{true}) + 1$$

$$\operatorname{depth}(\mathtt{pred}\ t_1)  = \operatorname{depth}(\mathtt{true}) + 1$$

$$\operatorname{depth}(\mathtt{iszero}\ t_1)  = \operatorname{depth}(\mathtt{true}) + 1$$

$$\operatorname{depth}(\mathtt{if}\ t_1\ \mathtt{then}\ t_2\ \mathtt{else}\ t_3) = \max \left( \operatorname{depth}(t_1) + \operatorname{depth}(t_2) + \operatorname{depth}(t_3) \right) + 1$$

> **Proposition**: $\mid Consts(t) \mid \leq size(t)$
>
> 分为 3 个 cases
> - $t$ 是 constants：
>
>   $$
>   | \operatorname{Consts}(t) | = | \{ t \} |  = 1 = \operatorname{size}(t)
>   $$
>
> - $t$ 是 $\mathtt{succ}\ t_1$ 或 $\mathtt{pred}\ t_1$ 或 $\mathtt{iszero}\ t_1$
>
>   $$
>   | \operatorname{Consts}(t) | = | \operatorname{Consts}(t_1) | \le \operatorname{size}(t_1) < \operatorname{size}(t)
>   $$
>
> - $t$ 是 $\mathtt{if}\ t_1\ \mathtt{then}\ t_2\ \mathtt{else}\ t_3$：
>
>   $$
>   \begin{aligned}
>         | \operatorname{Consts}(t) | = &\ | \operatorname{Consts}(t_1) \cup \operatorname{Consts}(t_2) \cup \operatorname{Consts}(t_3) | \\
>         \leq &\ | \operatorname{Consts}(t_1) | + | \operatorname{Consts}(t_2) | + | \operatorname{Consts}(t_3) | \\
>         \leq &\ \operatorname{size}(t_1) + \operatorname{size}(t_2) + \operatorname{size}(t_3) \\
>         < &\ \operatorname{size}(t)
>   \end{aligned}
>   $$

## Theorem: Principles of induction on terms

Suppose $P$ is a predicate on terms.

Structural induction:

- If, for each term s,
  + given $P(r)$ for all immediate subterms $r$ of $s$
  + we can show $P(s)$
- then $P(s)$ holds for all $s$

# Semantic Styles

- Syntax（语法）：程序的结构
- Semantic（语义）：程序的值

## Operational semantics

Operational semantics 分为 small-step 和 big-step。

Operational Semantics 用一个 **abstract machine** 来定义程序的语义，machine 的每一个状态都是一个 term，状态之间用 transition function（对 $t$ 进行 simplification，或者终止程序）进行转移。machine 一开始的状态是 $t$，终止状态为 $t$ 的值。

通常一种语言会有多种 operational semantics，一种是贴近程序员的，另一种是贴近解释器或编译器的。证明这两种语义相同相当于证明语言实现的正确性。

## Denotational semantics

Devotional semiotics 将 term 看作是数学上的实体。

Denotational Semantics 首先确定了一个 semantics domains，然后定义一个 interpretation function 将 term 映射到 semantics domains。（domain theory）

Denotational Semantics 好处在于可以突出语言的核心概念，并且 semantics domain 可以导出许多 laws，比如用来确定两个程序是否相同。

## Axiomatic semantics

以上两种 semantics 都是先定义语言的行为，然后导出一些 laws。而 Axiomatic Semantics 将 laws 本身作为程序的定义。（Hoare Logic）

# Evaluation

## Booleans

![3-1 Booleans(B)](/img/in-post/post-tapl/3-1-booleans.png)

如图定义了 Booleans 的 syntax 和 semantics（即右边定义的 evaluation reduction，在 big-step 中称为 evaluation relation）。

前两条规则比较简单，重要的是第三条规则 `E-If`，它表明 `if` 语句的 guard `t_1` 可以被化简。而对于 `then` 分支和 `else` 分支不存在类似的规则，即必须从外到内化简。例如，不允许：

$$
\dfrac
{
  \operatorname{\mathtt{if}}\ \operatorname{\mathtt{true}}\ \operatorname{\mathtt{then}}\ (\operatorname{\mathtt{if}}\ \operatorname{\mathtt{false}}\ \operatorname{\mathtt{then}}\ \operatorname{\mathtt{false}} \operatorname{\mathtt{else}}\ \operatorname{\mathtt{false}}
  )
  \operatorname{\mathtt{else}}\ \operatorname{\mathtt{true}}
}
{
  \operatorname{\mathtt{if}}\ \operatorname{\mathtt{true}}\ \operatorname{\mathtt{then}}\ \operatorname{\mathtt{false}}\ \operatorname{\mathtt{else}}\ \operatorname{\mathtt{true}}
}
$$

`E-If` 的作用是决定哪里需要化简，而 `E-IfThen` 和 `E-IfElse` 真正进行了化简工作，所以前者称为 congruence rule，后者称为 computation rules。

> **Definitions**: Rules
>
> - An instance of an inference rule is obtained by consistently replacing each metavariable by the same term in the rule’s conclusion and all its premises (if any).
> - A rule is satisfied by a relation if, for each instance of the rule, either the conclusion is in the relation or one of the premises is not.

> **Definition**: One-step evaluation relation $\rightarrow$
>
> - The one-step evaluation relation $\rightarrow$ is the **smallest** binary relation on terms.
> - When the pair $(t,t')$ is in the evaluation relation, we say that “the evaluation statement (or judgment) $t \rightarrow t'$ is **derivable**.”

An evaluation statement $t \rightarrow t'$ is derivable iff there is a derivation tree with $t \rightarrow t'$ as the label at its root.

在一颗 derivation tree 上，`E-If` 是 internal nodes，`E-True` 和 `E-False` 是 leaf nodes。

>**Theorem**: Determinacy of one-step evaluation
>
>If $t \rightarrow t'$ and $t \rightarrow t''$, then $t' = t''$.

## Normal Forms

> **Definition**: Normal Form
>
> A term $t$ is in **normal form** if no evaluation rule applies to it.

显然，`true` 和 `false` 是 normal forms，因为没有针对它们的求解规则。

> **Theorem**: Every value is in normal form.

> **Theorem**: If $t$ is in normal form, then $t$ is a value.
>
> **注解**：只对当前的系统成立，对于后面的系统不一定成立

所有的 values 都是 normal form，但是 normal form 不一定都是 values。

> **Definition**: Multi-step evaluation relation $\rightarrow^*$
>
> The multi-step evaluation relation $\rightarrow^∗$ is the reflexive, transitive closure of one-step evaluation. That is, it is the smallest relation such that
> - if $t \rightarrow t'$ then $t \rightarrow^* t'$
> - $t \rightarrow^* t$ for all $t$
> - if $t \rightarrow^* t'$ $t' \rightarrow^* t''$, then $t \rightarrow t''$

> **Theorem**: Uniqueness of normal forms
>
> If $t \rightarrow^* u$ and $t \rightarrow^* u'$, where $u$ and $u'$ are both normal forms, then $u = u'$.

> **Theorem**: Termination of Evaluation
>
> For every term $t$ there is some normal form $t'$ such that $t \rightarrow t'$

## Arithmetic expressions

![](/img/in-post/post-tapl/3-2-arithmetic-expressions.png)

`succ(true)` 是非法的。

在计算 `pred (succ (pred 0))` 时，不能直接应用 `E-PredSucc`，因为其参数为 `nv`，而 `pred 0` 并非 `nv`。所以必须用 derivation tree。

$$
\frac{
  \frac{
    \frac{\quad}{\operatorname{pred} 0 \rightarrow 0} \text { E-PredSucc }
  } {
    \operatorname{succ} (\operatorname{pred} 0) \rightarrow \operatorname{succ} 0}{\text { E-Succ }
  }
}{
  \operatorname{pred}(\operatorname {succ}(\operatorname { pred } 0)) \rightarrow \operatorname { pred }(\operatorname{succ} 0)} \text { E-Pred }
$$

> **Definition**: Stuckness
>
> A closed term is **stuck** if it is in normal form but not a value.
>
> **注解**：Stuckness 意味着 operational semantics 遇到了 runtime errors。在这里表现为化简不下去。例如 `succ true`




