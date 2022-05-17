---
layout:     post
title:      "「The Missing Semester of Your CS Education」02 shell的细致讲解"
subtitle:   OS pre 03
date:       2022-03-04
author:     Leo

header-img: ""
catalog: true
tags: ["shell@Tags@Tags", "操作系统@Courses@Series", "MIT@Schools@Series"]
lang: zh
header-style: text
katex: true
---

> 这是对01的补充，更为详细和系统

本系列的资料[计算机教育中缺失的一课 · the missing semester of your cs education (missing-semester-cn.github.io)](https://missing-semester-cn.github.io/)

# Shell脚本

大多数shell都有自己的一套脚本语言，包括变量、控制流和自己的语法。shell脚本与其他脚本语言不同之处在于，shell脚本针对shell所从事的相关工作进行来优化。因此，创建命令流程（pipelines）、将结果保存到文件、从标准输入中读取输入，这些都是shell脚本中的原生操作，这让它比通用的脚本语言更易用。

再bash中为变量赋值的语法是`foo=bar`，访问变量中存储的数值用`$foo`

**注意**：`foo = bar`不能正确工作的，因为这种有空格，解释器会调用`foo`并将`=`和`bar`作为参数。（空格造成了分割参数的效果）

Bash中的字符串通过`'` 和 `"`分隔符来定义，但是它们的含义并不相同。以`'`定义的字符串为原义字符串，其中的变量不会被转义，而 `"`定义的字符串会将变量值进行替换。

```bash
foo=bar
echo "$foo"
# 打印 bar
echo '$foo'
# 打印 $foo
```

bash也支持`if`, `else`等条件语句，`while`,`for`等循环语句，并可以调用函数，并基于函数的参数(如果有的话)进行操作。

所以对于向`sed "s/\$1/\$2", 可以得到第一个参数和第二个参数，如果用' '，就是文本意义上的\$1,\$2

例子如下，会创建一个文件夹并使用cd进入该文件夹

```shell
mcd () {
    mkdir -p "$1"
    cd "$1"
}
# 运行时调用命令 mcd Name 就会创建叫name的文件夹并cd到那里面
```

`$1`是脚本的第一个参数，bash有很多特殊变量来参数参数，错误代码和相关变量（有点像mips的规定俗称的通用寄存器用法），常见的一些如下

- `$0` - 脚本名
- `$1` 到 `$9` - 脚本的参数。 `$1` 是第一个参数，依此类推。
- `$@` - 所有参数
- `$#` - 参数个数
- `$?` - 前一个命令的返回值
- `$$` - 当前脚本的进程识别码
- `!!` - 完整的上一条命令，包括参数。常见应用：当你因为权限不足执行命令失败时，可以使用 `sudo !!`再尝试一次。
- `$_` - 上一条命令的最后一个参数。如果正在使用的是交互式shell，你可以通过按下 `Esc` 之后键入` .` 来获取这个值

命令通常使用 `STDOUT`来返回输出值，使用`STDERR` 来返回错误及错误码，便于脚本以更加友好的方式报告错误。 返回码或退出状态是脚本/命令之间交流执行状态的方式。返回值0表示正常执行，其他所有非0的返回值都表示有错误发生。

退出码可以搭配`&&` (与操作符) 和 `||` (或操作符)使用，用来进行条件判断，决定是否执行其他程序。 `A && B`中意思是A为真（返回0），才会执行B，而`A || B`，当`A`为真时，不执行B，只有`A`为假时，才会执行B 

它们都属于**短路运算符**(short-circuiting)，同一行的多个命令可以用`;`分隔，程序 `true` 的返回码永远是`0`，`false` 的返回码永远是`1`。

看几个例子

```shell
#:means output
false || echo "Oops, fail" 
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed" 
#

false ; echo "This will always run"
# This will always run
```

还有一种模式是以变量的形式获取一个命令的输出，可以通过**变量替换**（command substitution)

标准形式为`$(CMD)`，这时候会将`CMD`的输出结果替换掉`$(CMD)`,例如，如果执行 `for file in $(ls)` ，shell首先将调用`ls` ，然后遍历得到的这些返回值。

此外，还有一个特性为**进程替换**（process substitution), `<(CMD)`会执行`CMD`并将结果输出到一个临时的文件中，并将`<(CMD)`替换成临时文件名。这在希望返回值通过文件而不是STDIN传递时很有用。例如， `diff <(ls foo) <(ls bar)` 会显示文件夹 `foo` 和 `bar` 中文件的区别。

(用一个临时文件存储中间信息，方便后续操作)

再来看一个例子，这段脚本会遍历提供的参数，用`grep`搜索字符串`foobar`,如果没有找到，则将其作为注释追加到文件中

