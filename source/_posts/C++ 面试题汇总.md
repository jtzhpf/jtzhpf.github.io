---
title: "C/C++ 面试题"
#category: CS&Maths
#id: 57
date: 2024-2-10 09:00:00
tags: 
  - C
  - C++
toc: true
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


### 怎么样禁止产生堆对象？(静态分配)
在C++中，如果你想禁止某个类生成堆对象（即只能在栈上创建该对象），你可以通过将类的 `operator new` 和 `operator delete` 重载并将它们声明为私有或删除（`= delete`）。这会阻止在堆上使用 `new` 操作符创建对象。

以下是一个简单的示例：

**方式 1: 重载并私有化 `operator new` 和 `operator delete`**
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

**方式 2: 删除 `operator new` 和 `operator delete`**
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

### 怎么样禁止产生栈对象？（动态分配）
在 C++ 中，可以通过将类的构造函数或析构函数声明为 `private` 或 `protected` 来禁止从栈上创建该类的对象。这样，外部代码无法直接在栈上声明类的对象，但可以通过允许堆上分配对象来控制对象的生命周期。

#### 方法 1: 使用私有构造函数和析构函数

通过将类的构造函数和析构函数声明为 `private` 或 `protected`，可以防止在栈上创建对象。用户可以通过静态工厂方法或动态内存分配（使用 `new`）来创建对象。


```cpp
#include <iostream>

class MyClass {
private:
    MyClass() { std::cout << "Object created\n"; }  // 私有构造函数
    ~MyClass() { std::cout << "Object destroyed\n"; }  // 私有析构函数

public:
    // 静态工厂方法，通过堆上分配对象
    static MyClass* createObject() {
        return new MyClass();
    }

    // 提供一个删除对象的接口
    static void destroyObject(MyClass* obj) {
        delete obj;
    }

    void display() const {
        std::cout << "Display method called\n";
    }
};

int main() {
    // MyClass obj;  // 错误！构造函数是私有的，不能在栈上创建对象

    // 只能通过堆上分配
    MyClass* obj = MyClass::createObject();
    obj->display();

    // 销毁对象
    MyClass::destroyObject(obj);

    return 0;
}
```

#### 方法 2: 使用 `protected` 构造函数和析构函数

使用 `protected` 而不是 `private` 可以让类的派生类（子类）继承并创建对象，但仍然阻止从栈上直接创建基类对象。

```cpp
#include <iostream>

class BaseClass {
protected:
    BaseClass() { std::cout << "BaseClass created\n"; }  // 受保护的构造函数
    ~BaseClass() { std::cout << "BaseClass destroyed\n"; }  // 受保护的析构函数
};

class DerivedClass : public BaseClass {
public:
    DerivedClass() {
        std::cout << "DerivedClass created\n";
    }
    ~DerivedClass() {
        std::cout << "DerivedClass destroyed\n";
    }
};

int main() {
    // BaseClass base;  // 错误！受保护的构造函数禁止栈上创建

    DerivedClass derived;  // 允许栈上创建派生类对象
    return 0;
}
```

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

### 传参顺序

```C++
#include <iostream>
using namespace std;

int f(int n) 
{
	cout << "f():"<<n << endl;
	return n;
}

void func(int param1, int param2)
{
	int var1 = param1;
	int var2 = param2;
	printf("var1=%d,var2=%d\n", f(var1), f(var2));//如果将printf换为cout进行输出，输出结果则刚好相反
	cout <<"var1="<< f(var1)<<",var2="<< f(var2) << endl;
}

int main(int argc, char* argv[])
{
	func(1, 2);
	return 0;
}
//输出结果
//f():2
//f():1
//var1=1,var2=2
//var1=f():1
//1,var2=f():2
//2
```

### 移动构造函数
移动构造函数在 C++ 中用于实现对象的“移动语义”，这使得资源（如内存或文件句柄）从一个对象转移到另一个对象，而无需复制。移动构造函数通过使用 `std::move` 来获取一个右值引用，从而避免不必要的资源复制操作，提高程序的性能。

下面是一个简单的例子，展示了如何实现和使用移动构造函数：

