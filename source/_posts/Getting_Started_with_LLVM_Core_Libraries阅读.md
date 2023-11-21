---
title: "Getting Started with LLVM Core Libraries 阅读"
category: CS&Maths
#id: 57
date: 2023-11-21 09:00:00
tags: 
  - Staitc Analysis
  - Symbolic Execution
  - LLVM
  - Book
  - Clang
  - Compiler
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
mathjax: true
---

这本书是在找 Static Analyzer 时意外发现的，其余部分暂时不打算理睬，先重点关注 Static Analyzer 部分。

<!--more-->---

## Forward Dataflow Analysis
![Forward Dataflow Analysis 的过程以及可能出现的 False Positive 的情况](/Getting_Started_with_LLVM_Core_Libraries阅读/image1.png)

## Symbolic Execution
![利用 Symbolic Execution 对上述例子进行分析](/Getting_Started_with_LLVM_Core_Libraries阅读/image2.png)

![](/Getting_Started_with_LLVM_Core_Libraries阅读/image3.png)

![Symbolic Execution 相较于 Forward Dataflow Analysis，其同样合并了部分节点，但不能合并的又没有强行合并](/Getting_Started_with_LLVM_Core_Libraries阅读/image4.png)

