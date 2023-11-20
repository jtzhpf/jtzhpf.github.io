---
title: "Clang 阅读随笔"
category: CS&Maths
#id: 57
date: 2023-11-20 09:00:00
tags: 
  - LLVM
  - Clang
  - Compiler
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---
这是在阅读 Clang 时遇到不会内容时的随笔，看到哪写到哪，可能不会有任何的结构（Who knows）。

😘 在此先疯狂鸣谢一波 ChatGPT 和 Claude 😍。

<!--more-->

## CompilerInstance 类
CompilerInstance 是 LLVM/Clang 编译器框架中的一个类，用于管理整个编译过程中的状态和组件。它负责协调和组织编译器的不同部分，包括预处理器、语法分析器、语义分析器等。在编译的不同阶段，CompilerInstance 提供了访问和控制这些组件的方法。

### CompilerInstance 的结构
CompilerInstance 的具体结构可能会有一些复杂，因为它需要协调各个编译器组件的工作。一般而言，它包含了以下关键成员：

- 语法树（AST）： 用于表示源代码结构的树形数据结构。AST 在编译器的语法分析阶段构建，以便更容易进行语义分析和代码生成。
- 预处理器（Preprocessor）： 负责对源代码进行预处理，包括宏展开、条件编译、去注释等。预处理器生成预处理后的源代码，供后续的编译阶段使用。
- 语义分析器（Sema）： 在 AST 的基础上进行语义分析，执行类型检查、符号解析等操作。语义分析是编译器的重要阶段，它确保程序语义的正确性。
- 代码生成器（Code Generator）： 将语义分析阶段得到的中间表示（IR）转换为目标代码。这是编译器的最后阶段。
- 源码管理器（SourceManager）： 负责跟踪源代码的位置信息，以便在编译过程中进行错误报告和调试信息生成。
- 前端选项（FrontendOptions）： 包含与编译过程相关的各种选项，如编译目标、优化级别等。

这只是 CompilerInstance 结构的一般概述，具体实现可能更为复杂。

## Static Analyzer

Static Analyzer 的源代码入口主要位于 Clang 代码库的 `lib/StaticAnalyzer` 目录中。以下是几个关键文件和目录：

- **`lib/StaticAnalyzer/Core`：** 包含 Static Analyzer 的核心实现，如路径敏感性分析器、检查器管理器等。
- **`lib/StaticAnalyzer/Checkers`：** 包含各种 Checker 的实现，每个 Checker 都有一个对应的目录，其中包含 Checker 的具体实现和规则定义。
- **`lib/StaticAnalyzer/Frontend`：** 包含与前端集成相关的代码，处理命令行参数，设置分析配置等。
- **`lib/StaticAnalyzer/PathSensitive`：** 包含路径敏感性分析的相关实现。
- **`lib/StaticAnalyzer/Frontend/AnalysisConsumer.cpp`：** 这个文件定义了 `AnalysisConsumer` 类，它是 Static Analyzer 在编译过程中的 AST 消费者，负责驱动整个静态分析过程。
- **`lib/StaticAnalyzer/Core/CheckerManager.cpp`：** 这个文件包含 `CheckerManager` 类的实现，负责管理所有的 Checker。
- **`tools/scan-build`：** 该工具用于通过 Clang Static Analyzer 进行静态分析。
  
调用情况如图所示：
![Clang Static Analyzer 调用情况](/Clang阅读随笔/image1.png)

总体而言，`AnalysisConsumer.cpp` 和 `CheckerManager.cpp` 可以看作是 Static Analyzer 的源代码入口，它们定义了整个分析过程的驱动和管理逻辑。其他目录则包含了各个组件的具体实现。

### Clang 静态分析模式
在 Clang 的静态分析器（`/llvm/tools/clang/lib/StaticAnalyzer/Frontend/AnalysisConsumer.cpp: HandleTranslationUnit()`）中，`AM_Syntax` 和 `AM_Path` 分别代表不同的分析模式，用于控制静态分析的行为。以下是它们的区别：

1. **`AM_Syntax`（语法分析模式）：**
   - **含义：** `AM_Syntax` 表示语法分析模式，指的是在分析中仅关注语法层面的结构，而不考虑程序的具体路径执行信息。
   - **行为：** 在 `AM_Syntax` 模式下，静态分析器主要关注程序的语法结构，执行基本的语法检查和分析，例如识别语法错误、检查变量的声明和使用情况等。这种模式下的分析通常更快，但可能会错过一些路径敏感性的问题。

2. **`AM_Path`（路径敏感分析模式）：**
   - **含义：** `AM_Path` 表示路径敏感分析模式，指的是在分析中考虑程序的具体路径执行信息，以检测可能的路径相关问题。
   - **行为：** 在 `AM_Path` 模式下，静态分析器会模拟程序的不同执行路径，考虑程序在不同条件下的行为。这种模式下的分析可以发现更多的潜在问题，例如路径上的条件分支错误、空指针解引用等。然而，路径敏感分析通常会增加分析的复杂性和执行时间。

总的来说，区别在于分析是否关注程序的路径执行信息。`AM_Syntax` 主要关注语法结构，而 `AM_Path` 则在此基础上考虑了路径敏感性，以更全面地发现潜在问题。

[Cross-checking Semantic Correctness: The Case of Finding File System Bugs](/2023/08/31/Cross-checking_Semantic_Correctness:The_Case_of_Finding_File_System_Bugs阅读/) 这篇文章中使用的是`AM_Syntax`。

