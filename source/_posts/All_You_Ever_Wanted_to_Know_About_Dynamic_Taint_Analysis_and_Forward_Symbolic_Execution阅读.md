---
title: "All You Ever Wanted to Know About Dynamic Taint Analysis and Forward Symbolic Execution 阅读"
category: CS&Maths
#id: 57
date: 2023-10-30 09:00:00
tags: 
  - S&P
  - Dynamic Analysis
  - Symbolic Execution
  - Paper
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---

本文详细介绍了动态污点分析（Dynamic Taint Analysis）和前向符号执行（Forward Symbolic Execution）。
<!--more-->

# Dynamic Taint Analysis
Dynamic Taint Analysis的主要思想是标记（或污点）程序中的数据，并跟踪这些标记在程序执行期间的传播。这样可以帮助识别潜在的安全漏洞，尤其是与数据流相关的问题，例如信息泄露、未经授权的数据修改等。

# Forward Symbolic Execution

Forward Symbolic Execution自动地探索程序的执行路径，以便寻找潜在的错误、漏洞或其他安全问题。该技术的核心思想是以符号形式表示程序的输入，并通过模拟程序执行过程来推导出符号执行路径，从而发现潜在的问题。
```mermaid
graph LR;
    A[Symbolic Execution]-->B[Forward Symbolic Execution];
    A[Symbolic Execution]-->C[Backward Symbolic Execution];
```
