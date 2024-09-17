
---
title: "C/C++ 面试题"
#category: CS&Maths
#id: 57
date: 2024-2-10 09:00:00
tags: 
  - C
  - C++
#toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
#timeline: article  # 展示在时间线列表中
mathjax: true
---


### 二维数组的指针与解指针
```C
#include <stdio.h>
typedef int A[10][20]; 
A a;
A* b=a;
void fun() { 
    printf("a:\t\t%d\n", a);
    printf("a[0]:\t\t%d\n", a[0]);
    printf("*a:\t\t%d\n", *a);
    printf("*a+1:\t\t%d\n", *a+1);
    printf("&a[0][0]:\t%d\n", &a[0][0]);
    printf("&a[0][1]:\t%d\n", &a[0][1]);
    printf("a+1:\t\t%d\n", a+1);
    printf("a[1]:\t\t%d\n", a[1]);
    printf("&a[1][0]:\t%d\n", &a[1][0]);
    printf("&a[1][1]:\t%d\n", &a[1][1]);
    printf("&a+1:\t\t%d\n", &a+1);
    printf("&a[0]+1:\t%d\n", &a[0]+1);
    printf("&a[0][0]+1:\t%d\n", &a[0][0]+1);
    // printf("&&a[0][0]+1:\t%d\n", &&a[0][0]+1);  // wrong
    printf("\n");
    printf("b:\t\t%d\n", b);
    printf("b+1:\t\t%d\n", b+1);
    printf("&b:\t\t%d\n", &b);
} 

int main() {
    fun(); 
    return 0;
}
```
```plaintext
a:              164651040
a[0]:           164651040
*a:             164651040
*a+1:           164651044
&a[0][0]:       164651040
&a[0][1]:       164651044
a+1:            164651120
a[1]:           164651120
&a[1][0]:       164651120
&a[1][1]:       164651124
&a+1:           164651840
&a[0]+1:        164651120
&a[0][0]+1:     164651044

b:              164651040
b+1:            164651840
&b:             164651024
```
二维数组`a[10][20]`中，`a+1`、`a[1]`、`&a[0]+1`表示第一层数组元素（a[0]、a[1]...）的地址，`*a+1`、`a[0][1]`、`&a[0][0]+1`表示第二层数组元素（a[0][0]、a[0][1]...）的地址，`&a`表示二维数组本身的地址。

如果将该二维数组赋值给指针`b`，则`b`表示二维数组本身的地址，`&b`表示指针本身的地址。


### 怎么样禁止产生堆对象？
在C++中，如果你想禁止某个类生成堆对象（即只能在栈上创建该对象），你可以通过将类的 `operator new` 和 `operator delete` 重载并将它们声明为私有或删除（`= delete`）。这会阻止在堆上使用 `new` 操作符创建对象。

以下是一个简单的示例：

### 方式 1: 重载并私有化 `operator new` 和 `operator delete`
```cpp
class MyClass {
public:
    // 公共构造函数和析构函数
    MyClass() {}
    ~MyClass() {}

private:
    // 私有化的 new 和 delete 操作符，禁止堆分配
    void* operator new(size_t size) {
        return ::operator new(size);
    }
    void operator delete(void* ptr) {
        ::operator delete(ptr);
    }
};

int main() {
    MyClass obj;               // 正确，栈上分配
    MyClass* pObj = new MyClass();  // 错误，无法在堆上分配对象
    return 0;
}
```

在这个示例中，`operator new` 和 `operator delete` 被声明为私有，这意味着在类的外部无法通过 `new` 操作符在堆上创建对象。由于 `main` 函数无法访问私有成员，尝试在堆上创建对象时会导致编译错误。

### 方式 2: 删除 `operator new` 和 `operator delete`
```cpp
class MyClass {
public:
    // 公共构造函数和析构函数
    MyClass() {}
    ~MyClass() {}

    // 禁止在堆上创建对象
    void* operator new(size_t) = delete;
    void operator delete(void*) = delete;
};

int main() {
    MyClass obj;               // 正确，栈上分配
    MyClass* pObj = new MyClass();  // 错误，无法在堆上分配对象
    return 0;
}
```

