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

# CompilerInstance 类
CompilerInstance 是 LLVM/Clang 编译器框架中的一个类，用于管理整个编译过程中的状态和组件。它负责协调和组织编译器的不同部分，包括预处理器、语法分析器、语义分析器等。在编译的不同阶段，CompilerInstance 提供了访问和控制这些组件的方法。

## CompilerInstance 的结构
CompilerInstance 的具体结构可能会有一些复杂，因为它需要协调各个编译器组件的工作。一般而言，它包含了以下关键成员：

- 语法树（AST）： 用于表示源代码结构的树形数据结构。AST 在编译器的语法分析阶段构建，以便更容易进行语义分析和代码生成。
- 预处理器（Preprocessor）： 负责对源代码进行预处理，包括宏展开、条件编译、去注释等。预处理器生成预处理后的源代码，供后续的编译阶段使用。
- 语义分析器（Sema）： 在 AST 的基础上进行语义分析，执行类型检查、符号解析等操作。语义分析是编译器的重要阶段，它确保程序语义的正确性。
- 代码生成器（Code Generator）： 将语义分析阶段得到的中间表示（IR）转换为目标代码。这是编译器的最后阶段。
- 源码管理器（SourceManager）： 负责跟踪源代码的位置信息，以便在编译过程中进行错误报告和调试信息生成。
- 前端选项（FrontendOptions）： 包含与编译过程相关的各种选项，如编译目标、优化级别等。

这只是 CompilerInstance 结构的一般概述，具体实现可能更为复杂。

# Static Analyzer

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

## Clang 静态分析模式
在 Clang 的静态分析器（`/llvm/tools/clang/lib/StaticAnalyzer/Frontend/AnalysisConsumer.cpp: HandleTranslationUnit()`）中，`AM_Syntax` 和 `AM_Path` 分别代表不同的分析模式，用于控制静态分析的行为。以下是它们的区别：

1. **`AM_Syntax`（语法分析模式）：**
   - **含义：** `AM_Syntax` 表示语法分析模式，指的是在分析中仅关注语法层面的结构，而不考虑程序的具体路径执行信息。
   - **行为：** 在 `AM_Syntax` 模式下，静态分析器主要关注程序的语法结构，执行基本的语法检查和分析，例如识别语法错误、检查变量的声明和使用情况等。这种模式下的分析通常更快，但可能会错过一些路径敏感性的问题。

2. **`AM_Path`（路径敏感分析模式）：**
   - **含义：** `AM_Path` 表示路径敏感分析模式，指的是在分析中考虑程序的具体路径执行信息，以检测可能的路径相关问题。
   - **行为：** 在 `AM_Path` 模式下，静态分析器会模拟程序的不同执行路径，考虑程序在不同条件下的行为。这种模式下的分析可以发现更多的潜在问题，例如路径上的条件分支错误、空指针解引用等。然而，路径敏感分析通常会增加分析的复杂性和执行时间。

总的来说，区别在于分析是否关注程序的路径执行信息。`AM_Syntax` 主要关注语法结构，而 `AM_Path` 则在此基础上考虑了路径敏感性，以更全面地发现潜在问题。

[Cross-checking Semantic Correctness: The Case of Finding File System Bugs](/2023/08/31/Cross-checking_Semantic_Correctness:The_Case_of_Finding_File_System_Bugs阅读/) 这篇文章中使用的是`AM_Syntax`。

# Control Flow Graph
控制流图（control-flow graph）简称CFG，是计算机科学中的表示法，利用数学中图的表示方式，标示计算机程序执行过程中所经过的所有路径。

节点表示基本块（basic block），边表示控制流的流向。Basic Block是CFG的主体。

> Basic Block：一个最长的语句序列，并保证入口只能在最开始指令且出口只能在最后一个指令。

## 构造Basic Blocks

