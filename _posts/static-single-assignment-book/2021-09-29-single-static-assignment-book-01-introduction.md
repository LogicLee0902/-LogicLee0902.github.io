---
layout: "post"
title: "「SSA Book」 01+02 Introduction & Properties"
subtitle: "Introduction"
author: "roife"
date: 2021-09-29

tags: ["SSA Book@Books@Series", "编译原理@Languages@Tags", "SSA@Tags@Tags", "代码优化@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# SSA 的定义

> A program is defined to be in **SSA** form if each variable is a target of exactly one assignment statement in the program text.

取决于 Def-Use 图的一些性质或者控制流的特性不同，SSA 有很多种类。但是所有 SSA 都有一些共有的特性，比如 **referential transparency** 等。

**Referential transparency** 即每个变量只有一个定义。函数式语言也有 **referential transparency** 的特性，这使得变量的语义只和定义处的 subexpression 有关而与语句的次序无关。

# $\varphi$ 函数

$\varphi$ 函数是 SSA 中处理分支控制流的方式，使得分支的数据可以在交汇点合并到一个变量，因此一般 $\varphi$ 函数会被放在基本块的头部。其形式一般如下：

$$
y \gets \varphi(x_i : B_i)
$$

从基本块 $B_i$ 过来时，取 $y_i \gets x_i$。

需要注意的是一个基本块中的多个 $\varphi$ 函数是并行执行的（这样编译器在放 $\varphi$ 函数的时候就不用关心顺序了）。

$\varphi$ 函数还有一些扩展，例如 $\varphi_{if}$ 函数和 $\gamma$ 函数。

# Def-Use 链

Use 链表示一个变量被使用处的集合，Def 链包含了定义变量处的集合（在 SSA 中是唯一的）。SSA 形式可以简化 DU 链，同时降低维护 DU 链的复杂度。

# Minimality