```cpp
#include <iostream>
#include <vector>
#include <utility>  // for std::move

class MyVector {
public:
    int* data;     // 指向动态分配的数组
    size_t size;   // 数组大小

    // 构造函数
    MyVector(size_t s) : size(s), data(new int[s]) {
        std::cout << "Constructing MyVector of size " << size << std::endl;
    }

    // 拷贝构造函数
    MyVector(const MyVector& other) : size(other.size), data(new int[other.size]) {
        std::cout << "Copy constructor called" << std::endl;
        std::copy(other.data, other.data + other.size, data);
    }

    // 移动构造函数
    MyVector(MyVector&& other) noexcept : size(other.size), data(other.data) {
        std::cout << "Move constructor called" << std::endl;
        // 移动后，将原对象的指针和大小置空
        other.data = nullptr;
        other.size = 0;         // 必须的，如果没有会出现double free的问题
    }

    // 析构函数
    ~MyVector() {
        std::cout << "Destructor called" << std::endl;
        delete[] data;
    }
};

int main() {
    // 创建一个对象
    MyVector vec1(5); // 调用构造函数

    // 使用 std::move 将 vec1 移动到 vec2
    MyVector vec2(std::move(vec1)); // 调用移动构造函数

    // vec1 已被移动，不能再访问其资源
    std::cout << "vec1's size after move: " << vec1.size << std::endl;
    std::cout << "vec1's data after move: " << vec1.data << std::endl;

    return 0;
}
```

**代码解释：**

- 1. **构造函数**：创建一个指定大小的动态数组，并初始化 `size` 和 `data`。
- 2. **拷贝构造函数**：深拷贝另一个对象的 `data`，创建新的内存块以避免资源共享。
- 3. **移动构造函数**：
   - 使用右值引用 `MyVector&&` 接收一个即将销毁的对象 `other`。
   - 将 `other` 的资源（`data` 和 `size`）转移给当前对象。
   - 将 `other` 的指针设为 `nullptr`，并将 `size` 设为 0，防止 `other` 在销毁时释放已转移的资源。
- 4. **析构函数**：在对象销毁时释放动态分配的内存。
- 5. **`std::move`**：将 `vec1` 转换为右值，以便调用移动构造函数，而不是拷贝构造函数。

**输出结果：**

```text
Constructing MyVector of size 5
Move constructor called
vec1's size after move: 0
vec1's data after move: 0
Destructor called
Destructor called
```

- 创建 `vec1` 时，调用了构造函数，分配了大小为 5 的数组。
- 通过 `std::move`，`vec1` 的资源（内存）被转移到了 `vec2`，因此调用了移动构造函数。
- 移动后，`vec1` 的 `size` 变为 0，`data` 变为 `nullptr`，确保移动语义的正确性。
- 在程序结束时，分别调用 `vec1` 和 `vec2` 的析构函数，清理资源。

### 基本数据类型大小
以下是常见基本数据类型在 32 位和 64 位系统上的大小对比：

| 数据类型    | 32 位系统大小 (bytes) | 64 位系统大小 (bytes) |
|-------------|------------------------|------------------------|
| `char`      | 1                      | 1                      |
| `short`     | 2                      | 2                      |
| `int`       | 4                      | 4                      |
| `long`      | 4                      | 8                      |
| `long long` | 8                      | 8                      |
| `float`     | 4                      | 4                      |
| `double`    | 8                      | 8                      |
| `long double`| 12 (或 16)             | 16                     |
| `pointer`   | 4                      | 8                      |


### 结构体大小

一、结构体对齐规则首先要看有没有用`#pragma pack`宏声明，这个宏可以改变对齐规则，有宏定义的情况下**结构体的自身宽度就是按照这个宏声明的和实际数据类型中最大值较小的那个**来决定，所有内存都按照这个宽度去布局，`#pragma pack` 参数只能是 '1', '2', '4', '8', or '16'。

二、在没有`#pragma pack`这个宏的声明下，遵循下面三个原则：

1. 第一个成员的首地址为0；\n
2. 每个成员的首地址是自身大小的整数倍；\n
3. 结构体的总大小，为其成员中所含**最大类型的整数倍**。