```shell
#!/bin/bash

echo "Starting program at $(date)" # date会被替换成日期和时间

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # 如果模式没有找到，则grep退出状态为 1
    # 我们将标准输出流和标准错误流重定向到Null，因为我们并不关心这些信息
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

条件语句的意思是比较`$?`是否等于0，bash有许多类似的比较操作，称作Test擦做，一般建议使用双括号`[[]]`

当执行脚本时，需要提供形式类似的参数，bash中有**通配**技术（globbing），有点像正则表达式，可以用通配符、花括号等

* 通配符`-`，当想要利用通配符进行匹配时，可以使用`?`和`*`来匹配一个或任意字符（可以为0个），对于文件`foo`, `foo1`, `foo2`, `foo10` 和 `bar`, `rm foo?`这条命令会删除`foo1` 和 `foo2` ，而`rm foo*` 则会删除除了`bar`之外的所有文件。(`rm`(remove)，删除命令)
* `{}`,有一系列存在公共子串的指令时，`{}`可以自动展开这些

```shell
convert image.{png,jpg}
# 会展开为
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# 会展开为
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# 也可以结合通配使用
mv *{.py,.sh} folder
# 会移动所有 *.py 和 *.sh 文件

mkdir foo bar

# 下面命令会创建foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h这些文件
touch {foo,bar}/{a..h}
touch foo/x bar/y
# 比较文件夹 foo 和 bar 中包含文件的不同
diff <(ls foo) <(ls bar)
# 输出
# < x
# ---
# > y
```

**注意，脚本并不一定只有用bash写才能在终端里调用**。比如说，这是一段Python脚本，作用是将输入的参数倒序输出：

```python
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

内核知道去用python解释器而不是shell命令来运行这段脚本，是因为脚本的开头第一行的**shebang。**

在 `shebang` 行中使用 `env`命令会利用环境变量中的程序来解析该脚本，这样就提高来脚本的可移植性。`env` 会利用PATH环境变量来进行定位。 例如，使用了`env`的shebang看上去时这样的`#!/usr/bin/env python`。

shell函数和脚本有如下一些不同点：

- 函数只能用与shell使用相同的语言，脚本可以使用任意语言。因此在脚本中包含 `shebang` 是很重要的。
- 函数仅在定义时被加载，脚本会在每次被执行时加载。这让函数的加载比脚本略快一些，但每次修改函数定义，都要重新加载一次。
- 函数会在当前的shell环境中执行，脚本会在单独的进程中执行。因此，**函数可以对环境变量进行更改**，比如改变当前工作目录，脚本则不行。脚本需要使用 [`export`将环境变量导出，并将值传递给环境变量。
- 与其他程序语言一样，函数可以提高代码模块性、代码复用性并创建清晰性的结构。shell脚本中往往也会包含它们自己的函数定义。

# shell工具

## 查看命令如何使用

---

如何为特定的命令找到适合的标记是需要考虑一个问题，例如`ls -l`, `mv -i` 和 `mkdir -p`,更普遍的来说，如何在命令行中后去相关信息的方法。

其实之前也提到了最常用的方法是`-h`和`man`,其中`man`显示的更为纤细， 事实上，目前给出的所有命令的说明链接，都是网页版的Linux命令手册。即使是安装的第三方命令，前提是开发者编写了手册并将其包含在了安装包中。在交互式的、基于字符处理的终端窗口中，一般也可以通过 `:help` 命令或键入 `?`来获取帮助。

## 查找文件

---

最常见的重复任务就是查找文件或目录。所有的类UNIX系统都包含一个名为 `find`]的工具，它是shell上用于查找文件的绝佳工具。`find`命令会递归地搜索符合条件的文件，例如：

```shell
# 查找所有名称为src的文件夹
find . -name src -type d
# 查找所有文件夹路径中包含test的python文件
find . -path '*/test/*.py' -type f
# 查找前一天修改的所有文件
find . -mtime -1
# 查找所有大小在500k至10M的tar.gz文件
find . -size +500k -size -10M -name '*.tar.gz'
```

除了列出所寻找的文件之外，find还能对所有查找到的文件进行操作。这能极大地简化一些单调的任务。

```shell
# 删除全部扩展名为.tmp 的文件
find . -name '*.tmp' -exec rm {} \;
# 查找全部的 PNG 文件并将其转换为 JPG
find . -name '*.png' -exec convert {} {}.jpg \;
```

不过`find`尽管功能强大，其弊端在于语法难记，比如`-name`, `-path`, `-size`, `-exec`等等等

>  记住，shell最好的特性就是您只是在调用程序，因此您只要找到合适的替代程序即可（甚至自己编写）。

因此，为了解决这个问题，拥有一个替代的`fd`,它有很多不错的默认设置，例如输出着色、默认支持正则匹配、支持unicode并且我它的语法更符合直觉。以模式`PATTERN` 搜索的语法是 `fd PATTERN`。

