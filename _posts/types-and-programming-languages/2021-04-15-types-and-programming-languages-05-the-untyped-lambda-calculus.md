---
layout: "post"
title: "「TAPL」 05 The Untyped Lambda-Calculus"
subtitle: "无类型 λ 演算"
author: "roife"
date: 2021-04-15

tags: ["Types and Programming Languages@Books@Series", "PKU - 编程语言的设计原理@Courses@Series", "程序语言理论@Tags@Tags", "类型系统@Tags@Tags", "OCaml@Languages@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

Lambda caluclus 是一种描述语言机制的 core language。除此之外，还有其他 calculi：
- pi-calculus: a popular core language for defining the semantics of **message-based concurrent languages**
- object calculus: distills the core features of **object-oriented languages**

传统的 Lambda Calculus 是一个 core language，可以给它添加其他特性构成新的语言。
- the pure untyped lambda-calculus：λ
- the lambda-calculus extended with booleans and arithmetic operations：λNB

# Basics

在 λ 演算中一切都是函数，包括函数的输入值和返回值都可以用函数表示。

λ 演算的中的 term 指的是任意的 term，而其中以 λ 开头的 term 被称为 **lambda-abstractions**。

在 λ 演算中，其语法由三种形式组成：
- 变量：`x`
- 抽象：一个 term `t` 中抽象出一个变量 `x`，记作 $\lambda x.t$
- 应用：将一个 term `t2` 作为参数传入另一个 term `t1`，记作 `t1 t2`

$$
\begin{aligned}
t \Coloneqq & & (\text{terms}) \\
    & x & (\text{variable}) \\
    & \lambda x.t & (\text{abstraction}) \\
    & t\ t & (\text{application}) \\
\end{aligned}
$$

## Abstract and Concrete Syntax

编程语言的语法有两种方式可以表示：一种是通常看到的字符串形式的代码，称为 concrete syntax（surface syntax）；另一种是编译器或者解释器中用树结构（AST）表示的语法，称为 abstract syntax。后者有利于对代码的结构进行操作，因此常在编译器和解释器中使用。

将 concrete syntax 转换为 abstract syntax 的过程共分为两步：
- 通过 lexer 将字符串解析成一个 token 序列，token 表示最小的语法单元，删除空格、注释等内容，并进行简单的转换（如数字数制、字符串等）
- 通过 parser 将 token 序列转换成 AST，这个阶段要处理运算符的结合性等问题

除此之外，一些语言还有其他的解析过程。例如将语言的一些特性定义成更基本的特性的 derived forms。然后定义 internal language（IL）表示一门仅包含了必要的 derived forms 的 core language，同时定义 external language（EL）表示包含了所有 derived forms 的 full language。并且在 parse 的阶段后，将 EL 转换成 IL，从而使得语言的核心更简单。

为了减少冗余括号，本书书写 λ 表达式时会遵从两个规则：
- application 采用左结合，即 `s t u` 表示 `(s t) u`
- abstraction 的 body 部分尽可能得长，如 $\lambda x. \lambda y. x y x$ 表示 $(\lambda x. (\lambda y. (x y) x))$

## Variables and Metavariables

出于习惯，`t` 表示任意的 term，`xyz` 表示任意的 metavariables。

## Scope

当变量 `x` 出现在 abstraction $\lambda x.t$ 的 body 部分时，称其是被**约束的（bound）**，$\lambda x$ 被称为 **binder**。反之，如果变量不被约束，在称为是**自由的（free）**。例如 $(\lambda x.x)x$ 中第一次出现的 `x` 表示一个 binder，第二次出现的 `x` 是 bound 的，第三次出现的 `x` 是 free 的。

如果一个 term 没有自由变量，则称为**封闭的**（closed）。封闭的 term 又被称为**组合子（combinators）**，例如 identify function：

$$
\mathtt{id} = \lambda x.x;
$$

## Operational Semantics

在 pure lambda calculus 中不包含任何数字、运算符等，唯一的运算只有 application。

Application 的步骤为将左侧的 abstraction 中的约束变量替换成右侧的组件。记作

$$
(\lambda x.t_{12}) t_2 \rightarrow [x \mapsto t_2] t_{12}
$$

其中 $[x \mapsto t_2] t_{12}$ 表示将 $t_{12}$ 中所有的自由出现的 $x$ 替换成 $t_2$。例如 $(\lambda x.x) = y$，$(\lambda x.x(\lambda x.x))(u r) => u r (λx.x)$。

类似于 $(\lambda x.t_{12}) t_2$ 的表达式被称为 **redex**（reducible expressions）。Redex 可以用 beta-reduction 进行重写。

## Evaluation strategies

例子：

$$
(\lambda x.x)\ ((\lambda x. x))\ (\lambda z. (\lambda x.x))\ z)) = \mathtt{id}\ (\mathtt{id}\ (\lambda z. \mathtt{id}\ z))
$$

