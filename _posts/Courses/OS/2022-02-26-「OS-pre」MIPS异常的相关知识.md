---
layout:     post
title:      "「OS pre」MIPS异常的相关知识"
subtitle:   关注了SR、Cause、EPC、BadVaddr
date:       2022-02-26
author:     Leo

header-img: ""
catalog: true
tags: ["OS@Tags@Tags", "操作系统@Courses@Series", "BUAA@Schools@Series", "MIPS@Tags@Rags"]
lang: zh
header-style: text
katex: true
---

> **协处理器**：FPA（Floating Point Accelerator，**浮点累加协处理器**，ARM上提供了一组协处理器指令专门实现浮点运算。）就是一个传统的协处理器，有着自己特有的指令集。
>
> 它的指令中保存了OpCode，并且指令区域被分为可以由多达4个协处理器使用。
>
> 对于MIPS来说，它也见将协处理器用于需要管理CPU环境的功能，包括异常的处理，Cahce的控制和内存的管理，它通过这样的分段使得结构更将多样化的同时，也不会影响软件的兼容性。
>
> MIPS中集合这些功能的协处理器称为**CP0**

# CP0

## 相关指令

> 这里关注有关从控制寄存器赋值的单指令

### mtc0

---

**“Move to co-processor zero”**, 写CP0寄存器

