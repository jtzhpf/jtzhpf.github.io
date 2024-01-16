---
title: "扩展 Static Analyzer"
category: CS&Maths
#id: 57
date: 2024-1-16 09:00:00
tags: 
  - LLVM
  - Clang
  - Compiler
  - Static Analysis
  - Symbolic Execution
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---

Static Analyzer中三个至关重要的数据结构是ProgramState、ProgramPoint和ExplodedGraph。

考虑此例：
```c
#include <stdio.h>
void my_function(int unknownvalue) {
  int schroedinger_integer;
  if (unknownvalue)
    schroedinger_integer = 5;
    printf("hi");
  if (!unknownvalue)
    printf("%d", schroedinger_integer);
}
```
`ProgramState`表示相对于当前状态的当前执行上下文。例如，在分析如上的代码时，它会注明某个变量具有值5。

`ProgramPoint`表示程序流中的特定点，可以是语句之前或之后，例如，将整数变量赋值为5之后的点。

`ExplodedGraph`表示可达程序状态的整个图。`ExplodedGraph`，或者说可达状态图，是对经典控制流图（CFG）的扩展。`ExplodedNode`继承自LLVM库的superclass `llvm::FoldingSetNode`。折叠在编译器的中间和后端广泛使用，LLVM库已经包括了一个用于这些情况的通用类。此图的节点由`ProgramState`和`ProgramPoint`的元组表示，这意味着每个程序点都与特定状态相关联。例如，将整数变量赋值为5之后的点具有将该变量链接到数字5的状态。


静态分析器的整体设计可以分为以下几个部分：

- engine：按照模拟路径并管理其他组件；
- state manager：负责处理ProgramState对象；
- constraint manager：用于推断由于按照给定程序路径进行的ProgramState引起的约束；
- store manager：负责程序存储模型。


# 特殊需求
假设现在有如下程序：
```c
#include <stdio.h>
int Off();
int On();
void test_loop(int wrongTemperature, int restart) {
    On();
    if (wrongTemperature) {
        Off();
    }
    if (restart) {
        Off();
    }
    On();
    //
    // code to keep working
    //
    Off();
}
```
要求`On()`和`Off()`不得连续执行2次。\
如果`wrongTemperature`和`restart`同时不为0，则连续调用`Off()`2次，违反规则。\
如果`wrongTemperature`和`restart`同时为0，则连续调用`On()`2次，也违反规则。


## ProgramState 不可变
`ProgramState`被设计成不可变的。一旦构建完成，它就不应该再发生改变。这是因为`ProgramState`表示为给定执行路径中的特定程序点计算的状态。不同于处理控制流图（CFG）的数据流分析，这里我们处理的是可达程序状态图，该图为每个不同的程序点和状态组合都有一个不同的节点。

当程序中存在循环时，引擎将创建一个全新的路径，记录有关这个新迭代的相关信息。这种方法确保了在不同迭代中能够保留先前计算的状态信息。

然而，在处理循环时，引擎也考虑了性能和避免重复计算的问题。如果引擎达到一个节点，该节点表示具有相同状态的给定循环体的相同程序点，引擎会断定在这条路径上没有新信息需要处理，并选择重用该节点而不是创建新的节点。这样做的目的是避免不必要的计算和提高性能。

另一方面，如果循环体不断使用新信息更新状态，符号引擎可能会面临限制。在模拟预定义次数的迭代后，符号引擎可能会放弃此路径，这个预定义次数在启动工具时是可配置的。这是为了防止无限循环或长时间运行的情况，同时也是一种权衡性能和分析深度的策略。

如上所述，我们的`OnOffState`中不必进行状态的修改。

```cpp
#include "ClangSACheckers.h"
#include "clang/StaticAnalyzer/Core/BugReporter/BugType.h"
#include "clang/StaticAnalyzer/Core/Checker.h"
#include "clang/StaticAnalyzer/Core/CheckerManager.h"
#include "clang/StaticAnalyzer/Core/PathSensitive/CallEvent.h"
#include "clang/StaticAnalyzer/Core/PathSensitive/CheckerContext.h"

using namespace clang;
using namespace ento;

class OnOffState {
private:
  enum Kind { On, Off } K;

public:
  OnOffState(unsigned InK) : K((Kind)InK) {}
  bool isOn() const { return K == On; }
  bool isOff() const { return K == Off; }
  static unsigned getOn() { return (unsigned)On; }
  static unsigned getOff() { return (unsigned)Off; }
  bool operator==(const OnOffState &X) const { return K == X.K; }
  void Profile(llvm::FoldingSetNodeID &ID) const { ID.AddInteger(K); }
};
```
`Profile`函数的存在是为了满足`ExplodedNode`作为`FoldingSetNode`子类的要求。所有的子类都必须提供这样的方法，以帮助LLVM的折叠机制追踪节点的状态，并确定两个节点是否相等（在这种情况下，它们会被折叠）。因此，`Profile`函数用`K`给出了我们的状态。

