---
title: "《Hyperkernel: Push-Button Verification of an OS Kernel》文章精读"
category: CS&Maths
#id: 57
date: 2023-9-3 09:00:00
tags: 
  - xv6
  - Staitc Analysis
  - SOSP
  - 论文
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---

修改xv6的内核接口，构建状态机规范和声明式规范以支持SMT（Satisfiability Modulo Theories）求解器来进行验证。通过有限化内核接口，使用硬件虚拟化简化虚拟内存推理，并在LLVM IR级别工作，以避免对C语义进行建模，从而实现了"一键验证"。
<!--more-->

## 规范
程序员使用Python编写两种规范来指定系统调用的期望行为：详细的**状态机规范**和更高级的**声明式规范**。两种规范都用Python表达，并使用Z3 SMT求解器进行验证。程序员使用C实现系统调用。验证器将Python规范和C编译成的LLVM IR实现转化为SMT查询，并调用Z3进行验证。验证后的代码与未验证的（可信任的）代码链接，生成最终的内核映像。

![](/《Hyperkernel: Push-Button Verification of an OS Kernel》文章精读/image1.png)

使用SMT求解器的优点是，如果验证失败，它能够生成一个测试用例，有助于定位和修复错误。验证器会生成一个具体的测试用例，包括内核状态和系统调用参数，以描述如何触发错误。如果两者之间存在不一致，验证器会显示违规情况。