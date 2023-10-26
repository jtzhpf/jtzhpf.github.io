---
title: "Push-Button Verification of File Systems via Crash Refinement 阅读"
category: CS&Maths
#id: 57
date: 2023-10-23 09:00:00
tags: 
  - xv6
  - Symbolic Execution
  - OSDI
  - OSDI'16
  - 论文
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
mathjax: true
---

本文提出了 Yggdrasil，Yggdrasil 是一个工具包，可用于编写具有一键验证功能的文件系统。它通过一种称为崩溃细化的新颖正确性定义实现自动化验证，无需手动注释或证明实现代码，并在出现错误时生成反例。崩溃细化适用于完全自动化的 SMT 推理，并使开发人员能够模块化地实现文件系统以进行验证。
![Yggdrasil的开发流程](/Push-Button Verification of File Systems via Crash Refinement 阅读/image1.png){width=500}
<!--more-->

## Yggdrasil
程序员需要编写 specification, implementation 和 consistency invariant (**all in python language**)。

验证器会生成反例来可视化 implementation 或 consistency invariant 中的 bug。

Yggdrasil 可以进行优化并重新验证代码以提高运行时性能。

验证通过后，Yggdrasil 会生成C代码，通过C编译器编译和链接生成可执行文件系统和[fsck检查器](https://en.wikipedia.org/wiki/Fsck)。

### specification
Yggdrasil文件系统规范包括三个部分：

- 表示逻辑布局的抽象数据结构（Abstract data structure）
- 定义预期行为的操作集合
- 定义实现是否符合规范的等价谓词（equivalence predicate）

下面均以作者实现的一个小文件系统YminLFS做演示。

#### Abstract data structure
![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image2.png){width=500}

抽象数据结构由五个抽象映射描述，包括childmap、parentmap和三个存储inode元数据的映射。其中，childmap将目录inode号和名称映射到子inode号，parentmap将inode号映射回其父目录的inode号。InoT和U64T是64位整数类型，NameT是字符串类型。

`FSSpec`数据结构对YminLFS的逻辑布局没有强制约束。`FSSpec`规范通过`invariant`来防止无效的布局。
![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image3.png){width=500}

`invariant`指出父子节点之间有效索引节点号的映射的相互一致。`ForAll`和`Implies`是内置的逻辑运算符。

#### 文件系统操作
文件系统操作包括只读操作和读写操作。只读操作包括`lookup`和`stat`。
![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image4.png){width=500}

修改文件系统的操作更复杂，因为它们涉及更新抽象映射的状态。
![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image5.png){width=500}

`InoT()`构造函数返回一个抽象的inode号码，必须是有效的且不在任何目录中。对文件系统的更改被包装在一个事务中，以确保它们是原子的。

#### state equivalence predicate
![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image6.png){width=500}


### implementation
在Yggdrasil中实现文件系统需要

- 选择磁盘模型
- 编写每个操作的代码
- 编写磁盘布局的consistency invariants 

#### 磁盘模型
Yggdrasil提供了异步磁盘模型和同步磁盘模型。

异步磁盘模型具有无限制的易失性缓存（volatile cache），并允许任意重排序。其接口包括以下操作：

- `d.write (a, v)`: write a data block `v` to disk address `a`; 
- `d.read (a)`: return a data block at disk address `a`; 
- `d.flush ()`: flush the disk cache. 

![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image7.png){width=500}

初始化磁盘创建一个空的根目录文件系统，包括超级块、根目录索引节点和索引节点映射块。初始化后，索引节点映射块只有一个条目，根目录索引节点没有指向数据块。超级块指向索引节点映射块，并存储下一个可用的索引节点和块号。

`mknod`命令将磁盘状态从`Figure 2a`变为`Figure 2b`，以将文件添加到根目录。

1. 为新文件添加一个索引节点块 $I_2$；
2. 为根目录添加一个数据块 $D$，现在根目录有一个条目，该条目将新文件的名称映射到它的索引节点号2；
3. 为更新后的根目录添加一个索引节点块 $I_1^{\prime}$，它指向它的数据块 $D$；
4. 添加一个索引节点映射块 $M^{\prime}$，其中包含两个条目：$1 \mapsto b_5$ 和 $2 \mapsto b_3$；
5. 最后，更新超级块 $SB$，使其指向最新的索引节点映射 $M^{\prime}$。

为了保证系统崩溃时数据的完整性，`mknod`必须发出`flush`操作。如果在最后两次写操作之间没有`flush`操作，磁盘可能会重新排序这些写操作。如果系统在重新排序的写操作之间崩溃，超级块将指向$b_6$中的垃圾数据，导致YminLFS状态损坏。在每次写操作之后插入了`flush`操作，共5个可解决这个问题。

#### Consistency invariants
Consistency invariants是文件系统实现的一种约束条件，用于确定磁盘状态是否对应于有效的文件系统映像。Yggdrasil使用Consistency invariants进行验证和运行时检查，以确保文件系统的正确性。验证时，Yggdrasil检查初始文件系统状态是否满足Consistency invariants，并将其作为每个操作的前置条件和后置条件进行检查。此外，Yggdrasil还可以生成类似于fsck的检查器，用于检查文件系统的完整性。

Consistency invariants约束了磁盘布局的三个组件：超级块$SB$、索引节点映射块 $M$ 和根目录数据块$D$。超级块约束要求下一个可用的索引节点号码$i$大于1，下一个可用的块号码$b$大于$2$，指向$M$的指针既为正数又小于$b$。索引节点映射约束确保$M$将范围$(0, i)$内的每个索引节点号码映射到范围$(0, b)$内的块号码。最后，根目录约束要求$D$将文件名映射到范围$(0,i)$内的索引节点号码。

## Crash Refinement

基于传统的状态机的证明强调步步互锁，要求 implementation 和 specification 始终保持一致，这个要求 is too strong to satisfy，因为文件系统在实际的读写时可能会出现乱序的情况（类似的CPU的指令重拍），此外还有崩溃、恢复等情况（~~**这俩和too strong有啥关系？**~~）。

本文提出的Crash Refinement放松了要求，**any** implementation 的状态只需要和 **some** specification 的状态一致即可。

![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image8.png){width=500}
![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image9.png){width=500}
![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image10.png){width=500}
![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image11.png){width=500}
**Crash-free equivalence**：如果在没有崩溃的情况下，implementation 和 specification 从等价一致状态开始，执行相同操作后能够到达等价一致状态，则 implementation 和 specification 在无崩溃的情况下等价。

![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image12.png){width=500}
**Crash refinement without recovery**：对**任意**的 crash schedule，implementation 在该 crash schedule 下执行后得到的任意状态（不包括恢复），specification 在**某个** crash schedule 下也能得到等价状态，则 implementation 是 specification 的 crash refinement。

![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image13.png){width=500}
![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image14.png){width=500}
**Recovery idempotence**：恢复函数是幂等的，多次崩溃和重启过程中调用多次该函数，结果状态相同。

![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image15.png){width=500}
**Crash refinement with recovery**：对**任意**的 crash schedule，implementation 在该 crash schedule 下执行后得到的任意状态，包括多次执行恢复函数后的任意状态，specification 在**某个** crash schedule 下也能得到等价状态，则 implementation 是 specification 的 crash refinement。

![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image16.png){width=500}
**No-op**: 不改变对外可见状态的后台操作。

![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image17.png){width=500}

## Yxv6 file system
![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image18.png){width=500}


![](/Push-Button Verification of File Systems via Crash Refinement 阅读/image19.png){width=500}
