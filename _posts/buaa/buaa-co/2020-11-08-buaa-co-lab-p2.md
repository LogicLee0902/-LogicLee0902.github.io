---
layout: "post"
title: "「BUAA-CO Lab」 P2 MIPS 汇编程序设计"
subtitle: "Verilog 模块设计"
author: "roife"
date: 2020-10-31

tags: ["C「BUAA - Computer Organization」", L「MIPS Assembly」", "BUAA", "计算机组成"]
status: Completed

language: zh-CN
catalog: true
header-image: ""
header-style: text
mathjax: true
---

# 课下总结

## 常用宏命令

### 二维数组取地址

```asm
.macro INDEX(%ans, %i, %j, %rank)
    multu %i, %rank
    mflo %ans
    add %ans, %j
    sll %ans, %ans, 2
.end_macro
```

### 读入和输出整数

```asm
.macro RI(%n)
    li $v0, 5
    syscall
    move %n, $v0
.end_macro

.macro PI(%n)
    li $v0, 1
    move $a0, %n
    syscall
.end_macro
```

### 保护和读取寄存器

```asm
.macro LOAD_LOCAL(%var)
	addi $sp, $sp, 4
	lw %var 0($sp)
.end_macro

.macro SAVE_LOCAL(%var)
	sw %var 0($sp)
	subi $sp, $sp, 4
.end_macro
```

### 输出空白和换行

```asm
.macro PSPACE
	la $a0, str_space
	li $v0, 4
	syscall
.end_macro

.macro PENTER
	la $a0, str_enter
	li $v0, 4
	syscall
.end_macro
```

## 常见错误

- 数据类型的大小: `int: 1word, 4byte`, 用 `lw/sw`, 记得 `sll $t0, $t0, 2`; `char: 1byte`, 用 `lb/sb`.
- 递归: 存储数据和恢复数据的顺序恰好相反