- Input：程序P的三地址码序列
- Output：程序P的basic blocks
- 算法
  - 确定leaders（每个basic block的头）
    - 序列中的第一个指令
    - 跳转指令的目标指令
    - 跳转指令的下一条指令
    - return指令
  - 每个Basic Block包含其leader至下一个leader前的所有语句
- 算法的另一种阐释
  - 初始语句作为第一个基本块的入口
  - 遇到标号类语句，结束当前基本块，标号作为新基本块的入口（标号不在当前基本块中，而是划到下一个基本块）
  - 遇到转移类语句时，结束当前当前基本块，转移语句作为当前基本块的结尾
  - 当给引用类型变量赋值时，结束基本块，作为出口
- 基本块有以下特点：
  - 单一入口点，其他程式中，没有任何一个分支指令的目标在这段程式基本块之内（基本块的第一行除外）。
  - 单一结束点，这段程式一定要执行完最后一行才会执行其他基本块的程式。
  - 因为上述特点，基本块中的程式，只要执行了第一行，后面的程式码就会依序执行，每一行程式都会执行一次。

## 构造CFG

添加边，在以下两种情况下：

- 无条件跳转：一条无条件跳转语句会创建一个有向边，将当前基本块的出口指向目标基本块的入口。
- 条件跳转：创建两条有向边，分别表示条件为真和条件为假时的目标基本块。
  

在构建控制流图（CFG）时，通常会引入两个特殊的节点，即entry节点和exit节点，以更好地表示程序的整体控制流。这两个节点不对应具体的代码块，而是代表程序的入口和出口。

- entry节点： entry节点表示程序的起始点。它没有对应的代码，而是作为整个控制流图的入口。从entry节点开始，程序的执行流程进入其他基本块。
- exit节点： exit节点表示程序的结束点。类似地，它也没有对应的代码，而是作为整个控制流图的出口。程序的执行流程可能从不同的基本块经过，最终都会汇聚到exit节点。


**示例：**

考虑以下伪代码：

```
1.  if condition
2.    statement1
3.  else
4.    statement2
5.  endif
6.  statement3
```


1. 划分基本块：

   - 基本块1: Line. 1
   - 基本块2: Line. 2 - 3
   - 基本块3: Line. 4
   - 基本块4: Line. 5 - 6
  
    `else` 为转移类语句，`endif` 为标号类语句。
2. 构建基本块间的控制流关系：

   - 从基本块1到基本块2，因为有条件跳转语句。
   - 从基本块1到基本块3，因为有条件跳转语句。
   - 从基本块2到基本块4，因为是顺序执行。
   - 从基本块3到基本块4，因为是顺序执行。

3. 建立CFG：

  ```mermaid
  graph TD
  
    entry --> 1[<div style="text-align:left;">1.  if condition</div>]
    1 --> 2[<div style="text-align:left;">2.    statement1<br>3.  else</div>]
    1 --> 3[<div style="text-align:left;">4.    statement2</div>]
    2 --> 4[<div style="text-align:left;">5.  endif<br>6.  statement3</div>]
    3 --> 4
    4 --> exit

    style entry fill:#98FB98,stroke:#4CAF50,stroke-width:2px;
    style exit fill:#98FB98,stroke:#4CAF50,stroke-width:2px;

  ```

# Static Analyzer Checker
`clang -cc1 -analyzer-checker-help` 命令可以查看 Clang 所支持的所有的 Static Analyzer Checker。单个 Checker 是 Static Analyzer 在代码中执行的单个分析单元。每个分析都针对特定类型的错误，静态分析器允许选择符合需求的任意一部分检查器。

