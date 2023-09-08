---
title: "《Cross-checking Semantic Correctness: The Case of Finding File System Bugs》文章精读"
category: CS&Maths
#id: 57
date: 2023-8-31 20:42:32
tags: 
  - Linux
  - File System
  - Staitc Analysis
  - SOSP
  - 论文
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---

主要思路是统计多个文件系统的实现，计算具体文件系统与多个文件系统总体之间的差异性（直方图/信息熵），从而窥探出其文件系统的具体实现在语义上的差异。
<!--more-->

## 设计
![Overview](/《Cross-checking Semantic Correctness: The Case of Finding File System Bugs》文章精读/image1.png)

1. To enable comparison, the source code of each file system is merged into one large file. This allows interprocedural analysis within a file system.
2. JUXTA collects execution information for each function in the file systems by constructing control-flow graphs (CFGs) and symbolically exploring these graphs from the entry point to the end of each function.
3. Symbols are canonicalized so that equivalent symbols have the same name across file systems. A path database that stores the extracted path information is also created, which allows applications like checkers and specification extractors to use the path information without reexecuting symbolic analysis. All of these facilitate comparison.
4. Symbolic execution is used to explore paths through functions and build a path database. The path info contains path conditions, return values, side effects, etc.
5. Two statistical methods are used to compare paths:
   - Histogram-based: Integer ranges are encoded into histograms and distances between histograms are computed.
   - Entropy-based: Used for **events** like flags or return value checks. Low non-zero entropy indicates a likely bug.（别的的实现均不包含这一event，则说明this event could be wrong implementation）
  
  ![Histogram-based comparison](/《Cross-checking Semantic Correctness: The Case of Finding File System Bugs》文章精读/image2.png)
  
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

## 实验复现
### 实验环境
- docker ubuntu 14.04
- gcc 4.8
- g++ 4.8

### 复现步骤

#### 下载 juxta
```shell
git clone https://github.com/sslab-gatech/juxta.git
```

#### 下载并测试 Linux
```shell
git clone https://github.com/torvalds/linux.git
cd linux
git checkout v4.0-rc2
cp ../juxta/config/config-x86_64-full-fs-4.0 .config
make; make clean
cd ../juxta
```

#### 编译 clang
在编译 clang 之前在 juxta 的 `Makefile` 中增加`COMPILER := -DCMAKE_CXX_COMPILER=g++-4.8 -DCMAKE_C_COMPILER=gcc-4.8`，并且将其添加到 cmake 的参数中。
```shell
make clang-full   (first time only)
make clang        (from the next)
```

#### 构建 path database
合并文件系统代码
```shell
cd analyzer
./ctrl.py merge_all  (for all file systems)
./ctrl.py merge ext4 (for ext4)
```

对合并的文件系统代码静态分析
```shell
./ctrl.py clang_all  (for all file systems)
./ctrl.py clang ext4 (for ext4)
```

构建路径数据库
```shell
./ctrl.py pickle_all (for all file systems)
```

#### code checker
```shell
./ckrtn.py
```

#### get sorted results of checker output
```shell
./catcklog.sh [checker output dir]
```

### 结果分析
TODO

### 改进
TODO