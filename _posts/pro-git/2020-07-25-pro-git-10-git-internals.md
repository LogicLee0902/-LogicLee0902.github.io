---
layout: "post"
title: "「Pro Git」 10 Git Internals (unfinished)"
subtitle: "Git 内部原理"
author: "roife"
date: 2020-07-25

tags: ["Pro Git@B", "Git@D", "未完成@D"]
language: zh-CN
catalog: true
header-image: ""
header-style: text
---

# Git Internals

## 底层命令与上层命令

底层命令指和底层工作有关的命令.

一个典型的 `.git` 目录:

``` shell
$ ls -F1
config # 项目特有配置
description # web 程序使用
HEAD # HEAD 指针
index # 暂存区信息
hooks/ # 钩子
info/ # global exclude 文件
objects/ # 存储数据
refs/ # 存储指针
```

## git 对象

### blob 对象

- `git hash-object`
  : 将数据保存在 `.git/objects` 返回 SHA-1

  - `-w`
    : 写入数据库
  - `--stdin`
    : 从标准输入读取, 不加就要加上路径

<!-- end list -->

``` shell
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
# 此时 objects 目录下就有一个同名文件

$ echo 'version 1' > test.txt
$ git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30
```

此时数据用 blob 对象存储

- `git cat-file <SHA-1>`
  : 用 SHA-1 取回数据

  - `-p`
    : 自动识别类型
  - `-t`
    : 输出类型

<!-- end list -->

``` shell
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content

$ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
$ cat test.txt
version 1

$ git cat-file -t 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
blob
```

### 树对象