`clang -cc1 -analyzer-checker-help` 命令输出如下：
```
OVERVIEW: Clang Static Analyzer Checkers List

USAGE: -analyzer-checker <CHECKER or PACKAGE,...>

CHECKERS:
  alpha.core.BoolAssignment       Warn about assigning non-{0,1} values to Boolean variables
  alpha.core.CallAndMessageUnInitRefArg
                                  Check for logical errors for function calls and Objective-C message expressions (e.g., uninitialized arguments, null function pointers, and pointer to undefined variables)
  alpha.core.CastSize             Check when casting a malloc'ed type T, whether the size is a multiple of the size of T
  alpha.core.CastToStruct         Check for cast from non-struct pointer to struct pointer
  alpha.core.FixedAddr            Check for assignment of a fixed address to a pointer
  alpha.core.IdenticalExpr        Warn about unintended use of identical expressions in operators
  alpha.core.PointerArithm        Check for pointer arithmetic on locations other than array elements
  alpha.core.PointerSub           Check for pointer subtractions on two pointers pointing to different memory chunks
  alpha.core.SizeofPtr            Warn about unintended use of sizeof() on pointer expressions
  alpha.core.TestAfterDivZero     Check for division by variable that is later compared against 0. Either the comparison is useless or there is division by zero.
  alpha.cplusplus.VirtualCall     Check virtual function calls during construction or destruction
  alpha.deadcode.UnreachableCode  Check unreachable code
  alpha.osx.cocoa.Dealloc         Warn about Objective-C classes that lack a correct implementation of -dealloc
  alpha.osx.cocoa.DirectIvarAssignment
                                  Check for direct assignments to instance variables
  alpha.osx.cocoa.DirectIvarAssignmentForAnnotatedFunctions
                                  Check for direct assignments to instance variables in the methods annotated with objc_no_direct_instance_variable_assignment
  alpha.osx.cocoa.InstanceVariableInvalidation
                                  Check that the invalidatable instance variables are invalidated in the methods annotated with objc_instance_variable_invalidator
  alpha.osx.cocoa.MissingInvalidationMethod
                                  Check that the invalidation methods are present in classes that contain invalidatable instance variables
  alpha.security.ArrayBound       Warn about buffer overflows (older checker)
  alpha.security.ArrayBoundV2     Warn about buffer overflows (newer checker)
  alpha.security.MallocOverflow   Check for overflows in the arguments to malloc()
  alpha.security.ReturnPtrRange   Check for an out-of-bound pointer being returned to callers
  alpha.security.taint.TaintPropagation
                                  Generate taint information used by other checkers
  alpha.unix.Chroot               Check improper use of chroot
  alpha.unix.MallocWithAnnotations
                                  Check for memory leaks, double free, and use-after-free problems. Traces memory managed by malloc()/free(). Assumes that all user-defined functions which might free a pointer are annotated.
  alpha.unix.PathCondExtract      Extract path conditions for each return code
  alpha.unix.PthreadLock          Simple lock -> unlock checker
  alpha.unix.SimpleStream         Check for misuses of stream APIs
  alpha.unix.Stream               Check stream handling functions
  alpha.unix.cstring.BufferOverlap
                                  Checks for overlap in two buffer arguments
  alpha.unix.cstring.NotNullTerminated
                                  Check for arguments which are not null-terminating strings
  alpha.unix.cstring.OutOfBounds  Check for out-of-bounds access in string functions
  core.CallAndMessage             Check for logical errors for function calls and Objective-C message expressions (e.g., uninitialized arguments, null function pointers)
  core.DivideZero                 Check for division by zero
  core.DynamicTypePropagation     Generate dynamic type information
  core.NonNullParamChecker        Check for null pointers passed as arguments to a function whose arguments are references or marked with the 'nonnull' attribute
  core.NullDereference            Check for dereferences of null pointers
  core.StackAddressEscape         Check that addresses to stack memory do not escape the function
  core.UndefinedBinaryOperatorResult
                                  Check for undefined results of binary operators
  core.VLASize                    Check for declarations of VLA of undefined or zero size
  core.builtin.BuiltinFunctions   Evaluate compiler builtin functions (e.g., alloca())
  core.builtin.NoReturnFunctions  Evaluate "panic" functions that are known to not return to the caller
  core.uninitialized.ArraySubscript
                                  Check for uninitialized values used as array subscripts
  core.uninitialized.Assign       Check for assigning uninitialized values
  core.uninitialized.Branch       Check for uninitialized values used as branch conditions
  core.uninitialized.CapturedBlockVariable
                                  Check for blocks that capture uninitialized values
  core.uninitialized.UndefReturn  Check for uninitialized values being returned to the caller
  cplusplus.NewDelete             Check for double-free and use-after-free problems. Traces memory managed by new/delete.
  cplusplus.NewDeleteLeaks        Check for memory leaks. Traces memory managed by new/delete.
  deadcode.DeadStores             Check for values stored to variables that are never read afterwards
  debug.ConfigDumper              Dump config table
  debug.DumpCFG                   Display Control-Flow Graphs
  debug.DumpCallGraph             Display Call Graph
  debug.DumpCalls                 Print calls as they are traversed by the engine
  debug.DumpDominators            Print the dominance tree for a given CFG
  debug.DumpLiveVars              Print results of live variable analysis
  debug.DumpTraversal             Print branch conditions as they are traversed by the engine
  debug.ExprInspection            Check the analyzer's understanding of expressions
  debug.Stats                     Emit warnings with analyzer statistics
  debug.TaintTest                 Mark tainted symbols as such.
  debug.ViewCFG                   View Control-Flow Graphs using GraphViz
  debug.ViewCallGraph             View Call Graph using GraphViz
  debug.ViewExplodedGraph         View Exploded Graphs using GraphViz
  llvm.Conventions                Check code for LLVM codebase conventions
  osx.API                         Check for proper uses of various Apple APIs
  osx.SecKeychainAPI              Check for proper uses of Secure Keychain APIs
  osx.cocoa.AtSync                Check for nil pointers used as mutexes for @synchronized
  osx.cocoa.ClassRelease          Check for sending 'retain', 'release', or 'autorelease' directly to a Class
  osx.cocoa.IncompatibleMethodTypes
                                  Warn about Objective-C method signatures with type incompatibilities
  osx.cocoa.Loops                 Improved modeling of loops using Cocoa collection types
  osx.cocoa.MissingSuperCall      Warn about Objective-C methods that lack a necessary call to super
  osx.cocoa.NSAutoreleasePool     Warn for suboptimal uses of NSAutoreleasePool in Objective-C GC mode
  osx.cocoa.NSError               Check usage of NSError** parameters
  osx.cocoa.NilArg                Check for prohibited nil arguments to ObjC method calls
  osx.cocoa.NonNilReturnValue     Model the APIs that are guaranteed to return a non-nil value
  osx.cocoa.RetainCount           Check for leaks and improper reference count management
  osx.cocoa.SelfInit              Check that 'self' is properly initialized inside an initializer method
  osx.cocoa.UnusedIvars           Warn about private ivars that are never used
  osx.cocoa.VariadicMethodTypes   Check for passing non-Objective-C types to variadic collection initialization methods that expect only Objective-C types
  osx.coreFoundation.CFError      Check usage of CFErrorRef* parameters
  osx.coreFoundation.CFNumber     Check for proper uses of CFNumberCreate
  osx.coreFoundation.CFRetainRelease
                                  Check for null arguments to CFRetain/CFRelease/CFMakeCollectable
  osx.coreFoundation.containers.OutOfBounds
                                  Checks for index out-of-bounds when using 'CFArray' API
  osx.coreFoundation.containers.PointerSizedValues
                                  Warns if 'CFArray', 'CFDictionary', 'CFSet' are created with non-pointer-size values
  security.FloatLoopCounter       Warn on using a floating point value as a loop counter (CERT: FLP30-C, FLP30-CPP)
  security.insecureAPI.UncheckedReturn
                                  Warn on uses of functions whose return values must be always checked
  security.insecureAPI.getpw      Warn on uses of the 'getpw' function
  security.insecureAPI.gets       Warn on uses of the 'gets' function
  security.insecureAPI.mkstemp    Warn when 'mkstemp' is passed fewer than 6 X's in the format string
  security.insecureAPI.mktemp     Warn on uses of the 'mktemp' function
  security.insecureAPI.rand       Warn on uses of the 'rand', 'random', and related functions
  security.insecureAPI.strcpy     Warn on uses of the 'strcpy' and 'strcat' functions
  security.insecureAPI.vfork      Warn on uses of the 'vfork' function
  unix.API                        Check calls to various UNIX/Posix functions
  unix.Malloc                     Check for memory leaks, double free, and use-after-free problems. Traces memory managed by malloc()/free().
  unix.MallocSizeof               Check for dubious malloc arguments involving sizeof
  unix.MismatchedDeallocator      Check for mismatched deallocators.
  unix.cstring.BadSizeArg         Check the size argument passed into C string functions for common erroneous patterns
  unix.cstring.NullArg            Check for null pointers being passed as arguments to C string functions
```

