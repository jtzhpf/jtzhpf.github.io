---
title: "MirChecker: Detecting Bugs in Rust Programs via Static Analysis 阅读"
category: CS&Maths
#id: 57
date: 2024-3-11 09:00:00
tags: 
  - Staitc Analysis
  - Symbolic Execution
  - CCS
  - CCS'21
  - Paper
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
mathjax: true
---

本文主要从以下2点来进行错误的检测：

- 利用 Rust 编译器在编译时为无法检查的安全条件自动生成的断言，以其为验证条件，检查例如越界访问和整数溢出等问题。
- Unsafe code 破坏所有权系统，导致悬空指针、共享可变别名等问题。
<!--more-->

![](/Mirchecker:Detecting_Bugs_in_Rust_Programs_via_Static_Analysis阅读/image1.png)