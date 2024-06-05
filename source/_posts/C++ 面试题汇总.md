
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

```C++
#include <iostream>
using namespace std;
class A {
private:
    int value;

public:
    A(int n) { value = n; }
    A(A other) { value = other.value; } // Copy constructor for deep copying

    void Print() { std::cout << value << std::endl; }
};

int main(int argc, char *argv[])
{
    A a=10;
    A b=a; // Create a copy of object a using the copy constructor
    b.Print();
    return 0;
}
```
```C++
#include <iostream>
using namespace std;
class A {
private:
    int value;

public:
    A(int n) { value = n; }
    A(A& other) { value = other.value; } // Copy constructor for deep copying

    void Print() { std::cout << value << std::endl; }
};

int main(int argc, char *argv[])
{
    A a(10);
    A b(a); // Create a copy of object a using the copy constructor
    b.Print();
    return 0;
}
```

```C++
#include <iostream>
using namespace std;
class A {
private:
    int value;

public:
    A(int n) { value = n; }
    A(A& other) { value = other.value; } // Copy constructor for deep copying

    void Print() { std::cout << value << std::endl; }
};

int main(int argc, char *argv[])
{
    A a=10;
    A b(a); // Create a copy of object a using the copy constructor
    b.Print();
    return 0;
}
```
```C++
#include <iostream>
using namespace std;
class A {
private:
    int value;

public:
    A(int n) { value = n; }
    A(const A& other) { value = other.value; } // Copy constructor for deep copying

    void Print() { std::cout << value << std::endl; }
};

int main(int argc, char *argv[])
{
    A a=10;
    A b=a; // Create a copy of object a using the copy constructor
    b.Print();
    return 0;
}
```



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