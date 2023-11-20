---
title: "All File Systems Are Not Created Equal: On the Complexity of Crafting Crash-Consistent Applications 阅读"
category: CS&Maths
#id: 57
date: 2023-9-6 09:00:00
tags: 
  - Linux
  - File System
  - Dynamic Analysis
  - OSDI
  - OSDI'14
  - Paper
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---

本文介绍了基于现代文件系统的应用层崩溃一致性协议的首个全面研究。研究发现，应用程序使用复杂的更新协议来持久化状态，而这些协议的正确性高度依赖于底层文件系统的微妙行为，我们称之为持久性属性。作者开发了一个名为BOB的工具来测试持久性属性，并使用它来证明这些属性在六种流行的Linux文件系统中存在广泛的差异。作者还构建了一个名为ALICE的框架，分析应用程序更新协议并发现崩溃漏洞，即需要特定持久性属性才能保证正确性的更新协议代码。使用ALICE，作者分析了11个广泛使用的系统，并发现了60个漏洞，其中许多会导致严重后果。作者还展示了ALICE可以用于评估新文件系统设计对应用程序级一致性的影响。
<!--more-->

研究发现，应用程序的一致性高度依赖于底层文件系统的持久性属性。如果应用程序的正确性依赖于特定的文件系统持久性属性，就会存在崩溃漏洞；在不同的文件系统上运行应用程序可能导致不正确的行为。

![Crash States](/All_File_Systems_Are_Not_Created_Equal:On_the_Complexity_of_Crafting_Crash-Consistent_Applications阅读/image1.png)

## BOB 
BOB是一个简单的工具，用于收集文件系统下的块级追踪，并重新排序以探索可能出现的磁盘崩溃状态，用于测试文件系统的持久性属性。它通过运行一个简单的工作负载来测试持久性属性，收集块I/O并重新排序，生成可能的磁盘状态。然后运行文件系统恢复并检查持久性属性是否成立。如果BOB找到一个检查失败的磁盘映像，则知道该属性在文件系统上不成立。

![Persistence Properties](/All_File_Systems_Are_Not_Created_Equal:On_the_Complexity_of_Crafting_Crash-Consistent_Applications阅读/image3.png)

## ALICE
本文研究现代应用程序的崩溃一致性协议实现是否正确。为了回答这个问题，研究者们开发了一个名为ALICE的框架，可以系统地研究应用程序级别的崩溃一致性。

ALICE将应用工作负载中的系统调用跟踪转换为逻辑操作。逻辑操作将当前读写偏移量、文件描述符等细节抽象化，并将一组大量的系统调用和其他产生I/O行为转换为一小组文件系统操作。逻辑操作还将每个涉及的文件或目录关联到一个概念上的inode。

APM（Abstract Persistence Models）规定了文件系统中逻辑操作的原子性和顺序的所有限制，从而定义了可能的崩溃状态。

APM将崩溃状态表示为两个逻辑实体：包含数据和文件大小的file inode + 包含目录项的directory。

APM将逻辑操作分解为微操作，以捕捉中间崩溃状态。微操作是对每个逻辑实体执行的最小原子修改。共有五个微操作：write_block, change_file_size, create_dir_entry, delete_dir_entry。APM通过定义将逻辑操作转化为微操作来指定原子性约束。APM通过定义哪些微操作可以在其他微操作之前到达磁盘来指定排序约束。

![Default APM Constraints](/All_File_Systems_Are_Not_Created_Equal:On_the_Complexity_of_Crafting_Crash-Consistent_Applications阅读/image2.png)

ALICE将系统调用跟踪转换为微操作，并计算它们之间的顺序依赖关系。ALICE通过将所选集合中的微操作顺序应用于初始状态（表示为逻辑实体），构造新的崩溃状态。对于每个崩溃状态，ALICE将逻辑实体转换回实际文件，并将其提供给检查器进行验证。用户提供的检查器验证崩溃状态。

![Overview of ALICE](/All_File_Systems_Are_Not_Created_Equal:On_the_Complexity_of_Crafting_Crash-Consistent_Applications阅读/image4.png)

### Finding Application Requirements

#### 测试跨系统调用的原子性
它通过构建每个前缀（即前X个系统调用，1<X<N）应用的崩溃状态来实现。如果在这些崩溃状态中发现应用程序不变量被违反，则表示开始了一个原子组。(apply prefix of operations)

#### 测试单个系统调用的原子性
通过将所有先前的系统调用应用于崩溃状态，并生成对应于系统调用不同中间状态的崩溃状态，检查是否违反应用程序不变量。 (apply prefix + partial operation)

#### 测试系统调用间的顺序依赖性
协议要求系统调用A必须在B之前进行持久化时，可以使用ALICE来测试这个属性。

ALICE从开始应用每个系统调用，直到达到B，但不包括A。这样可以检查A是否必须在B之前持久化。
如果B已应用但A没有，并且在这种情况下应用程序不变性被违反，表示存在A必须在B之前进行持久化的顺序依赖性。