对于`FoldingSetNode`，可以使用任何以`Add`开头的成员函数来通知唯一的位，以标识此对象实例（参见`llvm/ADT/FoldingSet.h`）。

简而言之，`Profile`函数用于为`ExplodedNode`对象实例创建一个唯一的标识符，以便在LLVM的折叠机制中追踪和管理这些节点。通过使用`AddInteger()`函数，`K`的数值被添加到唯一标识符中，这对于正确实现节点的折叠和唯一性检查是至关重要的。

# 定义 OnOffChecker

```c++
class OnOffChecker : public Checker<check::PostCall> {
private:
  mutable IdentifierInfo *IIOn, *IIOff;
  mutable std::unique_ptr<BugType> DoubleOffBugType;
  mutable std::unique_ptr<BugType> DoubleOnBugType;
  void initIdentifierInfo(ASTContext &Ctx) const;
  void reportDoubleOff(const CallEvent &Call, CheckerContext &C) const;
  void reportDoubleOn(const CallEvent &Call, CheckerContext &C) const;

public:
  OnOffChecker();
  /// Process on and off
  void checkPostCall(const CallEvent &Call, CheckerContext &C) const;
};
```

类的开头表明我们正在使用一个具有模板参数的 `Checker` 子类。对于这个类，你可以使用多个模板参数，它们表示你的检查器感兴趣访问的程序点。我们的检查器将从基类继承 `PostCall`。这种继承用于实现访问者模式，它将仅为我们感兴趣的对象调用我们，因此，我们的类必须实现成员函数 `checkPostCall`。

各种类型的程序点可见于 `CheckerDocumentation.cpp`。在我们的情况下，我们希望访问调用后的程序点，因为我们想要记录在调用函数之后状态的变化。

为了通知 Clang 我们正在处理新类型的错误，我们需要创建新的 `BugType` 实例。在这段代码中，有两个不同的 `BugType` 实例，分别用于报告两种不同的错误情况：当程序员调用 `On()` 两次和调用 `Off()` 两次时发生的错误。

# 注册映射关系
```c++
REGISTER_MAP_WITH_PROGRAMSTATE(OOS, int, OnOffState)
```

`REGISTER_MAP_WITH_PROGRAMSTATE` 是一个宏，用来在 `ProgramState` 中注册两个类型之间的映射关系。

第一个参数是 `map` 的名称，第二个参数是 `key` 的类型，第三个参数是 `value` 的类型。

所以这行代码表示在 `ProgramState` 中注册名为 `OOS` 的 `map`，`key` 的类型是 `int`，`value` 的类型是 `OnOffState`。

实际上我们不需要繁多的映射关系，因为我们在每个程序点始终只存储一个状态。因此，我们将始终使用键 1 来访问我们的映射。

要了解将信息注册到程序状态的其他方式，请查看 `CheckerContext.h` 中的宏定义。

# 实现 OnOffChecker

```c++
OnOffChecker::OnOffChecker() : IIOn(0), IIOff(0) {
  // Initialize the bug types.
  DoubleOffBugType.reset(
      new BugType(this, "Double Off", "On and Off Test Error"));
  DoubleOnBugType.reset(
      new BugType(this, "Double On", "On and Off Test Error"));
}
```
构造函数通过使用 `unique_ptr` 的 `reset()` 成员函数实例化了新的 `BugType` 对象。三个参数分别为checker、bug名称、bug类别。

```c++
void OnOffChecker::initIdentifierInfo(ASTContext &Ctx) const {
  if (IIOn)
    return;
  IIOn = &Ctx.Idents.get("On");
  IIOff = &Ctx.Idents.get("Off");
}
```
`ASTContext` 对象保存了包含用户程序中使用的类型和声明的特定 AST 节点，`Idents`是`IdentifierTable`类型，我们可以使用它来找到我们有兴趣监视的函数的确切标识符。此处监视函数`On()`和`Off()`。

