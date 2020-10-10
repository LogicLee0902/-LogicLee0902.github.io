---
layout: "post"
title: "「BUAA-CO」 07 数字模块"
subtitle: "基础数字模块组成"
author: "roife"
date: 2020-10-10
tags: ["C「BUAA - Computer Organization」", "B「Digital Design and Computer Architecture」", "BUAA", "计算机组成", "数字电路", "L「Verilog-HDL」"]
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

先行进位加法器 (Carry-Lookahead Adder, CLA) 将加法电路分块, 同时使得每一块中尽快产生进位信号.

CLA 用产生信号 $G$ 和传播信号 $P$ 来描述进位.
- 在不考虑进位的情况下, 如果某一位必然产生进位, 则发出 $G_i$ 信号, 即 $G_i = A_i B_i$ 
- 在考虑进位的情况下, 如果某一位可以产生进位, 则发出 $P_i$ 信号, 即 $P_i = A_i + B_i$

用 $G_i$ 和 $P_i$ 重写进位逻辑, 可以得到:

$$C_i = G_i + P_i C_{i-1}$$

将其扩展到多块, 定义 $G_{i:j}$ 和 $P_{i:j}$ 为第 $i~j$ 列块的产生信号和传播信号.
- 块产生 $G$ 信号的条件: 当最高有效列发出 $G$ 信号, 或者最高有效列发出 $P$ 信号且前面产生了进位, 则这一块产生进位, 即 $G_{3:0} = G_3 + P_3 (G_2 + P_2 (G_1 + P_1 G_0))$
- 块产生 $P$ 信号的条件: 块中所有为都能产生 $P$ 信号, 即 $P_{3:0} = P_3 P_2 P_1 P_0$.

用 $G_{i:j}$ 和 $P_{i:j}$ 重写进位逻辑, 可以得到:

$$C_i = G_{i:j} + P_{i:j} C_j$$

![32 位 CLA](/img/in-post/post-buaa-co/32-bit-cla.png "32-bit-cla"){:height="500px" width="500px"}

所有的 CLA 块可以同时计算 $G$ 信号和 $P$ 信号. 关键路径从 $G_0$ 开始.

令 $t_pg$ 为产生 $G_i$ 和 $P_i$ 的单个逻辑门的延迟 , $t_{pg\\\_block}$ 为 $k$ 位块中产生 $P_{i:j}$ 和 $G_{i:j}$ 的延迟, $t_{AND\\\_OR}$ 为从 $C_{in}$ 到 $C_{out}$ 到达 $k$ 位 CLA 的最后的 AND/OR 逻辑的延迟, 计算延迟可得:

$$t_{CLA} = t_{pg} + t_{pg\\\_block} + (\frac{N}{k} - 1)t_{AND\\\_OR} + k t_{FA}$$

可以发现, 虽然CLA 快很多, 但是延迟还是线性增长的.

#### 前缀加法器

前缀加法器 (Prefix Adder, PA) 首先计算 2 位的块, 然后计算 4 位的块, 接着计算 8 的块, 直到计算完成.

$$S_i = (A_i \oplus B_i) \oplus C_{i-1}$$

定义 $i = -1$ 表示 $C_{in}$ 的列, 则 $G_{-1} = C_{in}$, $P_{-1} = 0$, $C_{i-1} = G_{i-1:-1}$. 可得

$$S_i = (A_i \oplus B_i) \oplus G_{i-1|-1}$$

接下来要快速计算 $G_{-1:-1}, G_{0:-1}, G_{1:-1}, \dots, G_{N-2:-1}$ 以及 $P_{-1:-1}, P_{0:-1}, P_{1:-1}, \dots, P_{N-2:-1}$. PA 首先用 AND 和 OR 门进行预计算产生 $P_i$ 和 $G_i$ 信号. 然后利用 上部分 $i:k$ 以及下部分 $k-1:j$ 计算 $i:j$ 位的 $G_{i:j}$ 和 $P_{i:j}$.

$$G_{i:j} = G_{i:k} + P_{i:k} G_{k-1:j}$$

$$P_{i:j} = P_{i:k} P_{k-1:j}$$

