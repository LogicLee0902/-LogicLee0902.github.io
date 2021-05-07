---
layout: "post"
title: "「TAPL」 09 Simple Extensions"
subtitle: "More special types"
author: "roife"
date: 2021-05-06

tags: ["Types and Programming Languages@Books@Series", "PKU - 编程语言的设计原理@Courses@Series", "程序语言理论@Tags@Tags", "类型系统@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# Base Types

Base types 指语言中的一些基础类型，也叫 atomic types，例如 Integer/String/Boolean/Float 等。在理论中，通常用 `A` 来指代这些 base types，同时用 $\mathcal{A}$ 表示 base types 组成的集合。

![11-1 Uninterpreted Base Types](/img/in-post/post-tapl/11-1-uninterpreted-base-types.png)

$$
(\lambda f : A \rightarrow A. \lambda x : A. f(f(x))) : (A \rightarrow A) \rightarrow A \rightarrow A
$$

# The `Unit` Type

`Unit` Type 也是一种 Base Type，通常可以在 ML 语言家族中看到。

其特殊之处在于 `Unit` Type 只有唯一一个 value，即 `unit`。

![11-2 Unit Type](/img/in-post/post-tapl/11-2-unit-type.png)

`Unit` type 在带副作用的语言中有很重要的作用。对于带副作用的操作的语句（例如赋值语句），可以用 `Unit` type 作为返回值。这点有点像 C/Java 里面的 `void`。

# Derived Forms: Sequencing and Wildcards

在带副作用的语言中，语句必须要按照特定的顺序来执行。

**Sequencing notation** $t\_1; t\_2$ 可以用来表示先执行 $t\_1$，完成后再执行 $t\_2$。

Sequencing 有两种形式化的定义：

- 定义一条 syntax 规则，两条 evaluation 规则
  + Evaluation rules

    $$
    \frac{t_1 \rightarrow t_1'}{t_1 ; t_2 \rightarrow t_1' ; t_2} \tag{E-Seq}
    $$

    $$
    \operatorname{\mathtt{unit}} ; t_2 \rightarrow t_2 \tag{E-SeqNext}
    $$

  + Typing rule

    $$
    \frac{\Gamma \vdash t_1 : \operatorname{\mathtt{Unit}} \quad \Gamma \vdash t_2 : T_2}{\Gamma \vdash t_1 ; t_2 : T_2} \tag{T-Seq}
    $$

- Derived form

  $$
  t\_1 ; t\_2 \overset{\text{def}}{=} (\lambda x : \operatorname{\mathtt{Unit}}. t\_2)\ t\_1
  $$

  其中 $x$ 是一个 fresh variable（即 $x$ 不在 $t\_2$ 中出现）。由于 call-by-value 的特性，会使得 $t\_1$ 先执行

前者的规则可以从后者推出。

> **Theorem** Sequencing is a derived form（成为 derived form 的条件）
>
> 记 $\lambda^E$ 为带 `Unit` type，`E-Seq`，`E-SeqNext` 与 `T-Seq` 的语言；记 $\lambda^I$ 为只带 `Unit` type 的 STLC。
>
> Let $e \in \lambda^E → \lambda^I$ be the *elaboration function* that translates from $\lambda^E$ to $\lambda^I$ by replacing every occurrence of $t\_1 ; t\_2$ with $(\lambda x : \operatorname{\mathtt{Unit}}. t\_2)\ t\_1$, where $x$ is chosen fresh in each case.
>
> For each term $t$ of $\lambda^E$, we have
>
> - $t \rightarrow_E t'$ iff $e(t) \rightarrow_I e(t')$
> - $\Gamma \vdash^E t : T$ iff $\Gamma \vdash^I e(t) : T$
>
> where the evaluation and typing relations of $\lambda^E$ and $\lambda^I$ are annotated with $E$ and $I$, respectively, to show which is which.

由于 sequencing 的规则可以由导出，所以我们只需要增加 external language 的复杂度，而不增加 internal language 的复杂度，这样使得其相关的定理证明和类型安全证明可以更加简单。这样的 derived forms 被称为**语法糖**（syntax sugar）。

另一种有用的 derived form 是 **wildcard**。即如果某个 abstraction 的参数没有用的话，就没必要为其取一个名字，直接用占位符（wildcard binder） `_` 代替。