Too tedious? OK, here is the Chinese Table version.

| 检查器      | 描述                                                                                                                               |
|-----|---------|
| alpha.core.BoolAssignment             | 警告：将非{0,1}值分配给布尔变量                                                                             |
| alpha.core.CallAndMessageUnInitRefArg | 检查函数调用和Objective-C消息表达式的逻辑错误（例如，未初始化的参数、空函数指针和指向未定义变量的指针）          |
| alpha.core.CastSize                   | 当对malloc分配的类型T进行强制转换时，检查大小是否是T大小的倍数                                            |
| alpha.core.CastToStruct               | 检查从非结构指针到结构指针的转换                                                                             |
| alpha.core.FixedAddr                  | 检查将固定地址赋给指针的情况                                                                             |
| alpha.core.IdenticalExpr              | 警告：在运算符中意外使用相同的表达式                                                                         |
| alpha.core.PointerArithm              | 检查除数组元素之外的位置上的指针算术运算                                                                    |
| alpha.core.PointerSub                 | 检查两个指针指向不同内存块的情况下的指针减法                                                              |
| alpha.core.SizeofPtr                  | 警告：在指针表达式上意外使用sizeof()                                                                       |
| alpha.core.TestAfterDivZero           | 检查除以后与0进行比较的变量。要么比较是无用的，要么存在除以零的情况                                        |
| alpha.cplusplus.VirtualCall           | 在构造或析构期间检查虚拟函数调用                                                                            |
| alpha.deadcode.UnreachableCode        | 检查无法访问的代码                                                                                         |
| alpha.osx.cocoa.Dealloc               | 警告：Objective-C类缺乏正确实现-dealloc方法                                                                |
| alpha.osx.cocoa.DirectIvarAssignment   | 检查直接对实例变量进行赋值的情况                                                                          |
| alpha.osx.cocoa.DirectIvarAssignmentForAnnotatedFunctions | 在带有objc_no_direct_instance_variable_assignment注解的方法中检查直接对实例变量进行赋值的情况 |
| alpha.osx.cocoa.InstanceVariableInvalidation | 检查在带有objc_instance_variable_invalidator注解的方法中无效的实例变量                                  |
| alpha.osx.cocoa.MissingInvalidationMethod | 检查包含无效实例变量的类中是否存在无效方法                                                              |
| alpha.security.ArrayBound             | 警告：缓冲区溢出（较旧的检查器）                                                                         |
| alpha.security.ArrayBoundV2           | 警告：缓冲区溢出（较新的检查器）                                                                         |
| alpha.security.MallocOverflow         | 检查malloc()参数的溢出                                                                                   |
| alpha.security.ReturnPtrRange         | 检查将越界指针返回给调用者的情况                                                                         |
| alpha.security.taint.TaintPropagation  | 生成其他检查器使用的污点信息                                                                             |
| alpha.unix.Chroot                     | 检查chroot的不当使用                                                                                    |
| alpha.unix.MallocWithAnnotations       | 检查内存泄漏、double free和use-after-free问题。跟踪malloc()/free()管理的内存。假设可能释放指针的所有用户定义函数都有注释。 |
| alpha.unix.PathCondExtract            | 提取每个返回码的路径条件                                                                               |
| alpha.unix.PthreadLock                | 简单的锁 -> 解锁检查器                                                                                    |
| alpha.unix.SimpleStream               | 检查流API的误用                                                                                        |
| alpha.unix.Stream                     | 检查流处理函数                                                                                        |
| alpha.unix.cstring.BufferOverlap      | 检查两个缓冲区参数的重叠                                                                              |
| alpha.unix.cstring.NotNullTerminated  | 检查不是以null结尾的字符串参数                                                                        |
| alpha.unix.cstring.OutOfBounds        | 检查字符串函数中的越界访问                                                                            |
| core.CallAndMessage                   | 检查函数调用和Objective-C消息表达式的逻辑错误（例如，未初始化的参数、空函数指针）                        |
| core.DivideZero                       | 检查除以零                                                                                            |
| core.DynamicTypePropagation           | 生成动态类型信息                                                                                      |
| core.NonNullParamChecker              | 检查传递给函数的空指针参数，这些参数是引用或带'nonnull'属性的函数                                       |
| core.NullDereference                  | 检查对空指针的解引用                                                                                  |
| core.StackAddressEscape               | 检查栈内存地址不逃逸函数                                                                              |
| core.UndefinedBinaryOperatorResult    | 检查二进制运算符的未定义结果                                                                          |
| core.VLASize                          | 检查对未定义或大小为零的VLA的声明                                                                      |
| core.builtin.BuiltinFunctions         | 评估编译器内建函数（例如，alloca()）                                                                 |
| core.builtin.NoReturnFunctions        | 评估已知不会返回调用者的“panic”函数                                                                    |
| core.uninitialized.ArraySubscript     | 检查未初始化值用作数组下标                                                                            |
| core.uninitialized.Assign             | 检查对未初始化值的赋值                                                                              |
| core.uninitialized.Branch             | 检查未初始化值用作分支条件                                                                          |
| core.uninitialized.CapturedBlockVariable | 检查捕获未初始化值的块                                                                              |
| core.uninitialized.UndefReturn        | 检查未初始化值是否返回给调用者                                                                        |
| cplusplus.NewDelete                   | 检查double-free和use-after-free问题。跟踪new/delete管理的内存。                                       |
| cplusplus.NewDeleteLeaks              | 检查内存泄漏。跟踪new/delete管理的内存。                                                              |
| deadcode.DeadStores                   | 检查将值存储到后续永远不会读取的变量                                                                    |
| debug.ConfigDumper                    | 转储配置表                                                                                            |
| debug.DumpCFG                         | 显示控制流图                                                                
| debug.DumpCallGraph                   | 显示调用图                                                                                               |
| debug.DumpCalls                       | 按引擎遍历调用时打印调用                                                                                 |
| debug.DumpDominators                  | 打印给定CFG的支配树                                                                                      |
| debug.DumpLiveVars                    | 打印活动变量分析的结果                                                                                 |
| debug.DumpTraversal                   | 按引擎遍历时打印分支条件                                                                               |
| debug.ExprInspection                  | 检查分析器对表达式的理解                                                                               |
| debug.Stats                           | 使用分析器统计信息发出警告                                                                             |
| debug.TaintTest                       | 将受污染的符号标记为受污染。                                                                          |
| debug.ViewCFG                         | 使用GraphViz查看控制流图                                                                             |
| debug.ViewCallGraph                   | 使用GraphViz查看调用图                                                                               |
| debug.ViewExplodedGraph               | 使用GraphViz查看爆炸图                                                                              |
| llvm.Conventions                      | 检查LLVM代码库的代码约定                                                                              |
| osx.API                               | 检查对各种Apple API的正确使用                                                                         |
| osx.SecKeychainAPI                    | 检查Secure Keychain API的正确使用                                                                      |
| osx.cocoa.AtSync                      | 检查用作@synchronized互斥锁的nil指针                                                                   |
| osx.cocoa.ClassRelease                | 检查直接向类发送'retain'、'release'或'autorelease'的情况                                              |
| osx.cocoa.IncompatibleMethodTypes     | 警告：Objective-C方法签名存在类型不兼容的情况                                                           |
| osx.cocoa.Loops                       | 使用Cocoa集合类型改进循环建模                                                                          |
| osx.cocoa.MissingSuperCall            | 警告：缺少对super的必要调用的Objective-C方法                                                         |
| osx.cocoa.NSAutoreleasePool           | 在Objective-C GC模式下警告NSAutoreleasePool的次优用法                                                 |
| osx.cocoa.NSError                     | 检查NSError**参数的使用                                                                             |
| osx.cocoa.NilArg                      | 检查禁止向ObjC方法调用传递nil参数的情况                                                              |
| osx.cocoa.NonNilReturnValue           | 对确保返回非nil值的API进行建模                                                                         |
| osx.cocoa.RetainCount                 | 检查泄漏和不正确的引用计数管理                                                                        |
| osx.cocoa.SelfInit                    | 检查在初始化方法中是否正确初始化'self'                                                                  |
| osx.cocoa.UnusedIvars                 | 警告：未使用的私有ivar                                                                               |
| osx.cocoa.VariadicMethodTypes         | 检查将非Objective-C类型传递给期望仅接受Objective-C类型的可变集合初始化方法的情况                         |
| osx.coreFoundation.CFError            | 检查CFErrorRef*参数的使用                                                                            |
| osx.coreFoundation.CFNumber           | 检查CFNumberCreate的正确使用                                                                         |
| osx.coreFoundation.CFRetainRelease    | 检查对CFRetain/CFRelease/CFMakeCollectable的null参数的使用                                              |
| osx.coreFoundation.containers.OutOfBounds | 使用'CFArray' API时检查索引越界                                                                   |
| osx.coreFoundation.containers.PointerSizedValues | 如果使用非指针大小的值创建'CFArray'、'CFDictionary'或'CFSet'，发出警告                            |
| security.FloatLoopCounter             | 警告：使用浮点值作为循环计数器                                                                         |
| security.insecureAPI.UncheckedReturn  | 警告：使用必须始终检查返回值的函数                                                                     |
| security.insecureAPI.getpw            | 警告：使用'getpw'函数                                                                                 |
| security.insecureAPI.gets             | 警告：使用'gets'函数                                                                                 |
| security.insecureAPI.mkstemp          | 警告：当'mkstemp'的格式字符串中包含少于6个X时                                                          |
| security.insecureAPI.mktemp           | 警告：使用'mktemp'函数                                                                              |
| security.insecureAPI.rand             | 警告：使用'rand'、'random'及相关函数                                                                  |
| security.insecureAPI.strcpy           | 警告：使用'strcpy'和'strcat'函数                                                                    |
| security.insecureAPI.vfork            | 警告：使用'vfork'函数                                                                              |
| unix.API                              | 检查对各种UNIX/Posix函数的调用                                                                        |
| unix.Malloc                           | 检查内存泄漏、double free和use-after-free问题。跟踪malloc()/free()管理的内存。假设可能释放指针的所有用户定义函数都有注释。 |
| unix.MallocSizeof                     | 检查涉及sizeof的可疑malloc参数                                                                       |
| unix.MismatchedDeallocator            | 检查不匹配的deallocators                                                                             |
| unix.cstring.BadSizeArg               | 检查传递给C字符串函数的大小参数的常见错误模式                                                         |
| unix.cstring.NullArg                  | 检查将null指针作为参数传递给C字符串函数的情况                                                        |