![32 位 PA](/img/in-post/post-buaa-co/32-bit-pa.png "32-bit-pa"){:height="600px" width="600px"}

N 位前缀加法器的关键路径位 $\log_{2}N$ 步的黑色前缀单元获得所有前缀, 然后 $G_{i-1:-1}$ 通过底部的 XOR 们计算 $S_i$.

令 $t_{pg\\\_prefix}$ 为黑色前缀单元的延迟, 可得:

$$t_{PA} = t_{pg} + \log_{2}N(t_{pg\\\_prefix}) + t_{XOR}$$

可以发现, 前缀加法器的延迟以对数增长, 但是要消耗更多的硬件.

### 延迟

如在 32 位加法中, 假设输入门电路延迟为 $100\mathrm{ps}$, 全加器延迟为 $300\mathrm{ps}$.
- 行波进位加法器延迟为 $32 \times 300\mathrm{ps} = 9.6\mathrm{ns}$
- CLA 中
  + $t_{pg} = 100\mathrm{ps}$
  + $t_{pg\\\_block} = 6 \times 100\mathrm{ps} = 600\mathrm{ps}$
  + $t_{AND\\\_OR} = 2 \times 100\mathrm{ps} = 200\mathrm{ps}$
  + 则 $t_{CLA} = 100\mathrm{ps} + 600\mathrm{ps} + (32/4 - 1) \times 200\mathrm{ps} + (4 \times 300\mathrm{ps}) = 3.3\mathrm{ns}$.
- PA中
  + $t_{pg} = 100\mathrm{ps}$
  + $t_{pg\\\_prefix} = 200\mathrm{ps}$
  + 则 $t_{PA} = 100\mathrm{ps} + \log_{2}(32) \times 200\mathrm{ps} + 100\mathrm{ps} = 1.2ns$

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

| $F_{2:0}$ | 功能        | $F_{2:0}$ | 功能                   |
|-----------|-------------|-----------|------------------------|
| 000       | $A\ AND\ B$ | 100       | $A\ AND\ \overline{B}$ |
| 001       | $A\ OR\ B$  | 101       | $A\ OR\ \overline{B}$  |
| 010       | $A+B$       | 110       | $A-B$                  |
| 011       | 未使用      | 111       | SLT                    |

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

计数器可以直接用加法器和寄存器实现.

![Counter](/img/in-post/post-buaa-co/n-bit-counter.png "n-bit-counter"){:height="250px" width="250px"}

其他类型的计数器 (如 UP/DOWN 计数器, 加载新值的计数器) 都可以据此改造.

```verilog
always @(posedge clk, posedge reset)
    if (reset)  q <= 0;
    else        q <= q + 1;
```

## 移位寄存器

在移位寄存器中, 每个时钟上升沿到来时都会从 $S_{in}$ 中读入一位, 并且其他位向后移动一位.

![Shift Register](/img/in-post/post-buaa-co/shift-register.png "shift-register"){:height="600px" width="600px"}

类似的有并行移位寄存器, 可以一次性读入多位, 并慢慢将其移出.

![Shift Register with parellel load](/img/in-post/post-buaa-co/shift-register-with-parellel-load.png "shift-register-with-parellel-load"){:height="700px" width="700px"}

```verilog
always @(posedge clk, posedge reset)
    if (reset)      q <= 0;
    else if (load)  q <= d;
    else            q <= {q[N-2 : 0], sin};
```

### 扫描链

并行移位寄存器可以用于构建测试电路.
由于电路中寄存器数量过多, 因此不可能为每一个寄存器都提供输入接口, 此时可以用并行移位寄存器一位一位放入数据.

当电路处于工作状态时直接从 D 端读入数据, 忽略扫描链.

当电路处于测试状态时, 利用触发器串行地移入或移出数据.

![可扫描除法器](/img/in-post/post-buaa-co/scannable-flip-flop.png "scannabl-flip-flop"){:height="800px" width="800px"}

# 存储器阵列

## 概述

一个 $N$ 位地址和 $M$ 位数据的阵列有 $2^N$ 行和 $M$ 列, 每行称为一个字.

![通用存储器阵列](/img/in-post/post-buaa-co/generic-memory-array.png "generic-memory-array"){:height="200px" width="200px"}