- Rules for wildcard
  - Evaluation rule

    $$
    \lambda \_ : T_{11}. t_{12} \rightarrow t_{12} \tag{E-WildCard}
    $$

  - Typing rule

    $$
    \frac{\Gamma \vdash t_2 : T_2}{\Gamma \vdash \lambda \_: T_1. t_2 : T_2}
    $$
- Derived form

  $$
  \lambda \_ : S . t \overset{\text{def}}{=} \lambda x : S. t
  $$

  其中 $x$ 是一个 fresh variable。

# Ascription

![11-3 Ascription](/img/in-post/post-tapl/11-3-ascription.png)

Ascription 不会进行任何额外的运算，会在化简后直接返回原来的值，只用来标记类型。

Ascription 可以用来当作 typing assertions 或者 verifications，如果不成立会被 typechecker 报警。

除此之外，也可以用作：
- documentation：因为会直接返回计算得到的值，所以可以给 subexpressions 用
- 控制类型打印：如果定义了缩写，那么 typechecker 打印类型的时候会尽量使用缩写，但是有时候 typechecker 不能识别出缩写（或者因为其他原因不用缩写），可以用 ascription 声明类型，如 $(\lambda f : \operatorname{\mathtt{Unit}} \rightarrow \operatorname{\mathtt{Unit}}. f)\ \operatorname{\mathtt{as}}\ \operatorname{\mathtt{UU}} \rightarrow \operatorname{\mathtt{UU}};$
- 在 subtyping 声明类型

## Derived forms with multiple steps

Ascription 也是一种 derived form：

$$
t \operatorname{\mathtt{as}} T \overset{\text{def}}{=} (\lambda x : T. x)\ t
$$

这里使用 call-by-value 的特性来实现 evaluation 的效果。

注意，如果不要求 ascription 的返回值一定是一个 value，即使用下面的 evaluation rule，那么就不能直接将 term 当作参数传入 abstraction 了：

$$
t_1 \operatorname{\mathtt{as}} T \rightarrow t_1 \tag{E-AscribeEager}
$$

此时需要改成

$$
t \operatorname{\mathtt{as}} T \overset{\text{def}}{=} (\lambda x : \operatorname{\mathtt{Unit}} \rightarrow T. x\ \operatorname{\mathtt{unit}})\ (\lambda y : \operatorname{\mathtt{Unit}}. t) \quad \text{where $y$ is fresh}
$$

这里使用了 abstraction 阻止自动求值。

唯一的区别是 `E-AscribeEager` 求值只经过了一步，而这里需要两步进行 evaluation。这个也在意料之中，因为 sugering 本来就是为了简化语法的，那么 desugaring 也就有可能增加求值步骤。
要满足前面 derived forms 的条件的话，只要将原条件改成以下形式：

$$t \rightarrow_E t' \quad \text{iff} \quad e(t) \rightarrow^*_I e(t')$$

并且有