```C++
#include <iostream>
#include <stddef.h>
using namespace std;

int main()
{
	cout << "char: " << sizeof(char) << endl;       // char: 1
	cout << "short: " << sizeof(short) << endl;     // short: 2
	cout << "int: " << sizeof(int) << endl;         // int: 4
	cout << "long: " << sizeof(long) << endl;       // long: 8
	cout << "long long: " << sizeof(long long) << endl; // long long: 8
	cout << "float: " << sizeof(float) << endl;         // float: 4
	cout << "double: " << sizeof(double) << endl;       // double: 8
	cout << "long double: " << sizeof(long double) << endl; // long double: 16

	struct S1
	{
		int x;      // 0(4)
		char y;     // 4(1+3)
		int z;      // 8(4+4)
		double a;   // 16(8)
	};              // 24(以double 8B对齐)
	cout << offsetof(S1, x) << endl;
	cout << offsetof(S1, y) << endl;
	cout << offsetof(S1, z) << endl;
	cout << offsetof(S1, a) << endl;
	cout << sizeof(S1) << endl;

	struct S2
	{
		int x;      // 0(4)
		char y;     // 4(1+1)
		short b;    // 6(2)
		int z;      // 8(4+4)
		double a;   // 16(8)
	};              // 24(以double 8B对齐)
	cout << offsetof(S2, x) << endl;
	cout << offsetof(S2, y) << endl;
	cout << offsetof(S2, b) << endl;
	cout << offsetof(S2, z) << endl;
	cout << offsetof(S2, a) << endl;
	cout << sizeof(S2) << endl;

	struct S3
	{
		char a;     // 0(1)
		char b;     // 1(1)
		char c;     // 2(1)
	};              // 3(char 1B对齐)
	cout << offsetof(S3, a) << endl;
	cout << offsetof(S3, b) << endl;
	cout << offsetof(S3, c) << endl;
	cout << sizeof(S3) << endl;

	struct S4
	{
		char a;     // 0(1+1)
		short b;    // 2(2)
		char c;     // 4(1+1)
	};              // 6(short 2B对齐)
	cout << offsetof(S4, a) << endl;
	cout << offsetof(S4, b) << endl;
	cout << offsetof(S4, c) << endl;
	cout << sizeof(S4) << endl;

	struct S5
	{
		char a;     // 0(1+3)
		int  b;     // 4(4)
		short c;    // 8(2+2)
	};              // 12(short 2B对齐)
	cout << offsetof(S5, a) << endl;
	cout << offsetof(S5, b) << endl;
	cout << offsetof(S5, c) << endl;
	cout << sizeof(S5) << endl;

	struct S6
	{
		char d[7];  // 0(7+1)
		float a;    // 8(4)
		short  b;   // 12(2+2)
		char* c;    // 16(8)
	};              // 24(float 4B对齐)
	cout << offsetof(S6, d) << endl;
	cout << offsetof(S6, a) << endl;
	cout << offsetof(S6, b) << endl;
	cout << offsetof(S6, c) << endl;
	cout << sizeof(S6) << endl;

	struct S7
	{
		char d[7];  // 0(7+1)
		double a;   // 8(8)
		short  b;   // 16(2+6)
		char* c;    // 24(8)
		char e;     // 32(1+1)
		short f;    // 34(2+4)
	};              // 40(double 8B对齐)
	cout << offsetof(S7, d) << endl;
	cout << offsetof(S7, a) << endl;
	cout << offsetof(S7, b) << endl;
	cout << offsetof(S7, c) << endl;
	cout << offsetof(S7, e) << endl;
    cout << offsetof(S7, f) << endl;
	cout << sizeof(S7) << endl;
	return 0;
}
```
#### #pragma pack
加了 `#pragma pack` 会怎样？
```c++
#include <iostream>
#include <stddef.h>
using namespace std;

int main()
{
    #pragma  pack(1)
	struct S1
	{
		int x;      // 0(4)
		char y;     // 4(1)
		int z;      // 5(4)
		double a;   // 9(8)
	};              // 17
	cout << offsetof(S1, x) << endl;
	cout << offsetof(S1, y) << endl;
	cout << offsetof(S1, z) << endl;
	cout << offsetof(S1, a) << endl;
	cout << sizeof(S1) << endl;
	
	#pragma  pack(2)
	struct S2
	{
		int x;      // 0(4)
		char y;     // 4(1+1)
		int z;      // 6(4)
		double a;   // 10(8)
	};              // 18
	cout << offsetof(S2, x) << endl;
	cout << offsetof(S2, y) << endl;
	cout << offsetof(S2, z) << endl;
	cout << offsetof(S2, a) << endl;
	cout << sizeof(S2) << endl;
	
	#pragma  pack(16)
	struct S3
	{
		int x;      // 0
		char y;     // 4
		int z;      // 8
		double a;   // 16
	};              // 24
	cout << offsetof(S3, x) << endl;
	cout << offsetof(S3, y) << endl;
	cout << offsetof(S3, z) << endl;
	cout << offsetof(S3, a) << endl;
	cout << sizeof(S3) << endl;
	return 0;
}
```
**`#pragma pack` 对齐规则总结:**