### 位单元

存储器由位单元 (bit cell) 组成, 每个位单元存储了一位数据.

每个位单元和字线 (wordline) 以及位线 (bitline) 相连. 字线用于控制写入和读取状态, 位线用于存储数据.

读取时, 将位线置于浮空值, 字线置于高电平, 让位单元驱动位线; 写入时, 将位线置于写入值, 字线置于高电平, 使得值被写入.

### 存储器

存储器由译码器和位单元组成, 其中译码器用来控制字线读取数据.

![存储器阵列](/img/in-post/post-buaa-co/memory-array.png "memory-array"){:height="700px" width="700px"}

存储器可以有多个端口进行读写操作, 如图有两个读端口和一个写端口, 地址分别从 $A1 ~ A3$ 读入.

![存储器端口](/img/in-post/post-buaa-co/memory-ports.png "memory-ports"){:height="200px" width="200px"}

存储器可以分为 RAM (Random Access Memory) 和 ROM (Read Only Meomory). 二者区别在于数据是否是易失的.

RAM 可以分为 DRAM 和 SRAM, 前者利用电容充放电, 后者利用交叉耦合反相器.

## RAM

### DRAM

DRAM 利用电容充放电存储数据, 并且使用 nMOS 作为开关.

电容充电到 $V_{DD}$ 时值为 $1$, 放电到 $GND$ 时值为 $0$.

![DRAM](/img/in-post/post-buaa-co/dram.png "DRAM"){:height="200px" width="200px"}

读取时, 数据从电容传输到位线; 写入时, 数据从位线传输到电容. 由于读取会破坏电容的状态, 因此每次读取都必须进行一次重写.

由于电容会慢慢漏电, 所以必须几 ms 刷新一次 (读出后再重写).

### SRAM

SRAM 将数据存储在反相器中, 不需要进行刷新操作. 读写时两个 nMOS 同时打开.

![SRAM](/img/in-post/post-buaa-co/sram.png "SRAM"){:height="200px" width="200px"}

如果噪声干扰了 SRAM 的值, 反相器可以自动恢复.

数字系统中会利用小型 SRAM 来存储临时变量, 这种电路比触发器阵列更加紧凑. 其电路符号和存储器相同.

### 比较触发器/DRAM/SRAM

| 存储器类型 | 晶体管数 | 延迟 |
|------------|----------|------|
| 触发器     | ~20      | 快   |
| SRAM       | 6        | 中等 |
| DRAM       | 1        | 慢   |

DRAM 由于读取后需要刷新值, 因此吞吐量较低. 新技术如 DRAM(SDRAM) 和 (DDR)SDRAM 已经克服了这些问题. SDRAM 使用时钟使得寄存器访问流水线化, DDR 则同时使用时钟的上升沿和下降沿访问寄存器.

## ROM

ROM 用晶体管的存在与否来存储位.

![ROM](/img/in-post/post-buaa-co/rom-bit-cell.png "rom-bit-cell"){:height="250px" width="250px"}

读取时, 位线被缓慢拉伸至 $1$, 然后打开字线, 如果此处有晶体管 (接地) 则位线变为低电平, 否则将保持高电平.
因此 ROM 可以用 “点表示法” 来描述, 有点的地方代表此位为 $1$.

![ROM 点表示法](/img/in-post/post-buaa-co/rom-dot-notation.png "rom-dot-notation"){:height="500px" width="500px"}

ROM 还可以看成是由 AND 和 OR 门电路组合而成的逻辑. 其中 AND 门形成一个译码器, OR 门对数据进行选择, 让字线直接驱动位线.

![门电路 ROM](/img/in-post/post-buaa-co/rom-gate.png "rom-gate"){:height="300px" width="300px"}

### 可编程 ROM

可编程 ROM 分为一次可编程 ROM 和可重复编程 ROM.

一次可编程 ROM 也称熔丝烧断可编程 ROM, 通过选择性熔断熔丝来进行编程. 当熔丝存在时, 晶体管接地, 即值为 $0$, 否则为 $1$.

![熔丝熔断可编程 ROM](/img/in-post/post-buaa-co/fuse-programmable-rom.png "fuse-programmable-rom"){:height="250px" width="250px"}

