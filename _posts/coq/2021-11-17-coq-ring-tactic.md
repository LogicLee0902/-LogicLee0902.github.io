---
layout: "post"
title: "「Coq」 Tactic Ring"
subtitle: "环结构的标准化与等价性"
author: "roife"
date: 2021-11-17

tags: ["Coq@Languages@Tags", "Coq Tactics@Tags@Tags", "形式化验证@Tags@Tags"]
lang: zh
catalog: true
header-image: ""
header-style: text
katex: true
---

# `ring`

`ring` 可以处理环结构或半环结构，通过交换性和结合性将其标准化，并处理相关的等价问题。

环的定义：
1. 由 `+` 和 `*` 两种运算组成
2. 集合 $R$ 在 `+` 下构成阿贝尔群（满足结合律和交换律），单位元为 `0`
3. 集合 $R$ 在 `*` 下构成半群，单位元为 `1`
4. `+` 和 `*` 有分配律成立

- 定义 ordered product 为变量的乘积 $V_{i1} \times \dots \times V_{in}$。
- 定义单项式为常数和 ordered product 的乘积（如果常数为 `1` 可以省略）。
- 定义多项式为单项式之和

此时可以根据变量乘积中下标的的字典序定义多项式的 canonical sum。可以证明多项式的 canonical sum 是唯一的，且任意两个 canonical sum 形式相同的多项式等价。

逻辑运算中的 $\wedge$ 和 $\vee$ 也构成了环结构。

# The variables map

在化简到 canonical sum 时，可以将多项式的项映射成一个变量便于处理：

```lisp
(plus (mult (plus (f (5)) x) x)
      (mult (if b then (4) else (f (3))) (2)))
```

```coq
0 |-> if b then (4) else (f (3))
1 |-> (f (5))
2 |-> x
```

然后就可以将其变成变量组成的多项式：

$$
((V_1 \times V_2) + V_2) + (V_0 \times 2)
$$

其对应的 canonical sum 为

$$
(2 × V_0) + (V_1 × V_2) + (V_2 × V_2)
$$

# 用法

- `ring term1 term2 ...`
  : 可以对所有的 term 进行 normalize，`ring` 可以自动推断 term 对应的 type、variables map 以及所使用的 ring theory。

注意，在一个 `ring` tactic 中，所有 term 的 variables map 是共享的。

此外还可以单独使用 `ring` tactic，表示对当前 gaol 两边的多项式进行 normalization，并会尝试用 `congr_eqT` 和 `refl_equal`（$x + y = x + z \Rightarrow y = z$，$x \times z = x \times y \Rightarrow y = z$） 去解决或化简这个 goal。
