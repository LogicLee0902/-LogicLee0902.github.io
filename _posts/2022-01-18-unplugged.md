---
layout: "post"
title: "《同构》阅读笔记（正在更新）"
subtitle: "Notes for Unplugged"
author: "roife"
date: 2022-01-18

tags: ["Algebra"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# 自然数

## 皮亚诺自然数公理

1. 0 是自然数
2. 每个自然数都有它的下一个自然数，称为它的后继
3. 0 不是任何自然数的后继
4. 不同的自然数有不同的后继数
5. 如果自然数的某个子集包含 0，并且其中每个元素都有后继元素。那么这个子集就是全体自然数

这五条规则可以用符号表示：
1. $0 \in \mathbb{N}$
2. $\forall n \in \mathbb{N}, \exists n' \in \mathbb{N}$
3. $\forall n' \in \mathbb{N}, n \ne 0$
4. $\forall n, m \in \mathbb{N}, n' = m' \Rightarrow n = m$
5. $\forall S \subseteq \mathbb{N}, (0 \in S \wedge (\forall n \in S \Rightarrow n' \in S)), S = \mathbb{N}$

- 第三条公理用来排除 $\{0 \rightarrow 1, 1 \rightarrow 2, 2 \rightarrow 1\}$ 的情况；
- 第四条公理用来排除 $\{0 \rightarrow 1, 1 \rightarrow 1\}$ 的情况；
- 第五条公理用来排除 $\{0, 0.5, 1, 1.5, \dots\}$ 的情况

```haskell
data Nat = zero | succ Nat
```

## `foldn`

定义 $n = \mathtt{foldn} (\mathtt{zero}, \mathtt{succ}, n)$，即

$$
\mathtt{foldn} (z, f, 0) = z \\
\mathtt{foldn} (z, f, n') = f(\mathtt{foldn} (z, f, n))
$$

`foldn` 可以用于递归运算，利用 Curry-ing 还可以简化掉这个组合子的第三个参数。在利用 `foldn` 进行运算时，有一个技巧：利用元组存储中间值。

例如定义阶乘运算：

$$
c = (0, 1) \\
h(m, n) = (m', m' * n) \\
\mathtt{fact} = \mathtt{2nd} \circ \mathtt{foldn}(c, h)
$$

## 列表

```haskell
data List A = nil | cons(A, List A)
```

可以发现列表和自然数的定义非常相似。可以定义列表的连接操作：

$$
\mathtt{nil} + y = y \\
\mathtt{cons}(a, x) + y = \mathtt{cons}(a, x + y)
$$

同样可以定义 `foldr`，它会从右向左对元素进行操作：

$$
\mathtt{foldr}(c, h, \mathtt{nil}) = c \\
\mathtt{foldr}(c, h, \mathtt{cons}(a, x)) = h(a, \mathtt{foldr}(c, h, x))
$$

用 `foldr` 可以定义 `filter` 与 `map`：

$$
\mathtt{filter}(p) = \mathtt{foldr}(\mathtt{nil}, (p \circ \mathtt{1st} \mapsto \mathtt{cons}, \mathtt{2nd}))
$$

此处的 $\mapsto$ 为麦卡锡条件形式，$(p \mapsto f, g) \Leftrightarrow \mathtt{if}\ p(x)\ \mathtt{then}\ f(x)\ \mathtt{else}\ g(x)$

$$
\mathtt{map}(f) = \mathtt{foldr}(\mathtt{nil}, \mathtt{cons} \circ \mathtt{first}(f)) \\
\mathtt{first}(f, (x, y)) = (f (x), y)
$$

## 二叉树

```haskell
data Tree A = nil | node (Tree A, A, Tree A) -- A  为类型
```

同样可以定义 `foldt`：

$$
\mathtt{foldt}(f, g, c, \mathtt{nil}) = c \\
\mathtt{foldt}(f, g, c, \mathtt{node}(l, x, r)) = g(\mathtt{foldt}(f, g, c, l), f(x), \mathtt{foldt}(f, g, c, r))
$$

其中各个函数的类型为：
$$
\begin{aligned}
& f : A \rightarrow B \\
& \mathtt{foldt} : \mathtt{Tree}\ A \rightarrow B \\
& g : B \rightarrow B \rightarrow B \rightarrow B
\end{aligned}
$$

根据 `foldt` 可以定义 `mapt`：

$$
\mathtt{mapt}(f) = \mathtt{foldt}(f, \mathtt{node}, \mathtt{nil})
$$

利用 `List`，还可以定义多叉树和对应的 `foldm` 运算：

```haskell
data MTree A = nil | node (A, List (MTree A))
```

$$
\mathtt{foldm}(f, g, c, nil) = c \\
\mathtt{foldm}(f, g, c, \mathtt{node}(x, ts)) = \mathtt{foldr}(g(f (x), c), h, ts) \\
h(t, z) = \mathtt{foldm}(f, g, z, t)
$$

# 群

## 群的定义

**群**是一个集合 $G$ 与其上定义的某种二元运算 $\cdot$。它遵循四条公理：
1. 封闭性公理：对任何 $a, b \in G$，运算结果 $a \cdot b \in G$
2. 结合性公理：对任何 $a, b, c \in G$，有 $(a \cdot b) \cdot c=a \cdot (b \cdot c)$
3. 单位元公理：$G$ 中存在一个单位元 $e$，使得对任何 $a \cdot e=e \cdot a=a$
4. 消去公理：对任何 $a \in G$，都存在一个逆元 $a^{−1}$ 使得 $a \cdot a^{−1} = a^{−1} \cdot a = e$

下面的例子都是群：
- 全体整数和加法构成了整数加群
- 全体非零有理数和乘法下构成群
- 所有整数除以 5 的余数构成的集合 $\{0, 1, 2, 3, 4\}$ 与相加后对取 5 取模这个运算构成了**剩余类**（模 $n$ 称为模 $n$ 剩余群）
  + 注意模 $n$ 余数集合 $\{0, 1, 2, \dots n-1, n\}$ 与乘法取模不构成群（因为没有单位元）。但是如果 $p$ 是素数，$\{1, 2, \dots, n-1\}$ 与乘法取模可以构成**模 $n$ 乘法群**。

元素的个数称为群的**阶**。

## 幺半群

群的限制比较严格，可以去掉封闭性和逆元这两条约束，得到幺半群：

**幺半群**是一个集合 $G$ 与其上定义的某种二元运算 $\cdot$。它遵循两条公理：
1. 结合性公理：对任何 $a, b, c \in G$，有 $(a \cdot b) \cdot c=a \cdot (b \cdot c)$
2. 单位元公理：$G$ 中存在一个单位元 $e$，使得对任何 $a \cdot e=e \cdot a=a$

幺半群结构比群更加常见：
- 全体整数和乘法
- 长度有限的字符串和拼接操作
- 长度有限的列表和连接操作

## 半群

继续去掉单位元的约束，可以得到半群：

半群包括一个集合，以及定义在集合的**可结合**的二元运算。

## 群的性质

- 群的单位元是唯一的
- 逆元唯一存在

### 阶

对群中的一个元 $a$，能够满足 $am = e$ 的最小正整数 $m$ 叫做 $a$ 的**阶**。如果不存在这样的 $e$，称 $a$ 的阶是无限的。

根据鸽巢原理，可以证明**有限群的每个元素都有有限的阶**。

### 同态映射

对于映射 $A \xrightarrow{f}b$。如果对于 $a, b \in A$ 和 $f(a), f(b) \in B$ 总有 $f(a) \cdot f(b) = f(a \cdot b)$，则称 $f$ 是**同态映射**。

如果 $f$ 是满射，则称为**同态满射**。

如果 $f$ 是双射，则称为**同构**，记为 $A \cong B$。其中 $A \cong A$ 称为自同构。

### 变换

变换即为一个群的自映射 $\tau : A \rightarrow A$。例如 $A = \{1, 2\}$，变换 $\tau: 1 \rightarrow 2, 2 \rightarrow 1$。

为了方便，记 $a \rightarrow \tau(a) = a^\tau$。

所有的变换可以构成一个新的集合 $S = \{\tau, \lambda, \mu, \dots\}$。定义这个集合上的二元运算为“乘法”，则 $\tau \lambda : a \rightarrow (a^{\tau})^\lambda = a^{\tau \lambda}$。可以发现这种乘法满足结合律。乘法的单位元即为恒等运算 $\epsilon : a \rightarrow a$。

S 本身无法构成群（因为有些变换没有逆元），但是其子集 $G$ 可以构成逆元，其中 $G$ 只包含一一变换（即双射）。变换群一般也不是交换群。

变换群有很多性质：
- 一个集合 A 的所有一一变换构成一个变换群 G
- 任何一个群都同一个变换群同构