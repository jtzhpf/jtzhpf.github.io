---
title: C++ std::enable_if 的几种用法
category: CS&Maths
# id: 57
date: 2024-1-10 18:42:32
tags: 
  - C++
  - Metaprogramming
#toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: code  # 展示在时间线列表中
# gitalk: true # 启用 Gitalk 评论
---
`std::enable_if` 是 C++11 中 `<type_traits>` 头文件提供的一个模板类，用于在模板中根据条件启用或禁用特定的模板实例化。它通常与函数模板、类模板结合使用，以根据类型特性或其他条件进行编译时分支选择。

`std::enable_if` 提供了一种在编译时根据条件进行模板特化的强大机制，用于实现 SFINAE（Substitution Failure Is Not An Error）原则，避免在模板实例化时产生编译错误。


```cpp
template <bool B, class T = void>
struct enable_if {};

template <class T>
struct enable_if<true, T> {
    typedef T type;
};
```

这是 `std::enable_if` 的基本结构，它是一个模板类，根据布尔值 `B` 的真假来选择不同的模板实例化。

**基本用法：**

```cpp
template <typename T, typename std::enable_if<std::is_integral<T>::value, int>::type = 0>
void foo(T value) {
    // 函数体
}
```

在上面的例子中，`std::enable_if` 用于定义一个函数模板 `foo`，它仅在 `T` 是整数类型时有效。如果 `T` 是整数类型，`enable_if` 的 `type` 将是 `int`，函数就会被实例化；否则，`enable_if` 的 `type` 将不存在，编译器将忽略这个函数模板。

**使用模板别名简化语法（C++14及更高版本）：**

```cpp
template <typename T, typename = std::enable_if_t<std::is_integral<T>::value>>
void foo(T value) {
    // 函数体
}
```

C++11 引入了模板别名 `std::enable_if_t`，`enable_if_t`就是`enable_if::type`的重定义，它使语法更加清晰。

`typename = `：这部分是将 `std::enable_if` 的结果赋值给一个未命名的类型，通过 `typename` 关键字指示这是一个类型，而 `=` 意味着它是一个默认参数。由于我们不需要这个类型的名字，通常将其命名为 `_` 或者留空。

**使用在函数返回值中：**

```cpp
template <typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
bar(T value) {
    // 函数体
    return value;
}
```

这个例子中，`bar` 函数返回类型仅在 `T` 是整数类型时存在。

### Example
```cpp
#include <iostream>
#include <new>
#include <string>
#include <type_traits>
 
namespace detail
{ 
    void* voidify(const volatile void* ptr) noexcept { return const_cast<void*>(ptr); } 
}
 
// #1, enabled via the return type
template<class T>
typename std::enable_if<std::is_trivially_default_constructible<T>::value>::type 
    construct(T*) 
{
    std::cout << "default constructing trivially default constructible T\n";
}
 
// same as above
template<class T>
typename std::enable_if<!std::is_trivially_default_constructible<T>::value>::type 
    construct(T* p) 
{
    std::cout << "default constructing non-trivially default constructible T\n";
    ::new(detail::voidify(p)) T;
}
 
// #2
template<class T, class... Args>
std::enable_if_t<std::is_constructible<T, Args&&...>::value> // Using helper type
    construct(T* p, Args&&... args) 
{
    std::cout << "constructing T with operation\n";
    ::new(detail::voidify(p)) T(static_cast<Args&&>(args)...);
}
 
// #3, enabled via a parameter
template<class T>
void destroy(
    T*, 
    typename std::enable_if<
        std::is_trivially_destructible<T>::value
    >::type* = 0)
{
    std::cout << "destroying trivially destructible T\n";
}
 
// #4, enabled via a non-type template parameter
template<class T,
         typename std::enable_if<
             !std::is_trivially_destructible<T>{} &&
             (std::is_class<T>{} || std::is_union<T>{}),
             bool>::type = true>
void destroy(T* t)
{
    std::cout << "destroying non-trivially destructible T\n";
    t->~T();
}
 
// #5, enabled via a type template parameter
template<class T,
	 typename = std::enable_if_t<std::is_array<T>::value>>
void destroy(T* t) // note: function signature is unmodified
{
    for (std::size_t i = 0; i < std::extent<T>::value; ++i)
        destroy((*t)[i]);
}
 
/*
template<class T,
	 typename = std::enable_if_t<std::is_void<T>::value>>
void destroy(T* t) {} // error: has the same signature with #5
*/
 
// the partial specialization of A is enabled via a template parameter
template<class T, class Enable = void>
class A {}; // primary template
 
template<class T>
class A<T, typename std::enable_if<std::is_floating_point<T>::value>::type>
{}; // specialization for floating point types
 
int main()
{
    union { int i; char s[sizeof(std::string)]; } u;
 
    construct(reinterpret_cast<int*>(&u));
    destroy(reinterpret_cast<int*>(&u));
 
    construct(reinterpret_cast<std::string*>(&u), "Hello");
    destroy(reinterpret_cast<std::string*>(&u));
 
    A<int>{}; // OK: matches the primary template
    A<double>{}; // OK: matches the partial specialization
}
```

Output:

```plaintext
default constructing trivially default constructible T
destroying trivially destructible T
constructing T with operation
destroying non-trivially destructible T
```