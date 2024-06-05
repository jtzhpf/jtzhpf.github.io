---
title: "Understanding and Detecting Real-World Safety Issues in Rust 阅读"
category: CS&Maths
#id: 57
date: 2024-3-26 09:00:00
tags: 
  - Staitc Analysis
  - Paper
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
mathjax: true
---

本文是 [Understanding Memory and Thread Safety Practices and Issues in Real-World Rust Programs](https://jtzhpf.github.io/2024/03/25/Understanding_Memory_and_Thread_Safety_Practices_and_Issues_in_Real-World_Rust_Programs阅读/) 的续作，在前文 70 个内存安全问题和 100 个线程安全问题的基础上，增加了对 110 个 panic 问题的研究。

# Unexpected Panic Issues
文章中主要将 Panic 问题类型分为5类：

  - **Missing error handling**<br>
    Rust提供了Result和Option枚举，供被调用者返回值（Ok(T)或Some(T)）或通知调用者遇到的执行错误（Err(E)或None）。此外，为了提高编程便利性，Rust允许程序员假设被调用函数总是成功执行，并直接使用unwrap()或expect()返回的Result（或Option）对象上检索实际的返回值。然而，如果这种假设是错误的，unwrap()（或expect()）调用会触发恐慌。<br>
    ![](/Understanding_and_Detecting_Real-World_Safety_Issues_in_Rust阅读/image1.png)
    图10说明了Servo中的一个例子，以此来说明这个类别。程序员错误地假设第3行的parse()函数调用总是会返回Ok(Url)，从而导致他在返回的Result对象上调用unwrap()。然而，parse()函数调用可能会失败，并返回Err(ParseError)。因此，会触发恐慌。
  - **Wrong arithmetic operations**<br>
    for example, overflow, dividing by zero, casting a value to a wrong type
  - **Assertion errors**<br>
    程序员手动申明断言导致的，例如panic!(), unimplemented!(), unreachable!(), assert!()。
  - **Out of bounds**
  - **Others**
    例如，触发无限递归函数调用时堆栈溢出，调用了错误的库函数。

# Suggestions
- IDE 对生存期进行可视化分析
- 静态分析（本文采用的实验方法）
  - 针对 Memory Safety Issues，通过 MIR 中的 StorageLive / StorageDead 来对生存期进行分析，并维护好变量之间的引用（所有权）关系；
  - 针对 Thread Safety Issues，通过识别所有lock()的调用点，并从每个调用点提取正在获取的锁的信息以及用于保存返回值的变量。由于Rust在该变量的生命周期结束时隐式释放锁，因此记录此释放时间，然后检查在此时间之前是否已经获取了相同的锁，并在这种情况下报告双重锁定错误。此外，当一个结构体是可共享的（例如，实现 Sync 特征）并且有一个方法不可变地借用 self 时，我们可以分析 self 是否在方法中被修改以及修改是否是不同步的。
- 动态分析

# 实验
实验具体代码位于[https://github.com/BurtonQin/lockbud](https://github.com/BurtonQin/lockbud)。采用的方法是打印出 MIR，然后 Python/C++ 编写 detector 来分析。

已做到接近完美，内存的所有错误都能查找出（得益于rust语言直接标注了 StorageLive 和 StorageDead）

Thread Safety Issues 存在误报，因为 rust MIR 无显式的 unlock() 语句，需要 gen-kill 算法来计算 unlock 的位置。

Unexpected Panic Issues 也很简单，直接对着 Panic 语句来对代码进行检查。