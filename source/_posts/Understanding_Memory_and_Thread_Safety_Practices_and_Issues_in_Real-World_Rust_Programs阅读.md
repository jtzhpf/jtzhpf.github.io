---
title: "Understanding Memory and Thread Safety Practices and Issues in Real-World Rust Programs 阅读"
category: CS&Maths
#id: 57
date: 2024-3-25 09:00:00
tags: 
  - Staitc Analysis
  - Symbolic Execution
  - PLDI
  - PLDI'20
  - Paper
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
mathjax: true
---

本文研究涵盖了五个基于 Rust 的系统和应用程序（两个操作系统、一个浏览器、一个键值存储系统和一个区块链系统）、五个广泛使用的 Rust 库和两个在线漏洞数据库。总共研究了 850 个不安全代码用法、70 个内存安全问题和 100 个线程安全问题。

本文发现**所有内存安全错误都涉及不安全代码**，并且其中大多数也涉及安全代码。当程序员编写安全代码而没有注意其他相关代码不安全时，很容易发生错误。本文还发现 Rust 中生命周期的范围很难推理，特别是与不安全代码结合使用时，对生命周期的错误理解会导致许多内存安全问题。

本文还研究并发错误，包括非阻塞和阻塞错误。本文发现**非阻塞错误可能发生在不安全和安全代码中**，并且本文研究的**所有阻塞错误都发生在安全代码中**。尽管 Rust 中的许多错误模式遵循传统的并发错误模式（例如双锁、原子性违规），但 Rust 中的许多并发错误是由于程序员对 Rust（复杂）生命周期和安全规则的误解造成的。

本文构建了两个静态错误检测器（一个用于释放后使用错误，一个用于双锁错误）来对 Rust 错误检测进行了初步探索，发现了 10 个以前未知的错误。

# Memory Safety Issues
文章中主要将内存错误类型分为2大类，6小类：

- **Wrong Access**
  - **Buffer overflow**
  - **Null pointer dereferencing**
  - **Read uninitialized memory**
- **Lifetime Violation**
  - **Invalid free**<br>
      ![](/Understanding_Memory_and_Thread_Safety_Practices_and_Issues_in_Real-World_Rust_Programs阅读/image1.png)<br>
      变量 f 是一个指向未初始化内存缓冲区的指针，其大小与 struct FILE 相同（第 6 行）。在第 7 行将一个新的 FILE 结构赋给 \*f，会结束 f 指向的先前结构体的生命周期，导致 Rust 释放先前的结构体。所有先前结构体分配的内存都将被释放（例如，第 2 行的 buf 中的内存）。然而，由于先前的结构体包含一个未初始化的内存缓冲区，释放其堆内存是无效的。请注意，**这种行为是 Rust 独有的**，在传统语言中不会发生（例如，在 C/C++ 中 \*f=buf 不会导致 f 指向的对象被释放）。
  - **Use after free**<br>
      ![](/Understanding_Memory_and_Thread_Safety_Practices_and_Issues_in_Real-World_Rust_Programs阅读/image2.png)<br>
      当输入数据有效时，在第3行创建了一个BioSlice对象，并将其地址分配给指针p在第2行。p用于调用不安全函数CMS_sign()在第12行，并且在该函数内部被解引用。然而，BioSlice对象的生命周期在第5行结束，并且对象将在那里被丢弃。因此，使用p是在对象被释放后。
  - **Double free**

# Thread Safety Issues
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

## Suggestions
- IDE 对生存期进行可视化分析
- 静态分析
  - 针对 Memory Safety Issues，通过 MIR 中的 StorageLive / StorageDead 来对生存期进行分析，并维护好变量之间的引用（所有权）关系；
  - 针对 Thread Safety Issues，通过识别所有lock()的调用点，并从每个调用点提取正在获取的锁的信息以及用于保存返回值的变量。由于Rust在该变量的生命周期结束时隐式释放锁，因此记录此释放时间，然后检查在此时间之前是否已经获取了相同的锁，并在这种情况下报告双重锁定错误。此外，当一个结构体是可共享的（例如，实现 Sync 特征）并且有一个方法不可变地借用 self 时，我们可以分析 self 是否在方法中被修改以及修改是否是不同步的。
- 动态分析

## 实验
实验具体代码位于[https://github.com/system-pclub/rust-study](https://github.com/system-pclub/rust-study)。采用的方法是打印出 MIR，然后 Python/C++ 编写 detector 来分析。