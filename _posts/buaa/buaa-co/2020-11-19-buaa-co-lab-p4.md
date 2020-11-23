---
layout: "post"
title: "「BUAA-CO Lab」 P4 单周期 CPU (Verilog 实现)"
subtitle: "单周期 CPU (Verilog)"
author: "roife"
date: 2020-11-19

tags: ["C「BUAA - Computer Organization」", "BUAA", "计算机组成", "L「Verilog-HDL」"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
mathjax: true
---

# 课下总结

**一定要多做测试**

直接翻译 P3 电路就可以了, 会出问题的地方基本上是 Verilog 的问题.

本人在连接的过程中遇到的几个问题分享一下：
1. 注意阻塞赋值和非阻塞赋值的问题!
2. Debug 的时候可以强行进行综合, 然后看看编译器挑了哪些错, 注意 Warnings
3. 学长说考题还是跳转+计算+访存, 或者是跳转+简单计算+复杂计算. 可以试试看英文指令集中比较难的几个指令
4. 准备 (或者白嫖) 一个自动化测试工具.
5. 注意 verilog 中如果打了 typo, 可能会被编译器认为是一条 wire, 但是慎用 ``default net_type none` , 会出现奇怪的 bug, 建议仔细看 Warnings 就好

加指令步骤 (类似 p3): 考虑要用哪些部件 (和 r/i/j/b/l 比较一下, 类似于哪个), 然后把数据通路连上. 可以先在各个元件里完成各自要做的功能, 然后再连全局的数据通路. 最后在 CU 里添加控制信号.

# 课下实现

电路类似于 P3 分成了 IFU, NPC, EXT, CMP, DM, GRF, CU, 主电路.

有一点小更改就是 PC + IM = IFU, 然后多了一个 CMP 元件接到 NPC 上, 用于处理所有branch 指令的情况.

DM 的设计也不太一样, 可以直接这么写:

```verilog
// 读取
assign DMout = (DMType == `DM_w) ? `word :
               (DMType == `DM_h) ? sign_ext16(`half) :
               (DMType == `DM_hu) ? unsign_ext16(`half) :
               (DMType == `DM_b) ? sign_ext8(`byte) :
               (DMType == `DM_bu) ? unsign_ext8(`byte) :
               32'b0;
// 写入
if (DMType == `DM_w) `word <= WD;
else if (DMType == `DM_h) `half <= WD[15 + 16 * addr[1:1] -:16];
else if (DMType == `DM_b) `byte <= WD[7 + 8 * addr[1:0] -:8];
```

## Debug 工具

写了一个用于实时反汇编指令的 module, 详情见 [DASM](https://github.com/roife/dasm).

## 自动化测试

嫖了讨论区 auto-test 的 py 文件, 但是还没写 mips code generator.