1. **默认对齐（没有 `#pragma pack`）**：
   - 默认情况下，编译器按照结构体成员类型的大小（也叫“自然对齐”）进行对齐。每个成员在结构体中的地址必须是该成员大小的倍数。
   - 编译器会插入填充字节以确保每个成员在正确的对齐边界上。

2. **`#pragma pack(n)` 指定对齐**：
   - 使用 `#pragma pack(n)`，可以显式指定一个对齐大小 `n`，告诉编译器所有成员按照 `n` 字节对齐。
   - 当成员的自然对齐要求大于 `n` 时，成员会按 `n` 字节对齐（即强制对齐），如果自然对齐小于或等于 `n`，则按成员的自然对齐进行对齐。

3. **对齐的行为**：
   - **成员对齐**：`#pragma pack(n)` 会限制结构体中每个成员的对齐方式，所有成员的起始地址都要对齐到 `n` 或该成员类型的自然对齐值，取较小者。例如，如果 `n` 为 2，`int` 类型的成员将按 2 字节对齐（虽然它通常按 4 字节对齐）。
   - **结构体大小对齐**：结构体的总大小通常也会对齐到 `n` 字节或最大成员的自然对齐值的倍数。

#### 位域
除此之外，还有一种特殊情况，叫**位域**：
```C++
	struct S1
	{
        char a : 7;   // 7 bits
        int b : 11;   // 11 bits
        int c : 4;    // 4 bits
        int d : 10;   // 10 bits
        char index;   // 1 byte (8 bits)
	};                // 8 byte
	cout << offsetof(S1, a) << endl;    // offsetof 在位域情况下编译报错！
	cout << sizeof(S1) << endl;
```
在这个 `struct test` 中，使用了 **位域（bit field）** 来定义结构体成员。位域允许你定义精确的位数来存储某个成员。理解位域的对齐和存储方式稍微复杂一些，因为它涉及到位的排列和字节对齐。让我们逐一分析每个成员的存储方式和对齐情况。

**位域的存储规则：**
- 位域的存储依赖于具体的编译器实现，但通常来说，位域会按照其底层类型的大小进行分配和存储。例如，`int` 通常占 4 字节（32 位），位域会在这个范围内尽可能地放置多个字段。
- 如果一个字段不能放入当前的剩余位域空间中，那么它会被存储在新的机器字中，新的机器字会遵守该字段类型的对齐规则。

**逐个字段分析：**

1. **`char a : 7`**：
   - `char a` 占用 7 位，但由于 `char` 是 1 字节（8 位），剩下的 1 位可能留作填充位或用于存储后续位域。
   - 假设编译器允许多个位域共享同一个字节，因此 `a` 占据结构体的前 7 位。

2. **`int b : 11`**：
   - `int b` 需要 11 位。由于 `char a : 7` 已占用了 1 字节中的 7 位，`b` 的前 1 位将填充剩下的第一个字节，后续的 10 位将跨越到下一个 32 位的 `int` 单元。
   - 也就是说，`b` 的前 1 位放在第一个字节的最后一位，剩余的 10 位占用第二个字节的前 10 位。

3. **`int c : 4`**：
   - `c` 需要 4 位。剩下的 `b` 已经占用了 11 位，因此 `c` 可以紧接着在剩余的 `b` 后面的位上使用。这意味着 `c` 将占据剩余的 4 位，而不需要新开辟一个新的机器字。