此外，还可以通过[`locate`](https://man7.org/linux/man-pages/man1/locate.1.html)通过建立数据库的方法加快搜索，但数据库无法做到实时更新

这便需要我们在速度和时效性之间作出权衡。而且，`find` 和类似的工具可以通过别的属性比如文件大小、修改时间或是权限来查找文件，`locate`则只能通过文件名。 [here](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other)有一个更详细的对比。

## 查找文件内容

---

很多类UNIX的系统都提供了[`grep`](https://man7.org/linux/man-pages/man1/grep.1.html)命令，它是用于对输入文本进行匹配的通用工具。在上一篇中也有过介绍

`grep` 有很多选项，这也使它成为一个非常全能的工具。例如 `-C` ：获取查找结果的上下文（Context）；`-v` 将对结果进行反选（Invert），也就是输出不匹配的结果。举例来说， `grep -C 5` 会输出匹配结果前后五行。当需要搜索大量文件的时候，使用 `-R` 会递归地进入子目录并搜索所有的文本文件。

但是，仍热有很多办法可以对 `grep -R` 进行改进，例如使其忽略`.git` 文件夹，使用多CPU等等。

因此也出现了很多它的替代品，包括 [ack](https://beyondgrep.com/), [ag](https://github.com/ggreer/the_silver_searcher) 和 [rg](https://github.com/BurntSushi/ripgrep)。都很好用，其中 ripgrep (`rg`) 它速度快，而且用法非常符合直觉。例子如下：

```shell
# 查找所有使用了 requests 库的文件
rg -t py 'import requests' # -t means total
# 查找所有没有写 shebang 的文件（包含隐藏文件）
rg -u --files-without-match "^#!"
# 查找所有的foo字符串，并打印其之后的5行
rg foo -A 5
# 打印匹配的统计信息（匹配的行和文件的数量）
rg --stats PATTERN
```

与 `find`/`fd` 一样，重要的是你要知道有些问题使用合适的工具就会迎刃而解，而具体选择哪个工具则不是那么重要。

## 查找shell命令

---

首先，按向上的方向键会显示你使用过的上一条命令，继续按上键则会遍历整个历史记录。

`history` 命令允许以程序员的方式来访问shell中输入的历史命令。这个命令会在标准输出中打印shell中的里面命令。如果我们要搜索历史记录，则可以利用管道将输出结果传递给 `grep` 进行模式搜索。 `history | grep find` 会打印包含find子串的命令。

对于大多数的shell来说，可以使用 `Ctrl+R` 对命令历史记录进行回溯搜索。敲 `Ctrl+R` 后可以输入子串来进行匹配，查找历史命令行。

`Ctrl+R` 可以配合 [fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r) 使用。`fzf` 是一个通用对模糊查找工具，它可以和很多命令一起使用。这里可以对历史命令进行模糊查找并将结果以赏心悦目的格式输出。

另外一个和历史命令相关的技巧被称之为**基于历史的自动补全**。 这一特性最初是由 [fish](https://fishshell.com/) shell 创建的，它可以根据最近使用过的开头相同的命令，动态地对当前对shell命令进行补全。

以上的操作在 [zsh](https://github.com/zsh-users/zsh-history-substring-search)也同样适用（另一种shell）

还可以修改 shell history 的行为，例如，如果在命令的开头加上一个空格，它就不会被加进shell记录中。当输入包含密码或是其他敏感信息的命令时会用到这一特性。 为此需要在`.bashrc`中添加`HISTCONTROL=ignorespace`或者向`.zshrc` 添加 `setopt HIST_IGNORE_SPACE`。 如果不小心忘了在前面加空格，可以通过编辑。`bash_history`或 `.zhistory` 来手动地从历史记录中移除那一项。

## 文件夹导航

---

之前对所有操作都默认一个前提，即已经位于想要执行命令的目录下，但是在后面应该要达到高效地在目录 间随意切换。有几种方法，比如设置alias，使用 [ln -s](https://man7.org/linux/man-pages/man1/ln.1.html)创建符号连接等。而开发者们已经想到了很多更为精妙的解决方案。

**可以使用[`fasd`](https://github.com/clvv/fasd)和[autojump](https://github.com/wting/autojump)这两个工具来查找最常用或最近使用的文件和目录。**

Fasd 基于 [*frecency*](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm)对文件和文件排序，也就是说它会同时针对频率（*frequency* ）和时效（ *recency*）进行排序。默认情况下，`fasd`使用命令 `z` 帮助我们快速切换到最常访问的目录。例如， 如果您经常访问`/home/user/files/cool_project` 目录，那么可以直接使用 `z cool` 跳转到该目录。对于 autojump，则使用`j cool`代替即可。

还有一些更复杂的工具可以用来概览目录结构，例如 [`tree`](https://linux.die.net/man/1/tree), [`broot`](https://github.com/Canop/broot) 或更加完整的文件管理器，例如 [`nnn`](https://github.com/jarun/nnn) 或 [`ranger`](https://github.com/ranger/ranger)。

>**alias**:
>
>​	用于设置指令的别名。
>
>用户可利用alias，自定指令的别名。若仅输入alias，则可列出目前所有的别名设置。alias的效力仅及于该次登入的操作。若要每次登入是即自动设好别名，可在.profile或.cshrc中设定指令的别名
>
>### 语法
>
>```
>alias[别名]=[指令名称]
>```
>
>**参数说明**：若不加任何参数，则列出目前所有的别名设置。
>
>### 实例
>
>给命令设置别名
>
>[![b64wdJ.png](https://s1.ax1x.com/2022/03/07/b64wdJ.png)](https://imgtu.com/i/b64wdJ)
>
>除了以上用法之外，可以用unalias删除别名

