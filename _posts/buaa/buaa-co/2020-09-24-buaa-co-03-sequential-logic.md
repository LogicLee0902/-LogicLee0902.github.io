---
layout: "post"
title: "「BUAA-CO」 03 时序逻辑"
subtitle: "触发器, 时序逻辑, 状态机"
author: "roife"
date: 2020-09-24

tags: ["C「BUAA - Computer Organization」", "B「Digital Design and Computer Architecture」", "BUAA", "计算机组成", "数字电路"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
mathjax: true
---

# 锁存器与触发器

## RS 锁存器

锁存器是利用输出反馈到输入, 并且电路可以形成稳态, 从而做到记忆状态量.
RS 锁存器 (SR Latch) 的 R 端 (reset) 用于复位结果为 0, S 端 (set) 用于设置结果为 1. 两个端口不应该同时置 1.

| $S$ | $R$ | $Q$        |
|-----|-----|------------|
| 0   | 0   | $Q_{prev}$ |
| 0   | 1   | 0          |
| 1   | 0   | 1          |
| 1   | 1   | N/A        |

RS 锁存器可以用或非门或者与非门两种结构搭建, 表达式为 $Q = \overline{R \vert \overline{Q_{prev}}}$ 或者 $Q = \overline{\overline{S} \& \overline{Q_{prev}}}$.

![RS 锁存器](/img/in-post/post-buaa-co/sr-latch.png "SR Latch"){:height="500px" width="500px"}

缺点: RS 锁存器在 S 和 R 端同时生效时输出不能确定, 而且不能与 clk 信号同步, 所以一般要添加一个 enable 端.

## D 锁存器

D 锁存器 (D Latch) 在 SR 锁存器的基础上添加了一个 clk 信号, 用于控制 SR 锁存器的变化.

![D 锁存器](/img/in-post/post-buaa-co/d-latch.png "D Latch"){:height="300px" width="300px"}

| $CLK$ | $D$ | $\overline{D}$ | $S$ | $R$ | $Q$        |
|-------|-----|----------------|-----|-----|------------|
| 0     | X   | X              | 0   | 0   | $Q_{prev}$ |
| 1     | 0   | 1              | 0   | 1   | 0          |
| 1     | 1   | 0              | 1   | 0   | 1          |

D 锁存器避免了 RS 锁存器的非法状态, 同时允许用 clk 信号控制锁存器的状态变化.

缺点: 在一个时钟周期内, D 锁存器的输入信号如果多次变化, 那么输出信号也会变化 (理想是输出结果仅在时钟上升沿改变).

![D 锁存器的局限性](/img/in-post/post-buaa-co/d-latch-limitation.png "D Latch Limitation"){:height="400px" width="400px"}

## D 触发器

D 触发器 (D Flip-Flop) 利用一个 D 锁存器作为缓冲, 实现了输出仅在时钟上升沿改变的效果. 和输入端连接的称为主锁存器 (the master latch), 和输出端连接的称为从锁存器 (the salve latch).

![D 触发器结构](/img/in-post/post-buaa-co/d-flip-flop-internal.png "D Flip-Flop-internal"){:height="200px" width="200px"}

当 clk 为 0 时, 主锁存器随输入信号 D 变化, 从锁存器保持不变; 当 clk 从 0 变为 1 时, 主锁存器停止变化, 同时从锁存器获得主锁存器当前的值 (即时钟上升沿的值).

![D 触发器波形](/img/in-post/post-buaa-co/d-flip-flop-wave.png "D Flip-Flop wave"){:height="400px" width="400px"}

### D 触发器的延迟

由于 D 触发器有两个锁存器, 因此对于信号保持时间有要求.

![D 锁存器延迟](/img/in-post/post-buaa-co/d-flip-flop-timing.png "D Flip-Flop timing"){:height="600px" width="600px"}

- setup time: 时钟上升沿到来前信号至少要保持的时间 (改变主锁存器)
- hold time: 时钟上升沿到来后信号至少要保持的时间 (改变从锁存器)
- "CLK-to-Q" Delay: 时钟上升沿到输出信号改变的时间

## 带使能端的触发器

组合 MUX 和触发器可以实现带使能端的触发器 (一般不在时钟信号上进行门电路操作, 因为可能打乱电路的时序).

![带使能端的触发器](/img/in-post/post-buaa-co/enabled-flip-flop.png "Enabled Flip-Flop"){:height="200px" width="200px"}

## 带复位端的触发器

带复位端的触发器有两种, 同步复位和异步复位.
- 同步复位: 仅在时钟上升沿复位
- 异步复位: 随时都可以复位 (需要改变触发器内部电路, 所以下面只考虑同步复位)

![带复位端的触发器](/img/in-post/post-buaa-co/resettable-flip-flop.png "Resettable Flip-Flop"){:height="200px" width="200px"}

带使能端和复位端的触发器结合使用可以作为寄存器.

## 同步时序逻辑

非同步时序电路可能存在竞争 (racing, 类似于毛刺, 但是在时序电路中可能会导致寄存器状态改变) 的问题, 一般通过在电路中插入寄存器解决.

在同步时序电路中, 要求所有时钟信号相同 (即不能在时钟信号上使用门电路), 并且每一个环路上都有寄存器 (防止竞争).

# 有限状态机

有限状态机 (Finite State Machine, FSM) 是描述状态及转移的数据模型, 由次态逻辑 (next state logic), 寄存器和输出逻辑 (output logic) 组成, 一般可以分为 Moore 型 (输出仅取决于状态) 和 Mealy 型 (输出取决于输入和状态).

![Moore 型和 Mealy 型](/img/in-post/post-buaa-co/moore-mealy-internal.png "Moore machines and Mealy machines"){:height="600px" width="600px"}

## 构造 FSM

1. 状态规划
2. 确定主要状态转移, 并补充其他转移 (不能出现不完备的转移, 未定义的状态一般强制转移到初态)
3. 增加复位信号 (用来确保数字电路被有效初始化和严格同步)
4. 构造次态逻辑 (组合电路)
5. 产生输出信号 (需要决定是 Moore 还是 Mealy)

## 状态编码

状态编码一般有三种方案: 二进制编码 (logN : N), 格雷码 (logN : N) 和独热码 (N : N).
由于 FPGA 的寄存器数量多, 所以一般使用独热码提高电路效率和可靠性.

并且用独热码时可以进行表达式优化, 如 $\overline{R3} \& R2 \& \overline{1} \& \overline{R0}$ 可以优化为 $R2$.

## Moore 与 Mealy

Moore 和 Mealy 的区别在于, Moore 需要等待状态转移完成后才输出结果 (因此会晚一个周期), 而 Mealy 在输入的时候可以直接响应. 并且 Moore 的输出会宽一个周期 (因为状态机的状态改变需要一个周期, Mealy 则可以提前改变).

一般来说 Moore 多了一个起始信号, 所以需要更多的状态. 而在 Mealy 中每一种转移受到输入信号影响, 所以会在转移上的 `/` 后标注输出.

如一个响应 `01` 信号的电路:

![Moore 型和 Mealy 型的区别](/img/in-post/post-buaa-co/moore-mealy.png "Moore and Mealy"){:height="700px" width="700px"}

具体选择 Moore 还是 Mealy 要看对于输出信号时刻的要求.

## 未定义空间

如果状态机设计不完备, 或者发生其他错误, 就有可能导致它进入未定义空间.

# 时序逻辑的时间

# 参考资料

1. Digital Design and Computer Architecture 2nd, Chapter 2
2. Digital Design and Computer Architecture 2nd, Chapter 3
