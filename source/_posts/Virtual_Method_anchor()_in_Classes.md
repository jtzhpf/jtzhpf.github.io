---
title: "C++ 中的优化技巧——虚函数anchor()"
category: CS&Maths
#id: 57
date: 2024-1-18 09:00:00
tags: 
  - LLVM
  - Clang
  - C++
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---

在Clang中，很多类都有这么一个虚函数：
```c++
  virtual void anchor();
```

查阅[LLVM Coding Standards](https://llvm.org/docs/CodingStandards.html#provide-a-virtual-method-anchor-for-classes-in-headers)，得到如下解释：

> If a class is defined in a header file and has a vtable (either it has virtual methods or it derives from classes with virtual methods), it must always have at least one out-of-line virtual method in the class. Without this, the compiler will copy the vtable and RTTI into every .o file that #includes the header, bloating .o file sizes and increasing link times.

在 Clang 和 LLVM 的代码中，有时会看到一个空的虚函数 `virtual void anchor();`，其存在的主要目的是优化编译器生成的代码，具体来说是避免多个 `.o` 文件中重复生成虚表（vtable）和运行时类型信息（RTTI）。这是一种遵循 LLVM 编码标准的优化策略。

根据 LLVM 编码标准中的解释，定义在头文件中的类，如果包含虚函数或继承自具有虚函数的类，那么该类必须有至少一个 **out-of-line**（非内联）的虚函数实现。原因是：

1. **虚表（vtable）和 RTTI 的重复**：如果一个类的所有虚函数都在头文件中定义，编译器会在每个包含该头文件的 `.o` 文件中都生成一份虚表和 RTTI。这会导致：
   - `.o` 文件的大小变大。
   - 链接时间增加，因为每个 `.o` 文件都有相同的虚表和 RTTI，链接器需要处理这些重复的内容。

2. **避免膨胀**：通过提供一个 out-of-line 的虚函数（即该函数的实现放在 `.cpp` 文件中，而非直接在头文件中定义），编译器能够确保虚表和 RTTI 只在一个地方生成，从而避免重复和膨胀。

在很多情况下，这个 out-of-line 的虚函数会是一个空实现，如 `virtual void anchor();`，它的唯一目的就是提供这个 out-of-line 定义。这样做的好处是简化了类的定义，同时达到了优化编译结果的目的。

为了避免这个问题，有以下建议:

- 如果一个头文件类有vtable，那么在类中至少定义一个非内联的虚函数。
- 将vtable相关的类定义在cpp文件中，而不是头文件中。
- 将有vtable的类的定义分割成头文件中的声明和cpp文件中的定义。