检查器的命名规范遵循 `<package>.<subpackage>.<checker>`。

![package 命名与功能](/Clang阅读随笔/image2.png)

## 直接调用 Clang 命令运行 Checker 的方式
### 通过 Driver 运行
#### 启用指定包的所有检查器
```shell
clang --analyze -Xanalyzer -analyzer-checker=<package> <source-files>
```
- `--analyze`选项，表示运行 Clang 静态分析器。
- `-Xanalyzer -analyzer-checker=<package>`选项，表示将`-analyzer-checker=<package>`参数传递给 Clang 静态分析器。
- `-analyzer-checker=<package>`选项，表示启用指定包中的所有检查器。如果要同时指定多个包的所有检查器，则形如：`clang --analyze -Xanalyzer -analyzer-checker=core -Xanalyzer -analyzer-checker=cplusplus hello.cpp`。
- `source-files`选项，表示要分析的源程序文件列表，源程序文件之间以空格分隔。

#### 启用指定的检查器
```shell
clang --analyze -Xanalyzer -analyzer-checker=<package.subpackage.checker> <source-files>
```
如果要同时指定多个检查器，则形如：`clang --analyze -Xanalyzer -analyzer-checker=core.DivideZero -Xanalyzer -analyzer-checker=cplusplus.NewDelete hello.cpp`

