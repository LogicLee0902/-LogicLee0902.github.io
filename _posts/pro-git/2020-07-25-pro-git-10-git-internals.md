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

# 底层命令与上层命令

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

# git 对象

Git 本质上是一个 key-value 的数据库系统, 可以在其中插入任何内容, 并且返回一个对应的 key, 然后根据 key 取出对应的内容.

## blob 对象

- `git hash-object`
  : 将数据保存在 `.git/objects` 返回 SHA-1

  - `-w`
    : 写入数据库
  - `--stdin`
    : 从标准输入读取, 不加就要加上路径

``` shell
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
# 此时 objects 目录下就有一个同名文件, 记录了这个内容

$ echo 'version 1' > test.txt
$ git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30
```

此时数据用 blob 对象存储.

- `git cat-file <SHA-1>`
  : 用 SHA-1 取回数据

  - `-p`
    : 自动识别类型
  - `-t`
    : 输出类型

``` shell
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4 # 取出第一个版本
test content

$ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
$ cat test.txt
version 1

$ git cat-file -t 83baae61804e65cc73a7201a7252750c76066a30
blob
```

## 树对象

blob 对象只能存储文件内容, 树对象能够存储一些元数据信息. 一个树对象包含了一条或多条 tree entry, 每个 entry 包含一个指向 blob 或者子树对象的指针, 以及相应的元信息.

```shell
$ git cat-file -p master^{tree} # master^{tree} 表示当前 master 分支上最新 commit 指向的树对象
100644 blob a906cb2a4a904a152e80877d4088654daad0c859      README
100644 blob 8f94139338f9404f26296befa88755fc2598c289      Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0      lib # lib 是目录, 所以它是一个树对象指针, 指向另一个树对象

$ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b      simplegit.rb
```

![data-model-1](/img/in-post/post-pro-git/data-model-1.png){:height="450px" width="450px"}

Git 会从暂存区中提取内容创建树对象, 所以我们要先将内容写入暂存区 (这里使用底层命令而不用 `git add`).

- `git update-index`
  : 更新暂存区区域
  + `--add`
    : 将文件添加到暂存区
  + `--cacheinfo <mode> <object> <path>`
    : 文件第一次存入时使用, 指定文件模式/SHA-1/文件名 (`100644` 表示普通文件, `100755` 表示可执行文件, `120000` 表示符号链接等)
- `git write-tree`
  : 将暂存区内容写入一个树对象

```shell
$ git update-index --add --cacheinfo 100644 83baae61804e65cc73a7201a7252750c76066a30 test.txt

$ git write-tree
d8329fc1cc938780ffdd9f94e0d364e0ea74f579

$ git cat-file -p d8329fc1cc938780ffdd9f94e0d364e0ea74f579 # 查看树对象
100644 blob 83baae61804e65cc73a7201a7252750c76066a30 test.txt

$ git cat-file -t d8329fc1cc938780ffdd9f94e0d364e0ea74f579 # 查看 SHA-1 对应的对象类型
tree
```

- `git read-tree`
  : 将树对象读入暂存区
  + `--prefix=<name>`
    : 将读出的树对象放到 `<name>` 目录下

```shell
$ git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579

$ git write-tree
3c4e9cd789d88d8d89c1073707c3585e41b0e614

$ git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614
040000 tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579     bak
100644 blob 83baae61804e65cc73a7201a7252750c76066a30     test.txt
bak new.txt test.txt
```

## commit 对象

commit 对象可以保存每次 commit 的树对象 SHA-1 值, commit 时间等信息.

- `git commit-tree <SHA-1>`
  : 创建一个 commit 对象, SHA-1 为对应树对象的 SHA-1 值
  + `-p <SHA-1>`
    : 指定对应父 commit

```shell
$ echo 'first commit' | git commit-tree d8329f fdf4fc3344e67ab068f836878b6c4951e3b15f3d # 提供 commit message 与树对象的 SHA-1 值
fdf4fc3344e67ab068f836878b6c4951e3b15f3d

$ git cat-file -p fdf4fc3
tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
author Scott Chacon <schacon@gmail.com> 1243040974 -0700
committer Scott Chacon <schacon@gmail.com> 1243040974 -0700

first commit

$ echo 'second commit' | git commit-tree 0155eb -p fdf4fc3 cac0cab538b970a37ea1e769cbbde608743bc96d # 在上一个 commit 下创建一个新的 commit
cac0cab538b970a37ea1e769cbbde608743bc96d
```