可重复编程 ROM 分为可擦除 PROM (Erasable PROM, EPROM) , 电子可擦除ROM (Electrically Erasable PROM, EEPROM) 和闪存. 

EPROM 使用浮动栅晶体管代替 nMOS 和熔丝, 不需要物理连接. 对其使用高电平时, 可以产生绝缘体到浮动栅的电子沟道, 使晶体管开启并连接位线到字线. 而当期暴露在强烈紫外线半小时后, 电子会从浮动栅移走, 从而关闭晶体管.

EEPROM 和闪存原理类似, 由电路进行擦除, 不需要紫外线, 并且可以对单元进行单独擦除.

### ROM 实现查找表

ROM 可以用于实现组合逻辑的功能, 这样的存储阵列称为查找表 (Lookable Table, LUT). (因为 ROM 本身可以表示为 AND 和 OR 构建的门电路)

![ROM 实现查找表](/img/in-post/post-buaa-co/memory-array-as-lookup-table.png "memory-array-as-lookup-table"){:height="400px" width="400px"}

## HDL 实现存储器

### RAM

```verilog
reg [M-1 : 0] mem [2**N-1 : 0];

always @(posedge clk)
    if (we)     mem[adr] <= cin;
    
assign dout = mem[adr];
```

### ROM

```verilog
always @(*)
    case (adr)
        2'b00:  dout <= 3'b011;
        2'b01:  dout <= 3'b110;
        2'b10:  dout <= 3'b100;
        2'b11:  dout <= 3'b010;
    endcase
```

# 逻辑阵列

逻辑阵列类似于存储阵列, 可以对门电路进行编程, 分为可编程逻辑阵列 (Programmable Logic Array, PLA) 和现场可编程逻辑门阵列 (Field Programmable Gate Array, FPGA). 前者只能实现两级组合逻辑, 后者可以实现多级组合逻辑和时序逻辑, 并且集成了 I/O, RAM 阵列等功能.

## PLA

PLA 使用 SOP 的方式实现逻辑, 输入驱动 AND 阵列, 产生的蕴含项再驱动 OR 阵列.

![3\*3\*2 位 pla](/img/in-post/post-buaa-co/3-3-2-bit-pla.png "3-3-2-bit-pla"){:height="500px" width="500px"}

ROM 可以看作特殊的 PLA, 区别在于 ROM 的蕴含项总是 $2^N$ 的, 而PLA 不一定.

## FPGA

FPGA 是一个可配置逻辑元件 (Logic Element, LE) 阵列, 也称为可配置逻辑单元 (Configurable Logic Bloc, CLB). 每个 LE 都可以实现组合或时序逻辑功能. 多个 LE 组成了一个逻辑阵列单元 (Logic Array Block, LAB).

LE 被输入/输出元件 (I/O Element, IOE) 包围. IOE 将 LE 和芯片管脚连接, 可以通过编程将 LE 与其他 LE 或 IOE 连接.

![FGPA 结构](/img/in-post/post-buaa-co/fpga-layout.png "fpga-layout"){:height="500px" width="500px"}

## 阵列实现

为了减少尺寸和成本, 一般用 nMOS 或者动态电路实现 ROM 和PLA 而非逻辑门.

在类 nMOS 实现中, 译码器的输出被连接到 nMOS 的门上, 只有下拉 nMOS 网络到 GND 没有路径时, 弱上拉 pMOS 才输出高电平.

下拉 nMOS 被放在“点表示法”中每个没有点的交汇处. 当译码器输出 1 时, 对应的下拉 nMOS 被打开, 输出 0, 而字线上没有 nMOS 的地方由弱 pMOS 拉高至 1.

![类 nMOS 阵列 ROM](/img/in-post/post-buaa-co/rom-pseudo-nmos.png "rom-pseudo-nmos"){:height="600px" width="600px"}

逻辑阵列采用的原理类似.

![3\*3\*2 类 nMOS 阵列 PLA](/img/in-post/post-buaa-co/3-3-2-bit-pla-pseudo-nmos.png "3-3-2-bit-pla-pseudo-nmos"){:height="700px" width="700px"}