- full beta-reduction：随便选一个进行 reduce（any redex may be reduced at any time.）

  $$
  \begin{aligned}
      & \mathtt{id}\ (\mathtt{id}\ (\lambda z. \underline{\mathtt{id}\ z})) \\
    \rightarrow\ & \mathtt{id}\ (\underline{\mathtt{id}\ (\lambda z. z)}) \\
    \rightarrow\ & \mathtt{id}\ (\lambda z.z) \\
    \rightarrow\ & \lambda z.z
  \end{aligned}
  $$

- normal order strategy：先 reduce 最外面、最左边的 redex。

  $$
  \begin{aligned}
      & \underline{\mathtt{id}\ (\mathtt{id}\ (\lambda z. \mathtt{id}\ z))} \\
      \rightarrow\ & \underline{\mathtt{id}\ (\lambda z. \mathtt{id}\ z)} \\
      \rightarrow\ & \lambda z.\ \underline{\mathtt{id}\ z} \\
      \rightarrow\ & \lambda z.z
  \end{aligned}
  $$

- call by name strategy：call by name 和 normal order strategy 类似，但是它不允许在 abstraction 内部进行 reduce。`Call by name` 即调用的时候不计算值，而是直接传入对应的位置，用到的时候再调用

  $$
  \begin{aligned}
      & \underline{\mathtt{id}\ (\mathtt{id}\ (\lambda z. \mathtt{id}\ z))} \\
      \rightarrow\ & \underline{\mathtt{id}\ (\lambda z. \mathtt{id}\ z)} \\
      \rightarrow\ & \lambda z.\ \underline{\mathtt{id}\ z} \\
  \end{aligned}
  $$

  call-by-name 被很多语言都实现了，比如 Algol60 和 Haskell。而 Haskell 的更加特殊，使用了一个优化过的形式 `call by need`：即当使用的时候才进行 reduce 和 substitute。这样的 reduce 方法使得运行时环境要记录下这个 term 出现的位置（方便实时替换），因此这种 reduction relation 是基于 syntax graph 的，而非 AST。

- call by value strategy：最常用的 redex 策略。一个函数会被 reduce 仅当它的参数已经是一个 value 的时候

  $$
  \begin{aligned}
      & \mathtt{id}\ \underline{(\mathtt{id}\ (\lambda z. \mathtt{id}\ z))} \\
      \rightarrow\ & \underline{\mathtt{id}\ (\lambda z. \mathtt{id}\ z)} \\
      \rightarrow\ & \lambda z.\ \underline{\mathtt{id}\ z} \\
  \end{aligned}
  $$

其中，`normal order strategy` 和 `call by name` 都是 partial evaluation。它们在 reduce 的时候可能函数还没有被 apply。

`Call by value` 是 strict 的，即无论参数有没有用到，都会被 evaluate；反之 `call by name` 和 `call by need` 则只有在用到的时候才计算。

# Programming in the Lambda-Calculus

## Multiple Arguments

λ 演算中的多参数函数是通过高阶函数（higher-order functions）实现的。

