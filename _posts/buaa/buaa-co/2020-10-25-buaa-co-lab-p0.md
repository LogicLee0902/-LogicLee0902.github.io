---
layout: "post"
title: "「BUAA-CO Lab」 P0 Logisim 模块及状态机"
subtitle: "Logisim 电路模块设计"
author: "roife"
date: 2020-10-25

tags: ["C「BUAA - Computer Organization」", "BUAA", "计算机组成", "数字电路"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
mathjax: true
---

# 课上总结

第一题是建一个 Mealy 状态机, 第二题是一个简单组合电路, 第三题是 (稍微) 复杂一些的组合电路 (计算 md5).

第一次的题目应该不算难, 但是不知是什么原因没做出来第一题, 可能我平时对状态机也不是很对付. 总之还是勉强过了 P0, 但是丢了优秀 (而且前一天还用了 2 小时专门复习, 结果还是这样, 导致心情极差).

第三题需要找一个 32 位二进制数中第一次出现 `10101` 的位置, 一开始以为是类似于 GRF 一样的题目, 建了一半发现可以用 bit finder 做 (但是好像也有爷爷用倍增找, 太牛了).

多谢助教出了两个组合电路救我狗命, 希望下周 Verilog 不要再翻车了, 回去一定好好看计组 (悲).

总结:
1. 考试快结束了不要慌, 特别是应用题, 多猜出题人的想法
2. 课下多熟悉 Logisim 左边冷门组件的用法, 关键时刻说不定可以救命

# 课下总结

## 状态机类型

### Mealy

![Mealy](/img/in-post/post-buaa-co/p0-lab-mealy.png "p0-lab-mealy"){:height="400px" width="400px"}

### Moore

![Moore](/img/in-post/post-buaa-co/p0-lab-moore.png "p0-lab-moore"){:height="400px" width="400px"}

## reset 信号

- 异步清 0: reset 信号接寄存器和 counter, 输出信号接 MUX 后面
- 同步清0: reset 信号接 counter, 输出信号接 MUX 前面

## 易错点

- splitter 高低位分清
- 状态机使用独热码
