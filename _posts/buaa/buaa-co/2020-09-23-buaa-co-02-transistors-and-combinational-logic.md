---
layout: "post"
title: "「BUAA-CO」 02 晶体管和组合逻辑"
subtitle: "晶体管, 门电路, 卡诺图, 组合逻辑"
author: "roife"
date: 2020-09-23

tags: ["C「BUAA - Computer Organization」", "B「Digital Design and Computer Architecture」", "BUAA", "计算机组成", "数字电路"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
mathjax: true
---

# 数字电路

## 电平

一般定义驱动源输出高电压 ($V_{OH} \sim V_{DD}$) 时为高电平, 输出低电压 ($GND \sim V_{OL}$) 为低电平.

对于输入端, $V_{IH} \sim V_{DD}$ 为高电平, $GND \sim V_{IL}$ 为低电平.

电流的传播过程可能会混杂一定的噪声, 因此需要定义部分区域以容纳噪声, 即噪声容限 (noise margin).
如 $V_{IL} \sim V_{IH}$ 之间被设置为禁止区域, 无法预测行为.

## CMOS

MOS 可以作为电子开关使用, 一个 MOS 元件由三部分组成: gate, source, drain.

MOS 可以分为两种, nMOS 和 pMOS.

![mos](/img/in-post/post-buaa-co/mos.png "mos")

其中 nMOS 类似于 NPN 三极管, pMOS 类似于 PNP 三极管. 对于 nMOS, 当在 gate 施加导通电压时, source 和 drain 就可以导通, 电流可以流过. pMOS 恰好相反, 当在 gate 施加高电压时开关关闭.

由于 nMOS 需要 p 型 substrate, pMOS 需要 n 型 substrate, 所以可以将其做到一起, 称为 CMOS.

### 用 CMOS 搭建门电路

例如搭建一个 NAND 门:

![cmos-nand](/img/in-post/post-buaa-co/cmos-nand.png "cmos-nand")

一般来说, pMOS 和 nMOS 网络必然一个串联, 一个并联, 以防止产生短路和浮空状态.

# 逻辑运算

逻辑运算可以用门电路实现.

## 真值表

真值表转布尔表达式有两种方法:

1. Sum of Products (SoP) 看结果为 $1$ 的行: 如 $c = \overline{a} b + \overline{b} a$
2. Product of Sums (PoS) 看结果为 $0$ 的行: 如 $c = (a + b)(\overline{a} + \overline{b})$

## 等值演算

合理利用等值演算可以化简电路, 降低成本.

| Theorem                                                                                 | Dual                                                                                | Name                |
|-----------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|---------------------|
| $B \cdot 1 = 1$                                                                         | $B + 0 = B$                                                                         | Identity            |
| $B \cdot 0 = 0$                                                                         | $B + 1 = 1$                                                                         | Null Element        |
| $B \cdot B = B$                                                                         | $B + B = B$                                                                         | Idempotency         |
| $\overline{\overline{B}} = B$                                                           |                                                                                     | Involution          |
| $B \cdot \overline{B} = 0$                                                              | $B + \overline{B} = 1$                                                              | Complements         |
| $B \cdot C = C \cdot B$                                                                 | $B + C = C + B$                                                                     | Commutativity       |
| $(B \cdot C) \cdot D = B \cdot (C \cdot D)$                                             | $(B + C) + D = B + (C + D)$                                                         | Associativity       |
| $(B \cdot C) + (B \cdot D) = B \cdot (C + D)$                                           | $(B + C) \cdot (B + D) = B + (C \cdot D)$                                           | Distributivity      |
| $B \cdot (B + C) = B$                                                                   | $B + (B \cdot C) = B$                                                               | Covering            |
| $(B \cdot C) + (B \cdot \overline{C}) = B$                                              | $(B + C) \cdot (B + \overline{C}) = B$                                              | Combining           |
| $(B \cdot C) + (\overline{B} \cdot D) + (C \cdot D) = B \cdot C + \overline{B} \cdot D$ | $(B + C) \cdot (\overline{B} + D) \cdot (C + D) = (B + C) \cdot (\overline{B} + D)$ | Consensus           |
| $\overline{B_{0} \cdot B_{1} \cdots} = (\overline{B_{0}} + \overline{B_{1}} + \cdots)$  | $B_{0} + B_{1} = (\overline{B_{0}} \cdot \overline{B_{1}} \cdot \cdots)$            | De Morgan’s Theorem |

## 表达式化简 (卡诺图)

等式化简既可以使用等值演算, 也可以使用卡诺图 (Karnaugh Maps).

![卡诺图画圈](/img/in-post/post-buaa-co/karnaugh-maps.png "karnaugh-maps")

步骤:
1. 作出卡诺图
2. 画圈
3. 将每个圈的表达式写出来, 然后用 or 连接

### 特点

卡诺图的相邻的格子之间只有一位不同 (格雷码), 因此相邻都为 $1$ 时可以在表达式中消去变量.

### 画圈要求

- 圈尽量少, 尽量大, 且只能包含 $1$
- 可以环绕卡诺图的边界 (所以要注意边角的情况)
- 每个圈的边长必须是 $2$ 的整数次幂
- 可以重复圈一个 $1$ (幂等律)

### 无关项

电路中如果有部分输出为无关项, 那么可以将其标记为 `x` (不同于非法值), 在卡诺图中既可以当作 `0` 也可以当作 `1` 使用.

## X 和 Z

除了 `0` 和 `1` 以外, 电路中还有可能出现 `X` 和 `Z`.

- `X` 代表非法值, 如结点同时被 `0` 和 `1` 驱动导致处于禁止区域, 或者电路仿真中未设置初始值.
- `Z` 代表浮空值, 即没有被高电平或低电平驱动 (注意不同于低电平).

# 组合逻辑

## 多路复用器 (multiplexer, mux)



## 译码器 (Decoder)

# 组合逻辑的时序与延迟

## 关键路径与最短路径

## 毛刺

# 参考资料

1. Digital Design and Computer Architecture 2nd, Chapter 1, Chapter 2
