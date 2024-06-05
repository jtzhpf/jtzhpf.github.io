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
文章中主要将线程错误类型分为2大类：

  - **Blocking Bugs**
    - **Mutex and RwLock**
      ![](/Understanding_Memory_and_Thread_Safety_Practices_and_Issues_in_Real-World_Rust_Programs阅读/image3.png)<br>

      尽管双锁和锁顺序冲突等问题在传统语言中也很常见，但 Rust 复杂的生命周期规则及其隐式解锁机制使程序员更难编写无阻塞错误的代码。

      变量client是由RwLock保护的Inner对象。在第3行，它的读取锁被获取，并且其m字段被用作调用函数connect()的输入。如果connect()返回Ok，则在第7行获取写锁，并且在第8行修改内部对象。第7行处的写锁将导致双重锁定，因为由client.read()返回的临时引用持有对象的生命周期跨越整个匹配代码块，并且读取锁一直保持到第11行。修补程序是将connect()的返回保存到一个局部变量中，以在第4行释放读锁，而不是直接将返回值用作匹配代码块的条件。

    - **Condvar**
    - **Channel**
  - **Non-Blocking Bugs**
    ![](/Understanding_Memory_and_Thread_Safety_Practices_and_Issues_in_Real-World_Rust_Programs阅读/image4.png)<br>
    AuthorityRound是一个实现了Sync特性的结构体（因此，声明为Arc后，AuthorityRound对象可以被多个线程共享）。proposed 字段是一个原子布尔变量，初始化为false。函数generate_seal()的意图是一次只返回一个Seal对象，而程序员（不当地）在第3行和第4行使用 proposed 字段来实现这个目标。当两个线程在同一个对象上调用generate_seal()，并且它们都在执行第3行之前执行第4行之前完成时，两个线程都会得到一个Seal对象作为函数的返回值，违反了程序的预期目标。修补程序是在第6行使用原子指令来替换第3行和第4行。

    在此错误代码中，generate_seal() 函数通过更改 proposed 字段的值来修改不可变借用参数 &self。如果函数的输入参数设置为&mut self（可变借用），则当在没有持有锁的情况下调用generate_seal()时，Rust编译器将报告错误。换句话说，如果程序员使用可变借用，那么他们就可以在 Rust 编译器的帮助下避免该错误。

# Suggestions
- IDE 对生存期进行可视化分析
- 静态分析（本文采用的实验方法）
  - 针对 Memory Safety Issues，通过 MIR 中的 StorageLive / StorageDead 来对生存期进行分析，并维护好变量之间的引用（所有权）关系；
  - 针对 Thread Safety Issues，通过识别所有lock()的调用点，并从每个调用点提取正在获取的锁的信息以及用于保存返回值的变量。由于Rust在该变量的生命周期结束时隐式释放锁，因此记录此释放时间，然后检查在此时间之前是否已经获取了相同的锁，并在这种情况下报告双重锁定错误。此外，当一个结构体是可共享的（例如，实现 Sync 特征）并且有一个方法不可变地借用 self 时，我们可以分析 self 是否在方法中被修改以及修改是否是不同步的。
- 动态分析

# 实验
实验具体代码位于[https://github.com/system-pclub/rust-study](https://github.com/system-pclub/rust-study)。采用的方法是打印出 MIR，然后 Python/C++ 编写 detector 来分析。

缺点：

- 只能分析项目本级的 MIR，无法将项目及其所依赖的 crate 作为整体分析，若想分析所依赖 crate，必须单独进行分析。
- 没有考虑到数值的合理性。