---
layout: "post"
title: "「BUAA-CO」 07 数字模块"
subtitle: "基础数字模块组成"
author: "roife"
date: 2020-10-10
tags: ["C「BUAA - Computer Organization」", "B「Digital Design and Computer Architecture」", "BUAA", "计算机组成", "数字电路"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
mathjax: true
---

# 算术电路

## 加法

### 半加器与全加器

半加器是能执行一位加法的器件, 其中输入为 $A$, $B$, 输出为 $S$, $C_{out}$ 表示进位.

$$S = A \oplus B$$

$$C_{out} = AB$$

![半加器](/img/in-post/post-buaa-co/half-adder.png "half-adder"){:height="200px" width="200px"}

全加器相对于半加器多了一个输入 $C_{in}$ 表示接收的进位.

$$S = A \oplus B \oplus C$$

$$C_{out} = AB + AC_{in} + BC_{in}$$

![半加器](/img/in-post/post-buaa-co/half-adder.png "half-adder"){:height="200px" width="200px"}

### 进位传播加法器

进位传播加法器 (Carry Propagate Adder, CPA) 能对两个 $n$ 位数进行加法运算.

#### 行波进位加法器

行波进位加法器 (ripple-carry adder) 由加法器简单地连接得到, 速度较慢, 延迟线性增长.

![行波进位加法器](/img/in-post/post-buaa-co/ripple-carry-adder.png "ripple-carry-adder"){:height="400px" width="400px"}

#### 先行进位加法器

先行进位加法器 (Carry-Lookahead Adder, CLA) 将加法电路分块, 

## 减法

减去一个数即加上其补码, 所以 $A - B = A + \overline{B} + 1$.
在CPA中将输入 $B$ 取反, 并使 $C_{in} = 1$ 即可计算减法.


![减法器](/img/in-post/post-buaa-co/subtractor.png "subtractor"){:height="250px" width="250px"}

## 比较器

比较器分为相等比较器 (equality comparator) 和量值比较器 (magnitude comparator).

其中相等比较器直接利用异或门搭建, 可以比较两个数字是否相等.

![相等比较器](/img/in-post/post-buaa-co/equality-comparator.png "equality-comparator"){:height="500px" width="500px"}

量值比较器首先计算 $A - B$, 然后根据符号位判断关系.

![量值比较器](/img/in-post/post-buaa-co/magnitude-comparator.png "magnitude-comparator"){:height="250px" width="250px"}

## ALU

算术逻辑单元 (arithmetic/logical unit, ALU) 是多种运算 (加, 减, 比较, AND, OR) 的综合, 并用一个 $F$ 端口进行控制.

![ALU 符号](/img/in-post/post-buaa-co/alu.png "ALU"){:height="250px" width="250px"}

| $F_{2:0}$ | 功能      | $F_{2:0}$ | 功能                 |
|-----------|-----------|-----------|----------------------|
| 000       | $A AND B$ | 100       | $A AND \overline{B}$ |
| 001       | $A OR B$  | 101       | $A OR \overline{B}$  |
| 010       | $A+B$     | 110       | $A-B$                |
| 011       | 未使用    | 111       | SLT                  |

设计电路时可以利用编码的特性, 即 $F$ 的最后一位决定了输入 $B$ 的正负.

![ALU](/img/in-post/post-buaa-co/n-bit-alu.png "n-bit-alu"){:height="400px" width="400px"}

## 移位器和循环移位器

移位器一般有三种:
- 逻辑移位器: 逻辑左移 (LSL) 和逻辑右移 (LSR) 用 0 填补空位
- 算术移位器: 算术右移 (ASR) 时会用最高位填补 MSB
- 循环移位器: 将一端的数字填补到另一端

移位器可以用 n 个 n:1 的MUX 实现.

![Shifters](/img/in-post/post-buaa-co/4-bit-shifters.png "4-bit-shifters"){:height="700px" width="700px"}

![Rotators](/img/in-post/post-buaa-co/4-bit-rotators.png "4-bit-rotators"){:height="500px" width="500px"}

## 乘法

乘法是 AND, 移位和加法的综合.

如图, 第 $i$ 行为 $B_i\ AND(A_3, A_2, A_1, A_0)$.

![Multiplier](/img/in-post/post-buaa-co/4-bit-multiplier.png "4-bit-multiplier"){:height="800px" width="800px"}

## 除法
```algorithm
R' = 0
for i = N−1 to 0
    R = {R' << 1, Ai}
    D = R − B
    if D < 0 then   Qi = 0, R' = R  // R < B
    else            Qi = 1, R' = D  // R >= B
R = R'
```

$n$ 除法器阵列需要 $n^2$ 个除法器实现. 除法器中的 $N$ 端信号表示 $R - B$ 是否为负数, 可以从阵列每一行最左端的 $D$ 输出得到, 即差的符号位.

![Divider](/img/in-post/post-buaa-co/4-bit-divider.png "4-bit-divider"){:height="500px" width="500px"}

由于 $n$ 为除法器阵列的延迟以 $n^2$ 增长, 因此除法非常慢.

# 时序电路

## 计数器

## 移位寄存器