需要注意的是， 通过编译器驱动程序运行 Clang 静态分析器的方式，会根据系统环境默认提供一些检查器。比如，未显式指定任何检查器，但编译器驱动程序默认提供的检查器中包括core、unix、deadcode和cplusplus等的所有检查器，仍可以检测出“除 0”错误。可以通过`-###`打印详细参数。

需要注意的是，Clang 静态分析器可能不会一次性输出所有错误。这提示我们在修复警告后最好再次运行 Clang 静态分析器。

#### 禁用指定的检查器
```shell
clang --analyze -Xanalyzer -analyzer-disable-checker=<arg> <source-files>
```


### 通过 -cc1 运行
如果要真正实现只运行我们指定的检查器，则应该通过 `clang -cc1` 的方式直接进入前端运行 Clang 静态分析器。

#### 启用指定包的所有检查器
```shell
clang -cc1 -w -fcolor-diagnostics -analyze -analyzer-checker=<package> <source-files> -I<directory>
```
- `-w`选项，表示禁用编译器的所有诊断，即禁用类似于`-Wdivision-by-zero`的诊断。从而，只启用 Clang 静态分析器的分析功能以避免重复警告。
- `-fcolor-diagnostics`选项，表示输出带颜色的诊断信息。
- `-analyze`选项，表示运行 Clang 静态分析器。
- `-I`选项，表示为需要的头文件指定其所在的目录。

