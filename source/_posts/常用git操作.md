---
title: "常用git操作"
category: Life
#id: 57
date: 2023-9-16 20:42:32
tags: 
  - git
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: app  # 展示在时间线列表中
---
记录一下常用的git操作。
<!--more-->

## 合并 git push 记录
首先，在本地仓库中，确保工作目录是干净的，没有未提交的更改。可以使用 `git status` 命令来检查工作目录的状态。

执行以下命令，将本地分支的历史合并为一次提交：

```shell
git reset --soft HEAD~n
```
其中，`n` 是想要合并的提交次数。这个命令将会将最近的 `n` 次提交合并为一个暂存区的更改，但不会删除它们。

然后，使用以下命令创建一个新的提交，将这些更改提交到本地仓库：
```shell
git commit -m "合并多次提交"
```

最后，执行强制推送到远程仓库：

```shell
git push -f origin <branch_name>
```
其中，`<branch_name>` 是要推送的分支的名称。
这将覆盖远程仓库上的提交历史，并将多次提交合并为一次提交。请确保在执行强制推送之前，已经备份了远程仓库或确信不会造成数据丢失。

## tag 命令
### 打印所有标签
```shell
git tag
```

### 打印符合检索条件的标签
```shell
git tag -l <版本号>
```
如 `git tag -l 1.*.*` 为搜索一级版本为1的版本。

### 查看对应标签状态
```shell
git checkout <版本号>
```

### 创建轻量标签
轻量标签指向一个发行版的分支，其只是某commit的引用，不存储名称时间戳及标签说明等信息。
```shell
git tag <版本号>-light
```

### 创建带附注标签
相对于轻量标签，附注标签是一个独立的标签对象，包含了名称时间戳以及标签备注等信息，同时指向对应的commit。
```shell
git tag -a <版本号> -m "<备注信息>"
```
同时我们也可以向特定的commit添加标签，使用该commit对应的SHA值即可。
```shell
git tag -a <版本号> <SHA值> -m "<备注信息>"
```
比如 `git tag -a 1.0.0 0c3b62d -m "Release Edition v1.0.0"` 就是为SHA为0c3b62d的这次提交打了1.0发行版的tag。

### 删除本地标签
```shell
git tag -d <版本号>
```

### 推送所有标签
```shell
git push origin --tags
```

### 推送指定版本的标签
```shell
git push origin <版本号>
```

### 删除远程仓库的标签
新版本Git (> v1.7.0)
```shell
git push origin --delete <版本号>
```
新旧版本通用方法
```shell
git push origin :refs/tags/<版本号>
```