```c++
void OnOffChecker::checkPostCall(const CallEvent &Call,
                                 CheckerContext &C) const {
  initIdentifierInfo(C.getASTContext());

  if (!Call.isGlobalCFunction())
    return;
  if (Call.getCalleeIdentifier() == IIOn) {
    ProgramStateRef State = C.getState();
    const OnOffState *S = State->get<OOS>(1);
    if (S && S->isOn()) {
      reportDoubleOn(Call, C);
      return;
    }
    State = State->set<OOS>(1, OnOffState::getOn());
    C.addTransition(State);
    return;
  }
  if (Call.getCalleeIdentifier() == IIOff) {
    ProgramStateRef State = C.getState();
    const OnOffState *S = State->get<OOS>(1);
    if (S && S->isOff()) {
      reportDoubleOff(Call, C);
      return;
    }
    State = State->set<OOS>(1, OnOffState::getOff());
    C.addTransition(State);
    return;
  }
}
```
`checkPostCall` 是 `const` function，不应该修改 checker 的状态。

第一个参数 - `CallEvent` 类型：

- 该参数保存有关程序在此程序点之前调用的确切函数的信息。这是因为我们注册了一个调用后的访问器（post-call visitor）。
- 在这里，我们使用 `CallEvent` 对象来检查是否调用了 `On()``Off()` 函数。

第二个参数 - `CheckerContext` 类型：

- 这个参数是在此程序点唯一提供当前状态信息的源，因为我们的检查器被强制保持无状态。
- 我们使用这个参数来检索 `ASTContext` 并初始化我们的 `IdentifierInfo` 对象，这些对象用于检查我们正在监视的函数。

处理流程：

- 我们通过询问 `CallEvent` 对象来检查是否调用了 `On()` 函数。如果是，进入处理。
- `ProgramStateRef State = C.getState();` 获取当前程序状态。
- `const OnOffState *S = State->get<OOS>(1);` 通过调用 `get<OOS>(1)` 获取存储在程序状态中的 `OnOffState` 对象，OOS 是我们注册的映射的名称，1 是映射的位置。
  - 如果 `S` 不为空且确实为`On` (说明`On`已经是第二次遇见了)，那么调用 `reportDoubleOn(Call, C);` 报告错误并返回。
  - 否则，说明这次第一次遇见`On`，我们将程序状态更新为将 `OnOffState` 设置为开启状态：`State = State->set<OOS>(1, OnOffState::getOn());`
- 最后，通过 `C.addTransition(State);` 将新的程序状态传递给 ExplodedGraph 创建新边，记录状态变化。这些边仅在状态实际更改时创建。

```c++
void OnOffChecker::reportDoubleOn(const CallEvent &Call,
                                  CheckerContext &C) const {
  ExplodedNode *ErrNode = C.generateSink();
  if (!ErrNode)
    return;
  BugReport *R = new BugReport(*DoubleOnBugType,
                               "Call the On() function two times", ErrNode);
  R->addRange(Call.getSourceRange());
  C.emitReport(R);
}
```
代码中的 `generateSink()` 意味着在可达程序状态图中生成一个汇聚节点(Sink Node)。这表示在当前路径上发现了一个严重的错误，因此我们不希望继续分析该路径。

第6行代码创建了一个 `BugReport` 对象，指定了我们发现了一个特定类型 `DoubleOnBugType` 的新错误，并且我们可以自由地添加描述并提供刚刚构建的错误节点（Sink 节点）。

我们使用 `addRange` 成员函数将错误位置的源代码范围添加到 `BugReport` 中。这样，用户在收到错误报告时可以看到源代码中具体发生错误的位置。

最后通过 `C.emitReport(R);` 向 `CheckerContext` 发送错误报告，将 `BugReport` 发送到报告系统，使得错误信息能够在适当的时机被输出或显示给用户。


# 注册
将文件名 `OnOffChecker.cpp` 添加到 `lib/StaticAnalyzer/Checkers/CMakeLists.txt` 中。

在 `lib/StaticAnalyzer/Checkers/Checkers.td` 中添加 

```tablegen
def PowerPlantAlpha : Package<"powerplant">, InPackage<Alpha>;

let ParentPackage = PowerPlantAlpha in {
def OnOffChecker : Checker<"OnOffChecker">,
  HelpText<"Check for misuses of the nuclear power plant API">,
  DescFile<"OnOffChecker.cpp">;
} // end "alpha.powerplant"
```

# 执行结果

```plaintext
$ clang  --analyze -Xanalyzer -analyzer-checker=alpha.powerplant test.c -o test.o
test.c:10:9: warning: Call the Off() function two times
        Off();
        ^~~~~
test.c:12:5: warning: Call the On() function two times
    On();
    ^~~~
2 warnings generated.
```