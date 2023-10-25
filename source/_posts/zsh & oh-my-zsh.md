---
title: zsh & oh-my-zsh 常用命令
category: CS&Maths
#id: 57
date: 2023-9-5 18:42:32
tags: 
  - Linux
  - shell
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: app  # 展示在时间线列表中
---

鉴于经常需要配置环境，在此记录一下 zsh & oh-my-zsh 的常用命令。
<!--more-->

## zsh

zsh 是一款强大的虚拟终端，既是一个系统的虚拟终端，也可以作为一个脚本语言的交互解析器。

```shell
# 打开终端，在终端上输入: 
zsh --version

# 这个命令来查看我们的电脑上是否安装了 zsh 
```

```shell
# 查看系统当前 shell
cat /etc/shells 
```

```shell
# 修改当前用户 shell，注销后生效
chsh -s /bin/zsh
```

## 安装 oh my zsh

可以通过 curl 或 wget 两种方式来安装，用一条命令即可安装。

### curl 安装

GitHub：
```shell
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
Gitee ( 国内镜像 )：
```shell
sh -c "$(curl -fsSL https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh)"
```

### wget 安装

GitHub：
```shell
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```
Gitee ( 国内镜像 )：
```shell
sh -c "$(wget -O- https://gitee.com/pocmon/mirrors/raw/master/tools/install.sh)"
```

## 常用的 oh my zsh 插件

打开`~/.zshrc`文件找到`plugins = (git)`，这里是我们已经启用了那些插件

如果想要启用某个插件，装好之后直接修改`plugins = (插件A 插件B 插件C)`

### git

这个是装好 oh-my-zsh 就默认已经开启的

### z

这个是 oh-my-zsh 默认就装好的，需要自己开启。还有一个 autojump 插件和 z 功能差不多，autojump需要自己装

如果 z 插件历史记录太多，并且有一些不是自己想要的，可以用`z -x 不要的路径`删除

### zsh-autosuggestions

会记录你之前输入过的所有命令，并且自动匹配你可能想要输入命令，然后按`tab`补全

安装方法：

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

### zsh-syntax-highlighting

命令太多，有时候记不住，等输入完了才知道命令输错了，这个插件直接在输入过程中就会提示你，当前命令是否正确，错误红色，正确绿色

安装方法：

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

### sudo

偶尔输入某个命令，提示没有权限，需要加`sudo`，这个时候按两下`ESC`，就会在命令行头部加上`sudo`

## 解决 zsh 在 git 目录下变得卡顿的问题

在目录下通过设置标识关闭 dirty 检查：
```shell
git config --add oh-my-zsh.hide-dirty 1
```

若需要打开，则：
```shell
git config --add oh-my-zsh.hide-dirty 0
```