假设$s$ 是一个包含自由变量 `x`，`y` 的 term，`f` 是一个参数为 `x`，`y` 的函数：

$$
f = \lambda x. \lambda y. s
$$

$$
\begin{aligned}
f v w & = (f\ v) w \\
& = ((\lambda y.[x \mapsto v]s)w) \\
& = [y \mapsto w][x \mapsto v]s
\end{aligned}
$$

这种参数一个个被 apply 的过程称为 currying。

## Church Boolean

λ 演算中的 boolean 也可以用 λ 表达式表示。其中 `true` 和 `false` 分别是一个接受两个参数的函数，`true` 返回第一个参数，`false` 返回第二个参数。这种表示可以看作是 testing the truth of a boolean value。

### `true` & `false`

$$
\mathtt{tru} = \lambda t. \lambda f. t;
$$

$$
\mathtt{fls} = \lambda t. \lambda f. f;
$$

### `if`

定义一个类似 `if` 的 combinator `test`。在 `test b v w` 中，当 `b` 为 `true` 时返回 `v`，反之返回 `w`。

$$
\mathtt{test} = \lambda l. \lambda m. \lambda n. l\ m\ n;
$$

$$
\begin{aligned}
    &\mathtt{test}\ \mathtt{tru}\ v\ w \\
    =\ & \underline{(\lambda l. \lambda m. \lambda n. l\ m\ n)\ \mathtt{tru}}\ v\ w \\
    \rightarrow\ & \underline{(\lambda m. \lambda n. \mathtt{tru}\ m\ n)\ \mathtt{tru}\ v}\ w \\
    \rightarrow\ & \underline{(\lambda n. \mathtt{tru}\ v\ n)}\ w \\
    \rightarrow\ & \mathtt{tru}\ v\ w \\
    =\ & \underline{(\lambda t. \lambda f. t)\ v}\ w \\
    \rightarrow\ & \underline{(\lambda f. v)\ w} \\
    \rightarrow\ & v
\end{aligned}
$$

### `and` & `or` & `not`

- `and`：如果第一个数是 `tru`，则看第二个数；否则直接返回 `fls`

    $$
    \mathtt{and} = \lambda b.\lambda c.b\ c\ \mathtt{fls}
    $$

    $$
    \mathtt{and2} = \lambda b.\lambda c.b\ c\ b
    $$

- `or`：如果第一个数是 `tru`，则返回 `tru`；否则看第二个数

    $$
    \mathtt{or} = \lambda b.\lambda c.b\ \mathtt{tru}\ c
    $$

    $$
    \mathtt{or2} = \lambda b.\lambda c.b\ b\ c
    $$

- `not`：

    $$
    \mathtt{not} = \lambda b.b\ \mathtt{fls}\ \mathtt{tru}
    $$

示例：

$$
\begin{aligned}
    &\mathtt{and}\ \mathtt{tru}\ \mathtt{tru} \\
    =\ & \underline{(\lambda b. \lambda c.b\ c\ \mathtt{fls})\ \mathtt{tru}\ \mathtt{tru}} \\
    \rightarrow^*\ & \mathtt{tru}\ \mathtt{tru}\ \mathtt{fls} \\
    =\ & \underline{(\lambda t. \lambda f.t)\ \mathtt{tru}\ \mathtt{fls}} \\
    \rightarrow^*\ & \mathtt{tru}
\end{aligned}
$$

## Pair

$$
\mathtt{pair} = \lambda f. \lambda s. \lambda b.b\ f\ s;
$$

$$
\mathtt{fst} = \lambda p.p\ \mathtt{tru};
$$

$$
\mathtt{snd} = \lambda p.p\ \mathtt{fls};
$$

示例：

$$
\begin{aligned}
    &\mathtt{fst}\ (\mathtt{pair}\ v\ w) \\
    =\ &\mathtt{fst}\ (\lambda b.b\ v\ w) \\
    =\ &(\lambda p.\ p\ \mathtt{tru})(\lambda b.b\ v\ w) \\
    =\ &(\lambda b.b\ v\ w)\ \mathtt{tru} \\
    \rightarrow\ &\mathtt{tru}\ v\ w \\
    \rightarrow^*\ &v
