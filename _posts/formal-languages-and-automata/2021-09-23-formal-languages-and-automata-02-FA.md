---
layout: "post"
title: "「形式语言」02 FA"
subtitle: "有穷自动机"
author: "roife"
date: 2021-09-23

tags: ["形式语言与自动机@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# FA

有穷状态自动机（finite automata，FA）

$$M = (Q, \Sigma \delta, q_0, F)$$

- $Q$：状态（state）的非空有穷集合
- $\Sigma$：输入字母表（input alphabet）
- $\delta$：状态转移函数（transition function）。$\delta : Q \times \Sigma \rightarrow Q$
  + $\forall (q, a) \in Q \times \Sigma, \delta(q, a) = p$ 表示 $M$ 在状态 $q$ 下读入字符 $a$，则将状态转移到 $p$ 并将读头指向下一个字符串
- $q_0$：开始状态（initial state）。$q_0 \in Q$
- $F$：终止状态（final state）或接受状态（accept state）

用于识别语言时可以用 $\hat{\delta} : Q \times \Sigma^* \rightarrow Q$（后面的 $\delta$ 都是指 $\hat{\delta}$）：
- $\hat{\delta}(q, \varepsilon) = q$
- $\hat{\delta}(q, wa) = \delta(\hat{\delta}(q, w), a)$

## 接受

设 $M = (Q, \Sigma, \delta, q_0, F)$，对于 $\forall x \in \Sigma^*$，如果 $\delta(q_0, x) \in F$，则称 $x$ 被 $M$ 接受。

$$
L(M) = \{x | x \in \Sigma^x \operatorname{\mathtt{and}} \delta(q_0, x) \in F\}
$$

如果 $L(M_1) = L(M_2)$ 则两个 FA 等价。

## 即时描述

设 $M = (Q, \Sigma, \delta, q_0, F)$，$x, y \in \Sigma^*$，$\delta(q_0, x) = q$，则 $xqy$ 称为 $M$ 的一个即时描述（instantaneous description, ID），记作

$$
xq_0ay \vdash_M xayqb
$$

# DFA

如果 $\forall q \in Q, a \in \Sigma$，$\delta(q, a)$ 都有确定的值，则称之为有穷状态自动机（deterministic finite automaton，DFA）。

## DFA 上的等价类划分

定义 $\operatorname{\mathtt{set}}(q) = \\{ x \vert x \in \Sigma^*, \delta(q_0, x) = q\\}$。

在 DFA 上定义关系 $R_M$ 为 $\forall x, y \in \Sigma^*, x R_M y \Leftrightarrow \exists q \in Q, x \in \operatorname{\mathtt{set}}(q) \operatorname{\mathtt{and}} y \in \operatorname{\mathtt{set}}(q)$。

可以发现，$R_M$ 定义了 $\Sigma^*$ 上面的一个等价类。即 DFA 上面的每个状态都对应了 $\Sigma^*$ 的一个划分。

反过来，如果某个语言能划分成有限个等价类，那么就可以考虑用 DFA 描述。

# NFA

非确定性有穷状态自动机（non-deterministic finite automaton，NFA）

$$M =(Q, \Sigma, \delta, q_0, F)$$

- $Q, \Sigma, q_0, F$ 的意义与 DFA 相同
- $\delta: Q \times \Sigma \rightarrow 2^Q$
  + $\forall (q, a) \in Q \times \Sigma, \delta(q, s) = \\{p_1, p_2, \cdots p_m\\}$ 表示 $M$ 在状态 $q$ 下读入字符 $a$，则将状态转移到 $p_i$ 并将读头指向下一个字符串

同样 $\hat{\delta}$ 的定义也需要扩充：$\hat{\delta} : Q \times \Sigma^* \rightarrow 2^Q, q \in Q, w \in \Sigma^*, a \in \Sigma$
- $\hat{\delta}(q, \varepsilon) = \\{q\\}$
- $\hat{\delta}(q, wa) = \\{p \vert \exists r \in \hat{\delta}(q, w) \operatorname{\mathtt{where}} p \in \delta(r, a) \\}$

NFA 与 DFA 的区别在于，输入同一个字符可以有多个不同的转移路径。NFA 将 DFA 中的“值”变成了“集合”，此时可以看作是“拥有智能的”DFA，可以自动选择路径。

## DFA 与 NFA 等价
