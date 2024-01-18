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

如果一个类定义在header文件中，并且有虚函数或者派生自有虚函数的类，那么它就会有一个vtable。

对于有vtable的类，编译器需要为每个包含该头文件的编译单元生成vtable的副本。如果类中所有的虚函数都是内联的(inline)，那么vtable就会非常小，只包含几个函数指针。编译器为了优化，会选择把这种小的vtable和RTTI信息内联到每个包含头文件的.o文件中。

但是如果类中有至少一个实体虚函数，那么vtable大小会非常大，包含这个虚函数的指针和其他从基类继承的虚函数指针。这时编译器就不会内联整个大的vtable了，只会在每个.o文件中放一个指向这个vtable的指针，真正的vtable只会存在于定义该类的那个编译单元中。这样可以避免vtable被复制多次浪费空间，也加快了链接时间，因为不需要合并重复的符号了。

为了避免这个问题，有以下建议:

- 如果一个头文件类有vtable，那么在类中至少定义一个非内联的虚函数。
- 将vtable相关的类定义在cpp文件中，而不是头文件中。
- 将有vtable的类的定义分割成头文件中的声明和cpp文件中的定义。