\end{aligned}
$$

## Church Numerals

λ 演算中，自然数用 combinator 表示。其中，`s` 和 `z` 分别代表 `succ` 和 `zero`。
其意义为递归对于 `z` 调用 `n` 次 `s`，即 $s^n(z)$。（The number `n` is represented by a function that does something `n` times）

> 感觉在 λ 演算中，对于数据强调的不是如何存储，而是如何去使用它们。所以 `tru` 和 `fls` 对应了程序的选择结构；自然数对应了程序的归纳结构（类似于循环）。从这个角度看，数据和程序可以说有种类似于同构的属性。

$$
\begin{array}{l}
\mathrm{c}_{0}=\lambda s.\lambda z.\mathrm{z}; \\
\mathrm{c}_{1}=\lambda s.\lambda z.\mathrm{s}\ \mathrm{z}; \\
\mathrm{c}_{2}=\lambda s.\lambda z.\mathrm{s}\ (\mathrm{s}\ \mathrm{z}); \\
\mathrm{c}_{3}=\lambda s.\lambda z.\mathrm{s}\ (\mathrm{s}\ (\mathrm{s} \mathrm{z}));
\end{array}
$$

不难发现，$C_0$ 和 $\mathtt{fls}$ 的表示形式相同！

- 求后继数：直接套上一层 `s`（由于是 currying 的形式，所以结果还是 $\lambda s.\lambda z.t$）

    $$
    \mathtt{scc} = \lambda n.\lambda s.\lambda z.s\ (n\ s\ z);
    $$

    $$
    \mathtt{scc2} = \lambda n.\lambda s.\lambda z.\ n\ s\ (s\ z);
    $$

- 求和：`m` 的 `s` 不变，`z` 变成 `n`，意为在 `n` 上应用 `m` 次，即 $s^{n+m}(z) = s^n(s^m(z))$

    $$
    \mathtt{plus} = \lambda m.\lambda n.\lambda s.\lambda z. m\ s\ (n\ s\ z);
    $$

- 乘法：第一个数字的 `s` 变成 `plus n`，意为在 `z` 上调用 `m` 次 `plus n`，即 $s^{nm}(z) = (s^n)^m(z)$

    $$
    \mathtt{times} = \lambda m.\lambda n.m\ (\mathtt{plus}\ n)\ c_0;
    $$

    $$
    \mathtt{times2} = \lambda m.\lambda n.\lambda s.\lambda z.\lambda.m\ (n\ s)\ z;
    $$

    $$
    \mathtt{times3} = \lambda m.\lambda n.\lambda s.m\ (n\ s);
    $$

    其中 `times2` 比较有意思。其中 `n s` 的基数部分（`z`）接受的是上一次加法的结果，这样调用 `m` 次，即执行 `m` 次加法。`times3` 是 `times2` 的化简形式。

- 幂次：

    $$
    \mathtt{power} = \lambda m.\lambda n.\lambda s.n\ (\mathtt{times}\ m)\ c_1;
    $$

    $$
    \mathtt{power2} = \lambda m.\lambda n.\lambda s.\lambda z.n\ (\lambda f.m\ f\ s)\ s\ z;
    $$

    $$
    \mathtt{power3} = \lambda m.\lambda n.n\ m;
    $$

    其中有意思的是 `power2`，可以从 `power` 化简，也可以这么理解：

    考虑现在已经有了

    $$
    g_i = \lambda f'. \lambda z.\underbrace{f' \circ f' \circ \cdots \circ f'}_{m^i\ \text{times}}(z);
    $$

    $$
    m = \lambda f. \lambda z. \underbrace{f \circ f \circ \cdots \circ f}_{m\ \text{times}}(z);
    $$

    令 $m$ 中的每一个 $f$ 都变成 $\lambda z.g_i\ s\ z$，则得到

    $$
    g_{i+1} = \lambda s. \lambda z. m\ (\lambda z'.g_i\ s\ z')\ z;
    $$

    则

    $$
    \begin{aligned}
        g_n & = \lambda s. \lambda z. m\ (\lambda z'.g_{n-1}\ s\ z')\ z \\
            & = \lambda s. m\ (g_{n-1}\ s) \\
            & = \lambda s. m\ ((\lambda s.m\ (g_{n-2}\ s))\ s) \\
            & = \lambda s. m\ (m\ (g_{n-2}\ s)\ s) \\
            & = \lambda s. \underbrace{m\ (m\ (\dots\ s)\ s)}_{n\ \text{times}} \\
            & = \lambda s. n\ (\lambda f.m\ f\ s)
    \end{aligned}
    $$