此时使用 `git log` 就会发现我们已经用底层命令实现了上层命令 `git add/git commit` 的效果. 同时在 `.git/objects` 中可以找到各个文件版本对应的内容 (文件名为 SHA-1).

```shell
$ find .git/objects -type f
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30 # test.txt v1
.git/objects/ca/c0cab538b970a37ea1e769cbbde608743bc96d # commit 2
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4 # 'test content'
.git/objects/d8/329fc1cc938780ffdd9f94e0d364e0ea74f579 # tree 1
.git/objects/fd/f4fc3344e67ab068f836878b6c4951e3b15f3d # commit 1
```

可以发现文件夹名为 SHA-1 值的前两位.

## 对象 hash

### blob 对象

blob 对象的格式为 `blob <content length><NUL><content>`.

首先计算头信息. 假设文件内容为 `content`, 则 `header = "blob \(content.length)\0"`, `length` 为**字节长度**.

然后将二者而二进制拼接起来并计算 SHA-1: `store = header + store`, `hash = sha1(store)`.

接着用 zlib 压缩信息: `zlib_content = zlib(store)`. 并将其写入磁盘. 其中 hash 的前两位作为目录名, 后面部分作为文件名.

### 树对象

树对象的格式为 `tree <content length><NUL>[<file mode> <filename><NUL><item sha>]...<NUL>`, 其中 `<item sha>` 为二进制形式的 SHA-1.

首先计算树对象内容: `content = file_mode + filename + "\0" + bin(item_sha)` (如 `"100644 test.txt\0" + bin(sha)`).

然后拼接得到头信息: `header = header = "tree \(content.length)\0"` 其余步骤和 blob 对象的 hash 一样

### commit 对象

commit 对象的格式 (`content`) 如下:

```shell
commit <content length><NUL>tree <tree sha>
parent <parent sha>
[parent <parent sha> if several parents from merges]
author <author name> <author e-mail> <timestamp> <timezone>
committer <author name> <author e-mail> <timestamp> <timezone>

<commit message>
```

计算方法和上述相同.

# Git 引用

## 引用简介

原始的 SHA-1 值很难记忆, 所以可以将其存在文件中, 并给文件取一个简单的名字作为它的引用. 这些引用存在 `.git/refs` 下, 共有 `heads`, `tags` 和 `remotes` 三种, 分别代表 `HEAD` 引用、标签引用和远程引用.

可以直接将 SHA-1 值写入文件:

```shell
$ echo 1a410efbd13591db07496601ebc7a059dd55cfe9 > .git/refs/heads/master
```

然后就可以用 `master` 代替这个 SHA-1 值. 如 `git log master` 等价于 `git log 1a410e`. 当然, 这是不安全的, 最好用底层命令.

- `git update-ref <ref-path> <SHA-1>`
  : 更新 `ref`

```shell
$ git update-ref refs/heads/test cac0ca # 将 test 引用到 cac0ca
```

执行 `git branch <branch>` 时实际上执行了 `git update-ref`.

## HEAD 引用

HEAD 是一个**符号引用**, 它不指向某个具体的 commit, 而是一个指向另一个引用的指针, 通常 HEAD 指向当前所在的分支. 如果 HEAD 中是一个 SHA-1 值而不是另一个引用, 则此时处于分离 HEAD 状态.

```shell
$ cat .git/HEAD
ref: refs/heads/master
```

使用 `git checkout` 或 `git commit` 时会更改 `HEAD` 文件的内容.

- `git symbolic-ref <ref>`
  : 查看 ref 文件的内容
- `git symbolic-ref <sym-ref> <ref-path>`
  : 将 sym-ref 的指向 ref

```shell
$ git symbolic-ref HEAD refs/heads/test

$ cat .git/HEAD
ref: refs/heads/test
```

## 标签引用

标签对象也是一种 git 对象, 类似于 commit 对象包含日期注释等信息. 区别在于标签对象指向一个 commit 对象, 而不是树对象. 标签对象像一个不移动的分支, 永远指向同一个 commit 对象.

