---
title: "《Hyperkernel: Push-Button Verification of an OS Kernel》文章精读"
category: CS&Maths
#id: 57
date: 2023-9-3 09:00:00
tags: 
  - Linux
  - Staitc Analysis
  - SOSP
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---

通过修改xv6的内核接口以支持SMT（Satisfiability Modulo Theories）求解器。
<!--more-->
## 挑战与应对


|     | 挑战                                                                                                                                                                                                                                                     | 应对                                                                                                                                                                                                                                                                                                                             |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 一  | 内核维护着大量的数据结构和不变量来管理进程、虚拟内存以及设备等资源，Hyperkernel的接口需要支持高级属性(如进程隔离)的规范和验证。<br>同时这些规范又需要可以转换为SMT求解器可以处理的低级约束。<br>需要合理的接口设计使得在可用性和验证自动化之间取得平衡。 | 所有的系统调用、异常和中断的处理程序都不包含无限的循环和递归，使其可以使用SMT对其进行编码和验证。                                                                                                                                                                                                                                |
| 二  | 在x86架构中，内核和用户空间通常存在于相同的虚拟地址空间中，内核占据上半部分，而留给用户空间的是下半部分。内存的映射可能不是单射，例如两个虚拟地址映射到同一物理地址，且内核代码在执行时可以改变虚拟地址到物理地址的映射。                                | Hyperkernel在与用户空间分离的独立地址空间中运行，使用了identity mapping（将虚拟地址空间中内核区域的虚拟地址直接映射到物理地址，实现虚拟地址等同物理地址）。<br>为了有效地实现这种分离，Hyperkernel利用了由Intel的VTx和AMD的AMD-V提供的x86虚拟化支持，使内核和用户进程分别在根（host）和非根（guest）模式下运行，使用独立的页表。 |
| 三  | Hyperkernel是使用C语言编写的。C语言的低级操作（如访存等）使得精确建模C语言的语义和推理C程序变得困难。且编译器利用未定义行为以生成高效的代码。                                                                                                            | Hyperkernel在LLVM中间表示(IR)层面进行验证，相比C语言它具有更简单的语义，同时仍保持足够高级以避免推理机器细节。                                                                                                                                                                                                                   |

## Tricks
### Finite interfaces
将在执行中随着系统变化而变化代码执行次数的路径修改为与系统状态无关的路径。

追求最低可用文件描述符会导致代码路径爆炸。内核需要逐个检查所有可能的文件描述符,以找到最小的空闲描述符。SMT求解器需要展开这些代码路径来构建约束和验证程序。代码路径的数量会随着文件描述符表大小呈指数增长。验证时间和内存消耗都将随之大幅增加,难以完成验证。复杂性会随状态空间大小指数增长。即使可验证,也很难扩展到处理大规模的文件描述符情况,不易应用于实际的大型系统。这违反了有限状态验证的要求。接口的非有限性成为SMT验证的严重障碍。Hyperkernel通过修改dup的接口语义进行有限化:允许用户选择文件描述符号,而不是自动选择最小号。这样SMT求解器只需要展开一个常量级的代码路径,大大降低了验证复杂度。使验证成为与状态空间大小无关的常量时间操作,可以可扩展地适应大规模系统。

## 设计

1. To enable comparison, the source code of each file system is merged into one large file. This allows interprocedural analysis within a file system.
2. JUXTA collects execution information for each function in the file systems by constructing control-flow graphs (CFGs) and symbolically exploring these graphs from the entry point to the end of each function.
3. Symbols are canonicalized so that equivalent symbols have the same name across file systems. A path database that stores the extracted path information is also created, which allows applications like checkers and specification extractors to use the path information without reexecuting symbolic analysis. All of these facilitate comparison.
4. Symbolic execution is used to explore paths through functions and build a path database. The path info contains path conditions, return values, side effects, etc.
5. Two statistical methods are used to compare paths:
   - Histogram-based: Integer ranges are encoded into histograms and distances between histograms are computed.
   - Entropy-based: Used for events like flags or return value checks. Low non-zero entropy indicates a likely bug.
6. Bug reports are ranked to prioritize investigation of likely true positives. Metrics include histogram distance and entropy values.

## 实现

需要构建8个检查器：

1. 返回值检查器：比较不同文件系统对同一VFS接口的返回值，找到返回值有明显差异的文件系统。
2. 副作用检查器：比较同一VFS接口和返回值下的副作用，找到副作用有明显差异的文件系统。
3. 函数调用检查器：比较同一VFS接口下的函数调用，找到调用函数有明显差异的文件系统。
4. 路径条件检查器：比较同一VFS接口下的路径条件，找到路径条件有明显差异的文件系统。
5. 提取文件系统规范：从不同文件系统中提取出共性行为作为潜在规范。
6. 重构跨模块抽象：根据共性行为，可将其提升到VFS层，减少冗余。
7. 推断锁语义：推断每个路径所持有和释放的锁，跟踪受锁保护的字段。
8. 推断外部API语义：基于调用参数和返回值检查的频率分布，找出用法有明显差异的文件系统。

## 问题

1. 依赖于大量实现相同功能的模块