[![mtc0.png](https://s4.ax1x.com/2022/02/26/bVldIK.png)](https://imgtu.com/i/bVldIK)

利用通用寄存器的编号来控制一般是不好的（不是每一个都对应，且CP0的寄存器高度专门后），通常是用专用的名称（接下来会说明部分）

### mfc0

----

**“Move from co-processor zero”**读CP0寄存器

[![mfc0.png](https://s4.ax1x.com/2022/02/26/bV89V1.png)](https://imgtu.com/i/bV89V1)

同上，一般更通用的做法是用名字而不是序号

### rfe

---

**“Restore from exception”**，注意其不是“Return from exception”。该指令将状态寄存器恢复到先前进入到trap（我理解比较像错误处理那样的）之前的状态。这个指令与SR相关，从异常返回到用户模式的唯一安全方法是返回一个在延迟槽带有rfe的jr指令。  

## CPU Control Register（部分）（not MMU）

> 这里有一个关于保留字段的说明。 许多未使用的、无功能的控制字段被标记为“0”。 这些字段中的bit保证读为零，并且应该被写为零。 其他保留字段用“reserved”或“×”表示; 软件必须总是将它们写为零，并且不应该假设它将返回零或任何其他特定的值。  

### SR

---

| Name                | Description   | CP0 reg no. |
| ------------------- | ------------- | ----------- |
| SR(status Register) | CPU的模式标志 | 12          |

字段划分如下

[![fieldInSR.png](https://s4.ax1x.com/2022/02/26/bVUPIA.png)](https://imgtu.com/i/bVUPIA)

* **CU3，CU2:**

​	[31:30] 分别控制着co-processor 3和2的是用。在R30xx中，如果软件希望使用BrCond(3:2)输入引脚用作轮询（？有点没看懂），或加速异常解码，这些可能会被启用

> ​	**轮询（Polling）：**一种由CPU决策如何提供给周边设别服务的方式，又称“程控输入输出”（Programmed I/O），轮询法的概念是：由CPU定时发出询问，依序询问每一个周边设备是否需要其服务，有即给予服务，服务结束后再问下一个周边，接着不断周而复始。
>
> *——From 百度百科*
>
> **BrCond3-0*：CPU的输入，直接会被“coprocessor conditional branch’’ instructions.” 测试

* **CU1:**

  1代表用FPA，0代表禁用。当为0时，所有FPA指令都会引起异常。在没有FPA的设备中，它启用时是将BrCond（1）用作轮询输入
  
* **CU0:**
  置为1时，可以在用户模式用一些名义上的特权指令（罕见的情况）。当CPU的控制指令被编码为“co-processor 0”时，无论这一位是啥，都能在内核模式中使用
  
* **RE:**
  “Reverse Endianness”，反向字节顺序（感觉指的时大端法和小端法），假定操作系统软件提供了必要的支持，RE位允许使用一个字节顺序约定运行的二进制文件在使用相反约定的系统中运行。

* **BEV:**
  “Boot  Exception Vectors”, 当置于1时，CPU用ROM(kseg1)的异常入口点。在运行的系统中，一般都是置于0。它重新定位异常向量到RAM地址，加快访问速度，并允许使用“用户提供的”异常服务例程。

* **TS:**
  “TLB Shutdown”，如果一个程序地址同时匹配两个TLB条目，则设置TS。 在某些实现中，在这种状态下的长时间操作可能会导致内部争用和芯片损坏。 TLB Shutdown是终端，只有通过硬件复位才能清除。 在不包括TLB的基本成员中，该位由reset设置; 软件可以依靠这个特性来确定是否存在TLB支持硬件  

* **PE:**
  当缓存发生了奇偶校验错误时，就置它。这个条件不会产生异常，它实际上只对诊断有用。 MIPS体系结构具有缓存诊断功能，因为早期版本的CPU使用外部缓存，这提供了一种方法来验证特定系统的计时。 对于这些实现，缓存奇偶错误位是一个必要的设计调试工具。 

* **CM:**
  显示了最后一次使用D-cache操作的结果。当cache真的储存了寻址内存位置的数据（load击中的时候），将会置于1

* **PZ：**
  与奇偶校验位有关，如今不太使用了

* **SwC，IsC:**
  “Swap Caches” 和 “Isolate （data） Cache”，是一个Cache的模式位，用于管理Cache和诊断，简单来说

  * IsC置1：让所有load和store只访问data cache，绝对不会经过内存。在这种模式下，字的部分储存回事缓存条目失效。注意，当设置了这个位时，即使是未缓存（uncached）的数据也不会在总线上被看到，此外，该位不会通过reset初始化，在依赖外部数据引用之前，启动软件必须确保这个位置被正确初始化。
  * SwC置1：交换I-cache和D-cache的角色，让软件改写I-cache，访问I-cache原先无效的条目

* **IM:**
  “Interrupt Mask”: 8位的字段表现了中断的来源。当它激活的时候，会引起异常。其中的6bits个中断来源是外部的pins，另外两个bits是Cause寄存器中软件可写的中断位。
  任何中断一律平等，没有优先级，硬件也都是统一的处理他们

* **KUc, IEc:**
  两个CPU的保护位，KUc置1时运行时有内核特权，0是用户模式。在内核模式中，软件可以访问整个程序的地址空间，并且用CP0的指令。用户模式中，内存地址被限制在了0x0000_0000~0x7FFF_FFFF，并且运行CP0指令时会被拒绝，如果打破这些规则就会引发错误,而内核模式一般是用来处理中断的，因此直白说它可以表明是否处于中断中。
  IEc置0时，阻止CPU接受任何中断，1则允许

* **KUp,IEP:**
  “KU previous”，“IE previous”。在异常中，硬件会接受KUc和IEc的值并保存，在同时间如果改变了KUc，IEc变成[1, 0]，结合指令*rfe*，可以将KUp和IEp的值拷贝会KUc和IEc

* **KUo,IEo:**
  “KU old”，“IE old”。
  异常时，KUp和IEp的值存在这，结合俩看，6个字节的KU/IE总体来看是一个3层深，2位宽的栈，异常时入栈，rfe时出栈



### Cause寄存器

---

[![causeRegister.png](https://s4.ax1x.com/2022/02/26/bZKLAf.png)](https://imgtu.com/i/bZKLAf)

可以用来确定异常的类型和决定用来处理异常的方式，简单来说就是记录异常的信息

* **BD:**
"Branch Delay"如果置1了，则表明EPC并没有指向实际上导致异常的语句，而时它之前的分支指令（延迟槽相关）
当异常重新开始的点是一条在分支语句后的延迟槽的指令时， EPC必须指向分支指令，否则如果指向延槽的指令的话，CPU从异常处理程序返回后，会破坏了分支。（可能造成该走的分支结果没走了，因此需要返回到branch的那个指令）。
* **CE:**
"co-processor error"，当异常时因为协处理器格式指令针对一个在SR中被CUx位禁止的协处理器，这个2-bit就表示这个报错的协处理器的编号
* **IP:**
"Interrupt Pending"表示当前发生的中断（**但有可能被屏蔽导致无法发出**），这些位跟随六个硬件级别的CPU输入。[8:9]可读可写，并且保存着上次写入的内容。当适当的IM和IEc标志在SR启用时，激活的8位数任何一位都会导致中断，而且没有指示异常发生时发生了什么，而是显示了现在正在发生什么。  。
* ExcCode

​		表示了正在发生异常的种类。

[![ExcCode.png](https://s4.ax1x.com/2022/02/26/bZ8ONq.png)](https://imgtu.com/i/bZ8ONq)



### EPC寄存器

---

32位寄存器存着异常的返回点



### BadVaddr寄存器

---

32位寄存器指向造成如下异常（引用异常）的地址：当用户程序试图访问kuseg之外的地址时，或者当一个地址被错误地对齐到引用的数据大小时，设置任何与mmu相关的异常。  

在任何其他异常之后，这个寄存器是未定义的。 特别要注意的是，它不是在总线错误之后设置的。  