在这个示例中，我们将 `operator new` 和 `operator delete` 标记为 `= delete`，这会完全禁止在堆上创建和销毁对象。这样做的效果是，当你试图使用 `new` 操作符在堆上分配对象时，编译器会直接给出错误提示。

通过这些方法，你可以确保某个类只能在栈上创建，而无法在堆上分配。
### 怎么样禁止产生栈对象？
在C++中，如果你想禁止某个类生成栈对象（即只能在堆上动态分配该对象），你可以通过将类的析构函数声明为私有或受保护的来实现。这种方法会导致编译器禁止在栈上创建对象，因为栈对象的析构会在作用域结束时自动调用，而如果析构函数是私有或受保护的，那么在栈上创建该对象将会引发编译错误。

以下是一个简单的示例：

```cpp
class MyClass {
public:
    // 公共构造函数
    MyClass() {}

protected:
    // 受保护的析构函数
    ~MyClass() {}
};

int main() {
    MyClass obj;            // 错误，无法在栈上创建对象
    MyClass* pObj = new MyClass();  // 正确，可以在堆上动态分配对象
    delete pObj;            // 手动删除堆上的对象
    return 0;
}
```

在这个示例中，`MyClass`的析构函数是受保护的，这意味着你不能在类的外部直接销毁对象。因此，试图在栈上创建该类的对象将导致编译错误。然而，通过指针在堆上动态分配对象是允许的，因为你可以在堆上使用 `new` 操作符创建对象，并使用 `delete` 操作符手动销毁它们。

需要注意的是，使用这种方式的一个副作用是，派生类可能仍然可以在栈上创建对象，因为派生类可以访问受保护的成员。为了防止这种情况，可以将析构函数声明为私有，这样即使是派生类也无法在栈上创建对象。

```cpp
class MyClass {
public:
    // 公共构造函数
    MyClass() {}

private:
    // 私有析构函数
    ~MyClass() {}
};

int main() {
    MyClass obj;            // 错误，无法在栈上创建对象
    MyClass* pObj = new MyClass();  // 正确，可以在堆上动态分配对象
    delete pObj;            // 手动删除堆上的对象
    return 0;
}
```

通过将析构函数声明为私有，你可以确保该类对象只能通过堆分配方式创建，而不能在栈上直接创建。

### 对象复用与零拷贝
![](/C/C++%20面试题/image1.png)

```C++
#include <vector>
#include <string>
#include <iostream>
using namespace std;

struct Person
{
    string name;
    int age;
    //初始构造函数
    Person(string p_name, int p_age): name(std::move(p_name)), age(p_age)
    {
         cout << "\tI have been constructed" <<endl;
    }
     //拷贝构造函数
     Person(const Person& other): name(std::move(other.name)), age(other.age)
    {
         cout << "\tI have been copy constructed" <<endl;
    }
     //转移构造函数
     Person(Person&& other): name(std::move(other.name)), age(other.age)
    {
         cout << "\tI have been moved"<<endl;
    }
};

int main()
{
    vector<Person> people;

    Person p1("Alice", 30);
    
    // 使用 push_back 插入对象
    cout<<"1.push_back:"<<endl;
    people.push_back(p1); // 调用拷贝构造函数

    // 使用 push_back 插入对象
    cout<<"2.push_back:"<<endl;
    people.push_back(Person("Bob", 25)); // 调用移动构造函数，因为临时对象被传递
    
    // 使用 emplace_back 直接原地构造对象
    cout<<"3.emplace_back:"<<endl;
    people.emplace_back("Charlie", 20); // 直接构造，无需拷贝或移动

    return 0;
}
//预期结果：
//        I have been constructed  // 构造 p1
//1. push_back:
//        I have been copy constructed  // 复制 p1
//2. push_back:
//        I have been constructed  // 构造临时对象 "Bob"
//        I have been moved        // 移动临时对象 "Bob"
//3. emplace_back:
//        I have been constructed  // 直接构造 "Charlie"
```
```c++
//输出结果：
//        I have been constructed
//1.push_back:
//        I have been copy constructed
//2.push_back:
//        I have been constructed
//        I have been moved
//        I have been copy constructed
//3.emplace_back:
//        I have been constructed
//        I have been copy constructed
//        I have been copy constructed
```
嗯？怎么和说的不一样？最后**emplace_back**怎么调用了2次拷贝构造函数？
这次让我们换一下顺序。
```C++
int main()
{
    vector<Person> people;

    Person p1("Alice", 30);
        
    // 使用 emplace_back 直接原地构造对象
    cout<<"3.emplace_back:"<<endl;
    people.emplace_back("Charlie", 20); // 直接构造，无需拷贝或移动
    
    // 使用 push_back 插入对象
    cout<<"1.push_back:"<<endl;
    people.push_back(p1); // 调用拷贝构造函数
    
    // 使用 push_back 插入对象
    cout<<"2.push_back:"<<endl;
    people.push_back(Person("Bob", 25)); // 调用移动构造函数，因为临时对象被传递

    return 0;
}
//输出结果：
//        I have been constructed
//3.emplace_back:
//        I have been constructed
//1.push_back:
//        I have been copy constructed
//        I have been copy constructed
//2.push_back:
//        I have been constructed
//        I have been moved
//        I have been copy constructed
//        I have been copy constructed
```