- `iszero`：对于 $\lambda s. \lambda z. z$ 返回 $\mathtt{tru}$；对于 $\lambda s. \lambda z. s\ z$ 返回 $\mathtt{fls}$。直接令 `z` 返回 `tru`，`s` 返回 `fls`。

  $$
  \mathtt{iszero} = \lambda m. m\ (\lambda x. \mathtt{fls})\ \mathtt{tru};
  $$

- `pred`：求前置，思路比较巧妙。

  $$
  \mathtt{zz} = \mathtt{pair}\ \mathrm{c}_{0}\ \mathrm{c}_{0};
  $$

  $$
  \mathtt{ss} = \lambda p. \mathtt{pair}\ (\mathtt{snd}\ p)\ (\mathtt{plus}\ \mathtt{c}_1\ (\mathtt{snd}\ p));
  $$

  $$
  \mathtt{pred} = \lambda m. \mathtt{fst}\ (m\ ss\ zz);
  $$

  构造序列：$\mathtt{zz} = (0,0) \underbrace{\xrightarrow{\mathtt{ss}} (0,1) \xrightarrow{\mathtt{ss}} (1,2) \xrightarrow{ss} \cdots \xrightarrow{ss}}_{n\ \text{times}}\ (n-1,n)$，恰好执行了 $n$ 次，此时求一个 $\mathtt{fst}$ 即可。

  复杂度为 $O(n)$。

- 减法：利用 `pred` 实现

  $$
  \mathtt{subtract1} = \lambda m. \lambda n. n\ \mathtt{pred}\ m;
  $$

- 相等判断

  $$
  \mathtt{equal} = \lambda m. \lambda n. \mathtt{and}\ (\mathtt{iszero}\ \mathtt{pred}\ m\ n)\ (\mathtt{iszero}\ \mathtt{pred}\ n\ m);
  $$

- 列表：不难发现列表和自然数其实是同构的，因为它们都是递归定义的。其中 `cons` 对应了 `succ`，`nil` 对应了 `zero`。

  列表可以看作一个嵌套的 $\mathtt{pair}$，即 $(c\ x\ (c\ y\ (c\ z\ n)))$。其中 `c` 对应了 `fold` 函数，类似于 `s`，但是它接受两个参数。

  $$
  \mathtt{nil} = \lambda c. \lambda n. n;
  $$

  $$
  \mathtt{cons} = \lambda h. \lambda t. \lambda c. \lambda n. c\ h\ (t\ c\ n);
  $$

  $$
  \mathtt{head} = \lambda l. l\ (\lambda h. \lambda t. h)\ \mathtt{fls} = \lambda l. l\ \mathtt{tru}\ \mathtt{fls};
  $$

  $$
  \mathtt{isnil} = \lambda l. (\lambda h. \lambda t. \mathtt{fls})\ \mathtt{tru};
  $$

  $$
  \begin{aligned}
  \mathtt{tail} = \lambda l. \mathtt{fst}\ (l \\
  & (\lambda x. \lambda p. \mathtt{pair}\ (\mathtt{snd}\ p) (\mathtt{cons}\ x\ (\mathtt{snd}\ p))) \\
  & (\mathtt{pair}\ \mathtt{nil}\ \mathtt{nil}));
  \end{aligned}
  $$

  `tail` 的思路类似于 `pred`：$(\mathtt{nil}, \mathtt{nil}) \rightarrow (\mathtt{nil}, \mathtt{cons}\ a\ \mathtt{nil}) \rightarrow (\mathtt{cons}\ a\ \mathtt{nil}, \mathtt{cons}\ b\ (\mathtt{cons}\ a\ \mathtt{nil})) \rightarrow \dots \rightarrow \ (\mathtt{tail_e}, \mathtt{list_{reversed}})$

  除此之外，还有另一种构建方法：

  $$
  \mathtt{nil} = \mathtt{pair}\ \mathtt{tru}\ \mathtt{tru};
  $$

  $$
  \mathtt{cons} = \lambda h. \lambda t. \mathtt{pair}\ \mathtt{fls}\ (\mathtt{pair}\ h\ t);
  $$

  $$
  \mathtt{head} = \lambda z.\mathtt{fst}\ (\mathtt{snd}\ z);
  $$

  $$
  \mathtt{head} = \lambda z.\mathtt{snd}\ (\mathtt{snd}\ z);
  $$

  $$
  \mathtt{isnil} = \mathtt{nil}
  $$

