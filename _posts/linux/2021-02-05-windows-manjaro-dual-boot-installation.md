---
layout: "post"
title: "Win10/Manjaro 双系统安装"
subtitle: "在拯救者 y7000p "
author: "roife"
date: 2021-02-05

tags: ["Linux", "Manjaro"]
language: zh-CN
catalog: true
header-image: ""
header-style: text
---

# 前言

# 准备

# 安装系统

# 换源

# 安装软件及配置

## 开发工具链

### C/C++

```shell
$ yay -S base-devel # 包含了基础开发套件，如 gcc 等
$
```

### Coq

### Haskell

### Node.JS

###

## 编辑器

```shell
$ yay -S vim

$ yay -S visual-studio-code

$ yay -S emacs
# 使用 Spacemacs 作为基础配置
$ git clone -b develop https://github.com/syl20bnr/spacemacs ~/.emacs.d --depth=1
```

我日常以 Emacs 和 VSCode 为主力编辑器，因此会为二者装上大量的插件。而 Vim 作为轻量级编辑器使用，只会配置一个比较小的 `.vimrc` 文件。关于具体的插件和配置，等我下一篇写完了再附上来。

## IDE

### JetBrains

首先当然是要安装 JetBrains 全家桶。

```shell
$ yay
```

## 访问外网 `clash`

```shell
$ yay -S clash
```