```C++
int main()
{
    vector<Person> people;

    Person p1("Alice", 30);
    
    // 使用 push_back 插入对象
    cout<<"2.push_back:"<<endl;
    people.push_back(Person("Bob", 25)); // 调用移动构造函数，因为临时对象被传递
    
    // 使用 emplace_back 直接原地构造对象
    cout<<"3.emplace_back:"<<endl;
    people.emplace_back("Charlie", 20); // 直接构造，无需拷贝或移动
    
    // 使用 push_back 插入对象
    cout<<"1.push_back:"<<endl;
    people.push_back(p1); // 调用拷贝构造函数
    
    return 0;
}
//输出结果：
//        I have been constructed
//2.push_back:
//        I have been constructed
//        I have been moved
//3.emplace_back:
//        I have been constructed
//        I have been copy constructed
//1.push_back:
//        I have been copy constructed
//        I have been copy constructed
//        I have been copy constructed
```
我们可以发现**emplace_back确实没有调用拷贝/移动构造函数，只需使用普通构造函数**。那么上面这个问题是为什么？

输出和预期不符，主要原因在于 `std::vector` 的动态扩容行为。具体来说，当 `std::vector` 需要扩展其内部存储空间时，它会重新分配更大的存储空间，并将已有的元素拷贝或移动到新的存储空间中。因此，发生了多次拷贝构造或移动构造，而这些额外的操作就是导致输出和预期不同的原因。

`std::vector` 在插入元素时会动态调整其容量（`capacity`）。当 `size` 超过当前的 `capacity` `时，vector` 会重新分配更大的内存，并将旧的元素拷贝到新的位置。这就是为什么你看到多次调用拷贝构造函数的原因，尤其是在 `emplace_back` 和 `push_back` 操作之间。

默认情况下，`vector` 的扩展通常是按一定倍数增加容量，比如 2 倍。当 `vector` 需要重新分配存储空间时，会调用拷贝构造函数或移动构造函数来将现有元素复制或移动到新的内存中。

如果你想避免额外的拷贝或移动构造操作，可以在开始时手动预留足够的空间，使用 `vector::reserve` 函数。
```c++
int main()
{
    vector<Person> people;
    people.reserve(3); // 预留3个元素的空间，避免扩容

    Person p1("Alice", 30);

    // 使用 push_back 插入对象
    cout << "1.push_back:" << endl;
    people.push_back(p1); // 调用拷贝构造函数

    // 使用 push_back 插入对象
    cout << "2.push_back:" << endl;
    people.push_back(Person("Bob", 25)); // 调用移动构造函数

    // 使用 emplace_back 直接原地构造对象
    cout << "3.emplace_back:" << endl;
    people.emplace_back("Charlie", 20); // 直接构造，无需拷贝或移动

    return 0;
}
//输出结果：
//        I have been constructed
//1.push_back:
//        I have been copy constructed
//2.push_back:
//        I have been constructed
//        I have been moved
//3.emplace_back:
//        I have been constructed
```