## Enriching the Calculus

前面在 λ 演算中定义了布尔型和自然数，理论上已经可以构建出所有的程序了。但是为了简洁，这里开始会使用 λNB 作为系统表述，即将前面 untyped arithmetic expression 的内容加进来，将其看作 primitive 的存在。二者可以轻松地进行转换：

$$
\mathtt{realbool} = \lambda b. \mathtt{true}\ \mathtt{false}; \Leftrightarrow \mathtt{churchbool} = \lambda b. \mathtt{if}\ b\ \mathtt{then}\ \mathtt{tru}\ \mathtt{else}\ \mathtt{fls};
$$

$$
\mathtt{realnat} = \lambda m. m\ (\lambda x. \mathtt{succ}\ x)\ 0;
$$

$$
\mathtt{realeq} = \lambda m. \lambda n. (\mathtt{equal}\ m\ n\ \mathtt{true}\ \mathtt{false});
$$

注意 `succ` 本身的语法结构，不能对 church numerals 使用。

使用 λNB 的一个原因是 Church Numerals 的表示和运算太繁杂了，尽管结果和普通的运算等价，但是中间过程却很复杂，并且会影响到求值顺序。如果采用 call-by-value 的方法，那么对于 church numerals 来说不能提前化简数字（因为没有 apply `s` 和 `z`），此时 `scc c1` 和 `c2` 的形式有很大差别。

## Recursion

前面提到 normal forms 指的是无法继续化简的式子，但是有些 term 是没有 normal form 的，被称为 **diverge**。

**omega** 是一个一个 divergent combinator：

$$
\mathtt{omega} = (\lambda x.x\ x)\ (\lambda x.x\ x);
$$

虽然它只有一个 redex，但是进行 reduce 后发现又得到了一个和原式相同的 `omega`。

`omega` 有一个 generalization 的形式，被称为 **fixed-point combinator**，也叫 **call-by-value Y-combinator** 或者 **Z**：

$$
\mathtt{fix} = \lambda f. (\lambda x. f\ (\lambda y. x\ x\ y))\ (\lambda x. f\ (\lambda y. x\ x\ y))
$$

除此之外，`fix` 还有一种更简单的形式，但是无法在 call-by-value 中使用：

$$
Y = \lambda f. (\lambda x. f\ (x\ x))\ (\lambda x. f\ (x\ x))
$$

例如一个阶乘函数：

$$
g = \lambda f. \lambda n. \mathtt{if}\ \mathtt{realeq}\ n\ \mathtt{c}_0\ \mathtt{then}\ \mathtt{c}_1\ \mathtt{else}\ (\mathtt{times}\ n\ (f\ (\mathtt{prd}\ n)));
$$

$$
\mathtt{factorial} = \mathtt{fix}\ g;
$$

