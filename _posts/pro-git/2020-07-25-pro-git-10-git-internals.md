---
layout: "post"
title: "「Pro Git」 10 Git Internals"
subtitle: "Git 内部原理"
author: "roife"
date: 2020-07-25

tags: ["Pro Git@B", "Git@D"]
lang: zh
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

blob 对象只能存储文件内容, 树对象能够存储一些元数据信息.

```shell
$ git cat-file -p master^{tree} # master^{tree} 表示当前 master 分支上最新 commit 指向的树对象
100644 blob a906cb2a4a904a152e80877d4088654daad0c859      README
100644 blob 8f94139338f9404f26296befa88755fc2598c289      Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0      lib # lib 是目录, 所以它是一个树对象指针, 指向另一个树对象

$ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b      simplegit.rb
```

![data-model-1](/img/in-post/post-pro-git/data-model-1.png){:height="450px" width="450px"}