$$
\operatorname{if} e(t) \rightarrow_I s, \operatorname{then} s \rightarrow^* e(t') \operatorname{with} t \rightarrow_E t'
$$

# Let Bindings

`let` 可以把一个表达式绑定到一个名字上。例如 $\operatorname{\mathtt{let}} x = t\_1 \operatorname{\mathtt{in}} t\_2$ 表示将 $x$ 绑定到 $t\_1$ 并且用来求值 $t\_2$（evaluate the expression $t\_1$ and bind the name $x$ to the resulting value while evaluating $t\_2$）。其中 $t\_1$ 是 `let`-bound term，$t\_2$ 是 `let`-body。

![11-4 Let Binding](/img/in-post/post-tapl/11-4-let-binding.png)

`let` 使用 call-by-value 的策略，即 `let`-bound term 必须先求值，然后才能对 `let`-body 进行求值。

`let` 也可以定义成一个 derived form：

$$
\operatorname{\mathtt{let}} x = t_1 \operatorname{\mathtt{in}} t_2 \overset{\text{def}}{=} (\lambda x : T_1 . t_2)\ t_1
$$

注意到定义左边的 `let` 中并没有 $t\_1$ 的类型信息，而右边 desurgared 的形式却包含了 $x : T\_1$，说明如果要将 `let` 转换成 internal language，那么必须推导出它的类型信息。即展开 `let` 的过程不能看成对于 term 的 desurgaring 变换，而应该看作是在 typing derivation 上的变换。

$$
\frac{
	\frac{\vdots}{\Gamma \vdash t_1 : T_1}
  \quad
  \frac{\vdots}{\Gamma, x : T_1 \vdash t_2 : T_2}
} {
	\Gamma \vdash \operatorname{\mathtt{let}} x = t_1 \operatorname{\mathtt{in}} t_2 : T_2
} \text{T-Let}
\rightarrow
\frac{
  \frac{
    \frac{\vdots}{\Gamma, x : T_1 \vdash t_2 : T_2}
  }{
    \Gamma \vdash \lambda x : T_1. t_2 : T_1 \rightarrow T_2
  } \text{T-Abs}
  \quad
  \frac{\vdots}{\Gamma \vdash t_1 : T_1}
} {
  \Gamma \vdash (\lambda x : T_1. t_2)\ t_1 : T_2
} \text{T-App}
$$

由此可见 `let`-bindings 是一种比较特殊的 derived form。

> **Q** 能否将 `let`-bindings 的 derived form 定义为
>
>   $$
>   \operatorname{\mathtt{let}} x = t_1 \operatorname{\mathtt{in}} t_2 \overset{\text{def}}{=} [x \mapsto t_1] t_2
>   $$
>
> **A** 不可以。主要的问题在于这个定义无法排除掉一些 ill-typeness：
>
>   $$
>   \operatorname{\mathtt{let}} x = \operatorname{\mathtt{unit}}(\operatorname{\mathtt{unit}}) \operatorname{\mathtt{in}} \operatorname{\mathtt{unit}} \rightarrow [x \mapsto \operatorname{\mathtt{unit}}(\operatorname{\mathtt{unit}})] \operatorname{\mathtt{unit}}
>   $$
>
>   左边的 `let`-binding 显然是 ill-typed，但是右边由于 $\operatorname{\mathtt{unit}}$ 中不存在 $x$，导致类型系统会接受这个 term，导致错误。

# Pairs

![11-5 Pairs](/img/in-post/post-tapl/11-5-pairs.png)

**Pairs** 是一种新的类型，记作 $T\_1 \times T\_2$，称为 **product type** 或者 **cartesian product**。
这里将 pairs 用花括号包裹，实际上一般圆括号用得比较多一些。

Pairs 的规则使得其强制从左到右进行求值（`E-Pair2`），同时只有求值后才能提取其中的元素（`E-PairBeta`）。

# Tuples

**Tuples** 是 $n$ 元的 Pairs。

![11-6 Tuples](/img/in-post/post-tapl/11-6-tuples.png)

其中 $n$ 可以是 $0$，此时 tuple 为 $\\{\\}$。

注意 tuples 也是强制从左到右求值的。

# Records

![11-7 Records](/img/in-post/post-tapl/11-7-records.png)

**Records** 就是加上了 label 的 tuples。这个有点像 `struct`。

可以将 tuples 看作特殊的 records（label 是正整数），将 pairs 看作特殊的 tuples，三个用的也都是同一个符号。但是很多语言将 records 和 tuples 区分开来，因为二者在编译器中的实现不一样。

在很多语言中，records 中元素的顺序并不影响类型，例如 $\\\{a : \operatorname{\mathtt{Nat}}, b : \operatorname{\mathtt{Float}}\\\} = \\\{b : \operatorname{\mathtt{Float}}, a : \operatorname{\mathtt{Nat}}\\\}$。但是这里认为二者是不同的类型。但是第十五章中，通过一个 subtype relation 可以认为二者相同。（是否忽视 ordering 只是一个 taste 的问题，并且二者会对编译器的性能造成影响，这里讨论 ordering records 只是为了多讲一些东西）

## Pattern matching

前面介绍的 records 用了 projection 操作来提取内部的值，但是很多语言都支持使用 pattern matching 来完成这个操作。

这里通过引入 pattern syntax 来将 pattern matching 引入 λ 演算（注意是 untyped）。

![11-8 (Untyped) record patterns](/img/in-post/post-tapl/11-8-untyped-record-patterns.png)



# Sums

![11-9 Sums](/img/in-post/post-tapl/11-9-sums.png)

类似于 pairs 是特殊的 tuples，sums 也是一种特殊的 variants，用来表示多种集合的并集。