#### 启用指定的检查器
```shell
clang -cc1 -w -fcolor-diagnostics -analyze -analyzer-checker=<package.subpackage.checker> <source-files>
```

### 导出静态分析结果

我们可以通过如下命令查看 Clang 静态分析器提供的所有命令。

```shell
clang cc1 -help | grep analyzer
```

其中，与`--analyzer-output`标志相关的内容如下：

```
    --analyzer-output <value>   
                          Static analyzer report output format (html|plist|plist-multi-file|plist-html|sarif|text).
```

我们可以通过`--analyzer-output`标志指定静态分析结果的文件格式。命令如下：

```shell
clang -cc1 -w -fcolor-diagnostics -analyze -analyzer-checker=<package.subpackage.checker> <source-files> -analyzer-output <format> -o <output-file>
```

## 通过 scan-build 运行 Checker 的方式

要使用 scan-build，只要在构建命令的最前面添加 `scan-build` 即可，如下所示：

```shell
scan-build [scan-build options] <command> [command options]
```
常用的 scan-build options 如下表所示。

|scan-build options	|用途|
|-------|---------------------|
|-o <output location>	|指定分析结果的存储位置|
|-v	|输出分析过程，可以连续使用两个或三个 -v 以提高输出信息的详细度。|
|-V 或 --view	|在生成错误报告完成后，立即在浏览器中查看（即自动调用scan-view命令）|

要查看所有的 scan-build options，可以通过 `scan-build --help` 命令。


