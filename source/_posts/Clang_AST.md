---
title: "Clang AST"
category: CS&Maths
#id: 57
date: 2023-10-17 09:00:00
tags: 
  - LLVM
  - Clang
  - Compiler
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: code  # 展示在时间线列表中
---


<!--more-->
[Clang AST官网简介](https://clang.llvm.org/docs/IntroductionToTheClangAST.html)

查看AST的命令
```shell
clang -Xclang -ast-dump -fsyntax-only filename
```

现在有一个cpp文件`source.cpp`如下所示：
```c++
//file name: source.cpp
int foo(double x, double y)
{
  double result = (x / 777);
  
  int ans;
  
  if( result > 1 )
  {
    ans = 111;
  }
  else
  {
    ans = 222;
  }
  
  return ans;
}
```
查看AST得到：

![](/Clang_AST/image1.png)

## TranslationUnitDecl
**TranslationUnitDecl**表示**翻译单元声明**，这是AST的根节点，表示整个翻译单元，通常对应于一个源代码文件。`<invalid sloc>`表示无有效的源代码位置信息（此处不需要）。

## TypedefDecl
**TypedefDecl**表示**类型定义声明**，这些节点表示了类型别名的声明，通常用于为某种类型分配一个更易于使用的名称。`implicit`表示这些类型别名是隐式的，即它们不是由程序员显式定义的，而是由编译器自动生成的。

在这里，**TypedefDecl**有三个子节点，分别表示以下三个类型别名的声明：

- `__int128_t`：这个别名是对名为 `__int128` 的类型的声明。
- `__uint128_t`：这个别名是对名为 `unsigned __int128` 的类型的声明。
- `__builtin_va_list`：这个别名是对名为 `__va_list_tag [1]` 的类型的声明。

## FunctionDecl
**FunctionDecl**表示**函数声明**，也是每个函数的根节点。该函数只有2个子节点，分别是`ParmVarDecl`与`CompoundStmt`，前者代表参数，后者代表函数体。

`0x2bc8dd0` 是该节点的唯一标识符（通常是编译器生成的内部标识符）。

`<source.cpp:2:1, line:18:1>` 表示该函数声明出现在源代码文件`source.cpp`中，从第2行的第1列到第18行的第1列。

`line:2:5` 指示该函数声明在源代码中的位置，具体地说，是在第2行第5列。

`foo` 是函数的名称。

`'int (double, double)'` 表示该函数的返回类型为 `int`，并且有两个参数，两个参数的类型都是 `double`。

## ParmVarDecl
**ParmVarDecl**表示**参数变量声明**，即函数的输入参数。

`<col:9, col:16>` 和 `<col:19, col:26>` 分别表示它们在源代码中的位置范围，具体地说，第一个参数`x`在第9列到第16列之间，第二个参数`y`在第19列到第26列之间。

`col:16`和`col:26`表示它们的声明位置。

`used`表示这些参数在函数中被使用。

## CompoundStmt
**CompoundStmt**表示**复合语句条目**，表示一段由大括号围起来的代码段。

`<line:3:1, line:18:1>`表示该复合语句整体的起始位置与末位置。起始位置是3行1列，末尾位置是18行1列。

## DeclStmt
**DeclStmt**表示**声明语句**。
这是函数 `foo` 内部的一部分抽象语法树（AST），表示一个变量的声明和初始化。

- `VarDecl`（变量声明）:
   - 这个节点表示一个变量的声明。
   - `result` 是变量的名称。
   - `'double'` 表示该变量的类型为 `double`。
   - `cinit` 表示这是一个带有初始值的变量声明。

- `ParenExpr`（括号表达式）:
   - 这个节点表示一个带有括号的表达式，通常用于明确运算符优先级。

- `BinaryOperator`（二元运算符）:
   - 这个节点表示一个二元运算，用于执行除法运算。
   
- `ImplicitCastExpr`（隐式类型转换表达式）:
   - 这个节点表示隐式的类型转换，用于将表达式的类型转换为所需的类型。
   - 此处将左值 `result`（从 `0x38bae90` 节点引用）转换为右值。

- `DeclRefExpr`（声明引用表达式）:
   - `ParmVar 0x38bac90 'x' 'double'` 表示它引用了一个名为 `x` 的 `double` 类型的参数。

- `IntegerLiteral`（整数字面量）:
   - 这个节点表示一个整数字面量。
   - `'int'` 表示这个字面量的类型为 `int`。
   - `777` 是这个整数字面量的值。

综合起来，这部分AST表示了一个名为 `result` 的 `double` 类型的变量声明，它的初始值是 `x` 除以整数字面量 `777` 的结果。

`LValueToRValue` 是一个 C/C++ 中的隐式类型转换，它将左值（Lvalue）转换为右值（Rvalue）。这种转换的存在是为了确保程序在合适的上下文中使用变量，同时也是为了维护类型的一致性。以下是一些情况，你可能需要进行 `LValueToRValue` 的转换：

1. **赋值操作**：当你将一个左值赋值给一个右值时，编译器需要进行 `LValueToRValue` 的转换。这是因为左值表示内存位置，而右值表示值，赋值操作需要获取左值的值以进行赋值。

   ```c++
   int x = 5; // 5 是右值，但 x 是左值，所以需要 LValueToRValue 转换
   ```

2. **函数调用**：当你将一个左值作为参数传递给函数时，也会发生 `LValueToRValue` 转换。函数参数通常期望右值，因此左值会在函数调用时被转换为右值。

   ```c++
   int value = 10;
   someFunction(value); // value 是左值，但 someFunction 期望右值参数
   ```

3. **表达式计算**：在某些表达式中，需要将左值的值提取出来以进行计算。这也会触发 `LValueToRValue` 转换。

   ```c++
   int a = 5;
   int b = a + 3; // a 是左值，但在表达式中需要它的值，因此发生 LValueToRValue 转换
   ```

## IfStmt

**IfStmt**表示**条件语句**，此处包含一个条件表达式和两个条件成立时执行的代码块。

- `IfStmt`（条件语句）：
  - `BinaryOperator` 表示条件表达式，它计算一个条件是否为真。
  - `CompoundStmt` 包含了两个代码块，分别表示条件成立和条件不成立时执行的代码。

- `BinaryOperator`（二元运算符）：
  - `'_Bool'` 表示该表达式的结果类型为布尔值。
  - `'>'` 是条件判断符号，表示大于。
  - `ImplicitCastExpr` 是一个隐式类型转换表达式，它将左值 `result`（从 `0x38bae90` 节点引用）转换为右值。
  - `IntegerLiteral` 表示整数字面量 "1"。

- `CompoundStmt`（代码块）：
  - `0x38fbf50` 和 `0x38fbfe0` 分别表示两个代码块的唯一标识符。
  - `<line:9:3, line:11:3>` 和 `<line:13:3, line:15:3>` 表示这两个代码块的位置范围。
  
  - 第一个 `CompoundStmt` 包含一个 `BinaryOperator`，表示条件成立时执行的代码。
  - `BinaryOperator` 此时表示赋值操作，将变量 `ans` 的值设置为整数 `111`。
  
  - 第二个 `CompoundStmt` 包含一个 `BinaryOperator`，表示条件不成立时执行的代码。
  - `BinaryOperator` 此时表示另一个赋值操作，将变量 `ans` 的值设置为整数 `222`。

## ReturnStmt
**ReturnStmt**表示**返回语句**。

- `ImplicitCastExpr`（隐式类型转换表达式）：
  - `0x38fc058` 是隐式类型转换表达式的唯一标识符。
  - `<col:10>` 表示这个表达式的位置，即在第10列。
  - `int` 表示这个表达式的结果类型为整数。
  - `<LValueToRValue>` 表示这是一个从左值到右值的隐式类型转换。
  
- `DeclRefExpr`（声明引用表达式）：
  - `0x38fc030` 是声明引用表达式的唯一标识符。
  - `<col:10>` 表示这个表达式的位置，即在第10列。
  - `'int'` 表示这个引用的类型为整数。
  - `lvalue` 表示这是一个左值。
  - `Var 0x38fbdd0 'ans' 'int'` 表示这个引用指向名为 `ans` 的整数变量。