4. **`int d : 10`**：
   - `d` 需要 10 位。由于前面已经使用了 `11 + 4 = 15` 位，剩下的空间不足以存储 `d`。因此，`d` 将被存储在新的 `int` 单元中，跨越两个字节。

5. **`char index`**：
   - `index` 是一个完整的 `char`，占用 1 字节的存储空间，按 1 字节对齐。

**总体布局和对齐：**

- 位域成员 `a`, `b`, `c`, 和 `d` 将按照上面分析的位来分配，总共使用了 7 + 11 + 4 + 10 = 32 位（4 字节）来存储这些成员。
- 位域成员会被打包在一起，不会跨越超过其类型所能表示的范围。例如，`int` 位域成员不会跨越 32 位的机器字边界。
- 由于 `index` 是一个常规的 `char` 成员，它将按照 1 字节对齐，存储在新的字节空间中。

**计算大小：**

假设 `int` 是 4 字节，`char` 是 1 字节，并且结构体的对齐按 4 字节来进行：

1. **位域部分**：`a + b + c + d` 总共占 32 位，正好等于 4 字节（1 个 `int` 的大小）。
2. **`index` 成员**：`char index` 占用 1 字节。

因此，结构体的总大小为：
- 4 字节（位域部分）
- 1 字节（`char index`）
- 再加上对齐需要的 3 个填充字节（成员int为4字节，为了让整个结构体的大小符合 4 字节对齐规则）。

最终结构体的总大小为 8 字节。

### 静态类型、动态类型、静态绑定和动态绑定
在 C++ 中，静态类型、动态类型、静态绑定和动态绑定是重要的概念，用于理解如何解析函数调用和类型信息。以下是这几个概念的解释和示例：

#### 1. 静态类型（Static Type）
**静态类型**是编译时确定的类型。它是指在编译过程中变量的声明类型。编译器根据静态类型来检查代码的正确性。

**示例**：
```cpp
class Base {
public:
    void show() { std::cout << "Base show\n"; }
};

class Derived : public Base {
public:
    void show() { std::cout << "Derived show\n"; }
};

int main() {
    Base* basePtr; // 静态类型是 Base*
    Derived derivedObj;
    basePtr = &derivedObj; // 静态类型为 Base*，动态类型为 Derived
}
```
在这个示例中，`basePtr` 的静态类型是 `Base*`，它在编译时确定。

#### 2. 动态类型（Dynamic Type）
**动态类型**是在运行时确定的类型。它是对象的实际类型，而不是指向对象的指针或引用的类型。

**示例**：
```cpp
int main() {
    Base* basePtr = new Derived(); // 静态类型是 Base*，动态类型是 Derived
    basePtr->show(); // 调用的是 Derived::show，因为实际对象是 Derived
    delete basePtr;
}
```
在这个示例中，虽然 `basePtr` 的静态类型是 `Base*`，但动态类型是 `Derived`。因此，`basePtr->show()` 调用的是 `Derived` 类中的 `show` 函数。

#### 3. 静态绑定（Static Binding）
**静态绑定**（或称为早期绑定）是编译时确定函数的调用。函数调用的具体实现由编译器在编译阶段决定。

**示例**：
```cpp
class Example {
public:
    void display() { std::cout << "Static binding\n"; }
};

int main() {
    Example obj;
    obj.display(); // 这里的 display() 调用在编译时决定，属于静态绑定
}
```
在这个示例中，`obj.display()` 是静态绑定，因为编译器在编译时决定了 `display` 方法的调用。

#### 4. 动态绑定（Dynamic Binding）
**动态绑定**（或称为晚期绑定）是在运行时确定函数的调用。它通常用于虚函数（通过虚函数表 vtable）来实现多态性。

**示例**：
```cpp
class Base {
public:
    virtual void show() { std::cout << "Base show\n"; }
};

class Derived : public Base {
public:
    void show() override { std::cout << "Derived show\n"; }
};

int main() {
    Base* basePtr = new Derived(); // 静态类型是 Base*，动态类型是 Derived
    basePtr->show(); // 动态绑定，实际调用的是 Derived::show()
    delete basePtr;
}
```
在这个示例中，`basePtr->show()` 是动态绑定，因为实际的 `show` 方法在运行时通过 `Derived` 类的虚函数表来确定。

