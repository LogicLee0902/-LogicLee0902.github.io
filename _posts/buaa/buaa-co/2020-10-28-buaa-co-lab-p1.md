---
layout: "post"
title: "「BUAA-CO Lab」 P1 Verilog 模块及状态机 (unfinished)"
subtitle: "Verilog 模块设计"
author: "roife"
date: 2020-10-25

tags: ["C「BUAA - Computer Organization」", "B「Digital Design and Computer Architecture」", "L「Verilog-HDL」", "BUAA", "计算机组成",]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
mathjax: true
---

# 笔记

## 位扩展

`$signed()` 用法太玄妙了, 推荐全部用位扩展替代.

<!-- {%raw%} -->
```verilog
{{16{imm[15]}}, imm} // 将 [15:0] 的 imm 扩展到 32 位
```
<!-- {%endraw%} -->

## testbench 的优雅写法

```verilog
reg [0:7] in;
reg [0:1023] str = "test data";

initial begin
    // Initialize inputs
    // ...
    #5;
    reset = 0;
    while (!str[0:7]) str = str << 8;
    #5;
    while(str[0:7]) begin
        $display("%c", str[0:7]);
        in = str[0:7];
        str = str << 8;
        #5;
    end
end
```
