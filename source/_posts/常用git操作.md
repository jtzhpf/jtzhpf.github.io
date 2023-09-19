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