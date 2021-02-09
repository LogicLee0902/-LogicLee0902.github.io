---
layout: "post"
title: "Win 10 & Manjaro 双系统配置"
subtitle: "系统安装，软件配置及介绍（以 Y7000p 为例）"
author: "roife"
date: 2021-02-05

tags: ["Linux@T", "Manjaro@T"]
language: zh-CN
catalog: true
header-image: ""
header-style: text
---

# 前言

虽然是第二次使用 manjaro 了，但是中途还是遇到不小的麻烦，导致反反复复实验，终于总结出了一套不会出错的流程，记录下来备用。

# 准备

在 [Manjaro](https://manjaro.org/download) 下载 ISO 镜像，我这里用的是 KDE Plasma。

然后下载 [rufus](https://rufus.ie/en_IE.html) 用于写入镜像，下载完成后用 rufus 向 u 盘写入镜像即可。

接下来对磁盘进行分区，右键“此电脑” / 管理 / 磁盘管理，打开后在 C 盘进行“压缩卷”，我大概分配了 400GB 给 manjaro。

# 安装系统

插入 u 盘，重启电脑，选择 U 盘启动（拯救者连按 F12 进入启动选择界面），注意在选择显卡驱动的时候使用非开源镜像，点击 `Launch Installer` 就可以进入安装了。

安装过程没啥好说的。个人的话习惯将语言设置 American English，使用 Asia-Shanghai 时区。分区直接使用覆盖分区，然后选择刚刚的分区即可。

Office 这里建议使用 WPS Office，在安装完系统后自行安装。

# 换源

启动后第一件事是对 `pacman` 换源，启动终端输入：

```shell
$ sudo pacman-mirrors -i -c China -m rank
```

在跳出来的界面选一个源确定即可，然后更新系统。

```shell
$ sudo pacman -Syyu
```

# 双显卡

由于我是 Intel+NVIDIA，Manjaro 的驱动自动解决了双显卡的问题（真是方便啊）。

需要用 NVIDIA 启动的软件只需要在驱动前使用 `prime-run` 即可。

```shell
$ prime-run python
```

# 安装软件及配置

```shell
$ sudo pacman -S yay # 安装 yay，yay 对比 pacman 可以自动寻找可用的包
$ yay -S base-devel # 包含了基础开发套件，如 sudo，grep，gcc 等
```

## 输入法 `fcitx5`

使用 `fcitx5` 作为中文输入法。

```shell
$ yay -S fcitx5 fcitx5-configtool fcitx5-chinese-addons fcitx5-material-color
```

然后在 `~/.pam_environment` 中写入：

```shell
TK_IM_MODULE DEFAULT=fcitx
QT_IM_MODULE  DEFAULT=fcitx
XMODIFIERS    DEFAULT=\@im=fcitx
SDL_IM_MODULE DEFAULT=fcitx
```

## 访问外网（`clash`）

```shell
$ yay -S clash
```

装完后将 `Country.mmdb` 和 `config.yaml` 从另外一台电脑上复制过来，前者是一个数据库，后者是代理配置方案。
打开后可以在 [](clash.razord.top) 中用 GUI 的方式配置。

如果要设置为开机自启动，可以利用 `systemd` 设置：将以下内容保存在 `/etc/systemd/system/clash.service`。

```conf
[Unit]
Description=Clash service
After=network.target

[Service]
Type=simple
User=roife # 这里写上用户名
ExecStart=/usr/bin/clash
Restart=on-failure
RestartPreventExitStatus=23

[Install]
WantedBy=multi-user.target
```

然后设置 `clash` 为开机启动：

```shell
sudo systemctl daemon-reload
sudo systemctl enable clash
sudo systemctl start clash
sudo systemctl status clash
```

## 使用 ZSH

manjaro 的 zsh 自动集成了一系列插件，包括 `zsh-autosuggestions` 和 `zsh-syntax-highlighting`，包括了一套美观的主题，所以不需要再使用 `oh-my-zsh` 了，除非你还要安装额外的插件。

下面贴一份我常用的 alias，非常好用：

```shell
# alias
alias l='ls -laG'
alias grep='grep --color'
alias ec='emacsclient -n'

#alias-git
alias g='git'
alias ga='git add'
alias gc='git commit'
alias gp='git push'
alias gst='git status'
alias gr='git reset --HARD'
alias gd='git diff'
```

## 开发工具链

### C/C++

```shell
$ yay -S gdb
$ yay -S cmake
$ yay -S clang llvm lldb
```

### NodeJS

首先安装 `nvm` 和 `NodeJS`。

```shell
$ yay -S nvm
$ echo 'source /usr/share/nvm/init-nvm.sh' >> ~/.zshrc
$ nvm install stable && nvm use stable
```

然后安装 `npm`，并给 `npm` 换源。

```shell
$ yay -S npm

$ npm config set registry https://registry.npm.taobao.org
$ npm config get registry # 检测是否成功
```

### Ruby

主要是写博客用到了 Jekyll，需要用 Ruby 进行编译。这里安装 `rvm`，并用 `rvm` 安装 `ruby`。

```shell
# Install rvm
$ curl -L get.rvm.io > rvm-install
$ zsh < ./rvm-install
$ source ~/.zsh_profile

# Install ruby
$ rvm list known
$ rvm use 2.7

# Install bundler

$ gem install bundler
$ bundle install
$ bundle exec jekyll serve
```

### Python（Anaconda）

平常除了把 Python 当作脚本语言用以外，用得最多的还是 Anaconda。

```shell
$ yay -S anaconda
$ echo "export PATH=/opt/anaconda/bin:$PATH" # 添加环境变量
```

然后就可以用 `conda activate` 进入环境，退出用 `conda deactivate`。

#### Pytorch

如果要用到 Pytorch，注意检查一下自己前面的双显卡配置。

```shell
$ yay -S python-pytorch-cuda # 这句话会安装 CUDA
$ conda install pytorch torchvision torchaudio cudatoolkit=11.0 -c pytorch # 这句话要根据你的 CUDA 版本决定
```

第二句可以在 `https://pytorch.org/get-started/locally/` 中，选择自己的系统/CUDA 版本后找到适合自己电脑的命令。

其中 CUDA 的版本可以用 `$ nvidia-smi` 查看。

装完后进入 conda 环境测试一下。

```python
>>> import torch;
>>> torch.cuda.is_available() # 输出 true 则成功
```

### Java

`pacman` 和 `yarn` 只能下载 `open-jdk` 如果要 Oracle JDK，只能手动下载。

以 JDK8 为例，下载完 `pkg` 包放在 Download 目录里，然后运行命令就可以自动安装。

```shell
$ yay -S jdk8
```

当然，安装 open-jdk 就简单多了。

```shell
$ yay -S jdk11-openjdk jdk-openjdk
```

安装完可以用 `archlinux-java` 命令查看版本并切换。

### 其他语言

```shell
$ yay -S iverilog gtkwave # verilog
$ yay -S coq # coq
$ yay -S stack # haskell
$ yay -S racket # racket
```

## 编辑器

```shell
$ yay -S vim

$ yay -S visual-studio-code-bin

$ yay -S emacs

# 使用 Spacemacs 作为基础配置
$ git clone -b develop https://github.com/syl20bnr/spacemacs ~/.emacs.d --depth=1
```

我日常以 Emacs 和 VSCode 为主力编辑器，因此会为二者装上大量的插件。而 Vim 作为轻量级编辑器使用，只会配置一个比较小的 `.vimrc` 文件。关于具体的插件和配置，等我下一篇写完了再附上来。

### Emacs 与 fcitx5

首先在系统设置里面启用 `zh_CN.UTF-8`。

在 Emacs 中使用 fcitx5 需要配置 `locale`，即 `LC_CTYPE=zh_CN.UTF-8`。编辑 `/etc/locale.conf`

```shell
LANG=en_US.UTF-8
LC_CTYPE=zh_CN.UTF-8
```

然后 `sudo locale-gen`。

## IDE

### JetBrains

首先当然是要安装 JetBrains 全家桶，白嫖 Pro 版美滋滋。注意使用前必须要安装 Java 环境，且至少为 Java 11。

```shell
$ yay -S pycharm-professional webstorm intellij-idea-ultimate-edition clion
```

注意刚装上去的时候字体非常小，需要在菜单里配置 `custom font`。

### QT Creator

```shell
$ yay -S qtcreator
```

## 字体

```shell
$ sudo pacman -S ttf-roboto noto-fonts ttf-dejavu
# 文泉驿
$ sudo pacman -S wqy-bitmapfont wqy-microhei wqy-microhei-lite wqy-zenhei
# 思源字体
$ sudo pacman -S noto-fonts-cjk adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts
```

并且在 `~/.config/fontconfig/fonts.conf` 中写入：

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">

<fontconfig>

    <its:rules xmlns:its="http://www.w3.org/2005/11/its" version="1.0">
        <its:translateRule translate="no" selector="/fontconfig/*[not(self::description)]"/>
    </its:rules>

    <description>Manjaro Font Config</description>

    <!-- Font directory list -->
    <dir>/usr/share/fonts</dir>
    <dir>/usr/local/share/fonts</dir>
    <dir prefix="xdg">fonts</dir>
    <dir>~/.fonts</dir> <!-- this line will be removed in the future -->

    <!-- 自动微调 微调 抗锯齿 内嵌点阵字体 -->
    <match target="font">
        <edit name="autohint"> <bool>false</bool> </edit>
        <edit name="hinting"> <bool>true</bool> </edit>
        <edit name="antialias"> <bool>true</bool> </edit>
        <edit name="embeddedbitmap" mode="assign"> <bool>false</bool> </edit>
    </match>

    <!-- 英文默认字体使用 Roboto 和 Noto Serif ,终端使用 DejaVu Sans Mono. -->
    <match>
        <test qual="any" name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="prepend" binding="strong">
            <string>Noto Serif</string>
        </edit>
    </match>
    <match target="pattern">
        <test qual="any" name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="prepend" binding="strong">
            <string>Roboto</string>
        </edit>
    </match>
    <match target="pattern">
        <test qual="any" name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="prepend" binding="strong">
            <string>DejaVu Sans Mono</string>
        </edit>
    </match>

    <!-- 中文默认字体使用思源宋体,不使用 Noto Sans CJK SC 是因为这个字体会在特定情况下显示片假字. -->
    <match>
        <test name="lang" compare="contains">
            <string>zh</string>
        </test>
        <test name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="prepend">
            <string>Source Han Serif CN</string>
        </edit>
    </match>
    <match>
        <test name="lang" compare="contains">
            <string>zh</string>
        </test>
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="prepend">
            <string>Source Han Sans CN</string>
        </edit>
    </match>
    <match>
        <test name="lang" compare="contains">
            <string>zh</string>
        </test>
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="prepend">
            <string>Noto Sans Mono CJK SC</string>
        </edit>
    </match>

    <!-- 把Linux没有的中文字体映射到已有字体，这样当这些字体未安装时会有替代字体 -->
    <match target="pattern">
        <test qual="any" name="family">
            <string>SimHei</string>
        </test>
        <edit name="family" mode="assign" binding="same">
            <string>Source Han Sans CN</string>
        </edit>
    </match>
    <match target="pattern">
        <test qual="any" name="family">
            <string>SimSun</string>
        </test>
        <edit name="family" mode="assign" binding="same">
            <string>Source Han Serif CN</string>
        </edit>
    </match>
    <match target="pattern">
        <test qual="any" name="family">
            <string>SimSun-18030</string>
        </test>
        <edit name="family" mode="assign" binding="same">
            <string>Source Han Serif CN</string>
        </edit>
    </match>
    <!--
    <match target="pattern">
        <test qual="any" name="family">
            <string>Microsoft YaHei</string>
        </test>
        <edit name="family" mode="assign" binding="same">
            <string>Source Han Sans CN</string>
        </edit>
    </match>
    -->

    <!-- Load local system customization file -->
    <include ignore_missing="yes">conf.d</include>
    <!-- Font cache directory list -->
    <cachedir>/var/cache/fontconfig</cachedir>
    <cachedir prefix="xdg">fontconfig</cachedir>
    <!-- will be removed in the future -->
    <cachedir>~/.fontconfig</cachedir>

    <config>
        <!-- Rescan in every 30s when FcFontSetList is called -->
        <rescan> <int>30</int> </rescan>
    </config>

</fontconfig>
```

## 其他软件

```shell
$ yay -S range # 在命令行中浏览文件
$ yay -S colc  # 统计代码数量
$ yay -S screenfetch # 炫酷的电脑信息展示
$ yay -S tldr

# WPS Office
$ yay -S ttf-wps-fonts wps-office

# telegram
$ yay -S telegram

# GIMP
$ yay -S gimp
```

# 遇到的问题

## 双显卡

如果要配置双显卡，记得一定要在 BIOS 里面打开双显卡设置！
我没打开这个设置，还傻傻地找了半天驱动，浪费时间。

## KDE 登陆后黑屏

如果登陆后黑屏，但是能用 `alt + F2` 打开软件，可能是更新时不小心把 `plasma-desktop` 卸载了，需要重新装回来。

```shell
$ sudo pacman -S plasma-desktop
$ rm ~/.config/plasma-org.kde.plasma.desktop-appletsrc # 删掉有问题的配置
```

然后重启。

## 扩展分区

最简单的方法就是重新用 U 盘启动，然后用 U 盘的系统扩展。

注意，不能对自己正在使用的这个卷进行扩展。

# 感想

Manjaro 真的非常体贴，从软件安装到驱动设置，再到 `zsh` 的默认配置，从中可以看到这个发行版的满满的诚意。而且 Emacs 之类的软件在 Linux 下面也是真的好用，启动速度飞快，Mac 上则是有点拖延，Windows 上更不用说了。

但是 Linux 不管怎么说还是可能会遇到各种各样的问题，所以要做好折腾的准备。尤其是中文输入法、双显卡等历史遗留问题。

~~所以我还是想选择 Mac。~~