**总结:**
- **静态类型**：编译时确定的类型（指针或引用的声明类型）。
- **动态类型**：运行时确定的类型（对象的实际类型）。
- **静态绑定**：编译时确定函数调用。
- **动态绑定**：运行时确定函数调用，通常通过**虚函数**实现多态性。

这些概念在 C++ 的面向对象编程中尤为重要，有助于理解如何进行函数调用、实现多态以及管理类型信息。

### 虚函数和默认参数

```C++
#include <iostream>
using namespace std;

class E
{
public:
	virtual void func(int i = 0)
	{
		std::cout << "E::func()\t" << i << "\n";
	}
};
class F : public E
{
public:
	virtual void func(int i = 1)
	{
		std::cout << "F::func()\t" << i << "\n";
	}
};

void test2()
{
	F* pf = new F();
	E* pe = pf;
	pf->func(); //F::func() 1  正常，就该如此；
	pe->func(); //F::func() 0  哇哦，这是什么情况，调用了子类的函数，却使用了基类中参数的默认值！
}
int main()
{
	test2();
	return 0;
}

```
这个代码的行为展示了**虚函数**和**默认参数**在 C++ 中的工作原理。关键问题出现在：

```cpp
pe->func(); // F::func() 0
```

当调用 `pe->func();` 时，虽然函数调用的是派生类 `F` 的 `func()` 函数，但默认参数的值却使用了基类 `E` 中的默认值 `0`。这看似反常的行为其实是由 C++ 默认参数的机制造成的。

**关键点：**
1. **虚函数调用**：
   - C++ 中的虚函数调用通过**虚函数表（vtable）** 实现。在运行时，调用的函数是根据对象的动态类型决定的。因此，`pe` 是 `E*` 类型的指针，但它指向的是 `F` 类型的对象，所以调用的函数是 `F::func()`（派生类的重写函数）。

2. **默认参数**：
   - C++ 默认参数的解析是**在编译时**完成的，而不是在运行时。默认参数是编译器在解析函数调用时直接替换的。因此，默认参数的值取决于调用时编译器看到的函数签名。
   - 在 `pe->func()` 这一行代码中，虽然调用的是 `F::func()`，但编译器在解析 `pe` 时，看到的是 `E*` 类型指针，因此它使用了基类 `E` 中的默认参数 `0`。

**为什么发生这种情况？**

- 对于 `pf->func();`，编译器知道 `pf` 是 `F*` 类型的指针，所以使用 `F::func()` 的默认参数 `1`。
- 对于 `pe->func();`，编译器认为 `pe` 是 `E*` 类型的指针，因此它使用了 `E::func()` 的默认参数 `0`，尽管最终在运行时调用的是 `F::func()`。

**解决方案:**

要避免这种令人困惑的行为，有几种方式可以选择：

1. **避免在虚函数中使用默认参数**：
   - 默认参数在虚函数中容易引发混淆，因为参数是在编译时绑定，而虚函数是在运行时决定调用的。最好在虚函数的参数中避免默认值，显式传递参数。

2. **将默认参数设置为一致**：
   - 如果一定要使用默认参数，确保在基类和派生类中都使用相同的默认值。

**修改代码避免问题的示例:**

将基类和派生类中的默认参数值保持一致可以避免混淆：

```cpp
class E
{
public:
	virtual void func(int i = 1) // 修改默认值为 1
	{
		std::cout << "E::func()\t" << i << "\n";
	}
};
class F : public E
{
public:
	virtual void func(int i = 1) // 保持默认值一致
	{
		std::cout << "F::func()\t" << i << "\n";
	}
};
```

这样，调用 `pe->func();` 时，默认参数始终是 `1`，结果一致。

**总结:**

- 虚函数的决策是**运行时**基于对象的动态类型进行的。
- 默认参数的决策是**编译时**基于指针或引用的静态类型进行的。
- 在虚函数中使用默认参数容易引发不一致的行为，因此应尽量避免。