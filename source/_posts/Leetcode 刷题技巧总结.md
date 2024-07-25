---
title: "LeetCode 刷题技巧总结"
category: CS&Maths
#id: 57
date: 2024-7-25 20:42:32
tags: 
  - LeetCode
  - C++
  - JAVA
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: code  # 展示在时间线列表中
---
记录一下常用的 LeetCode 刷题技巧。
<!--more-->

# C++


# JAVA

## String
在刷 LeetCode 时，Java 的`String`类有以下一些常用操作：

### 字符串拼接
使用`+`运算符可以方便地将多个字符串拼接在一起。
例如：
```java
String str1 = "Hello";
String str2 = "World";
String result = str1 + " " + str2; 
```

### 获取字符串长度
通过`length()`方法获取字符串的长度。
```java
String text = "This is a test";
int length = text.length(); 
```

### 字符提取
可以使用`charAt(int index)`方法获取指定位置的字符。
例如：
```java
String word = "Java";
char ch = word.charAt(2); 
```

### 子字符串获取
`substring(int beginIndex)`和`substring(int beginIndex, int endIndex)`方法用于获取子字符串。
```java
String sentence = "Hello Java World";
String sub1 = sentence.substring(6); 
String sub2 = sentence.substring(6, 10); 
```

### 字符串比较
可以使用`equals(Object anObject)`方法比较两个字符串的内容是否相等。
```java
String strA = "apple";
String strB = "apple";
if (strA.equals(strB)) { 
    // 执行相应操作
}
```

### 字符串转换
例如将字符串转换为字符数组，使用`toCharArray()`方法。
```java
String str = "example";
char[] charArray = str.toCharArray(); 
```

### 字符串拆分
例如将字符串拆分为字符串数组，使用`split(String regex)`或`split(String regex, int limit)`方法。

- `regex`：要作为分隔符的正则表达式。可以是单个字符、字符序列或更复杂的正则表达式模式。例如，使用`","`来按照逗号进行分割，使用`"\\."`来按照点进行分割（点在正则表达式中有特殊含义，所以需要转义）。
- `limit`（可选参数）：限制分割后子字符串数组的长度。
    - 当`limit`大于 0 时，字符串最多被分隔`limit - 1`次，得到长度为`limit`的数组。如果字符串包含`limit - 1`个分隔符，则从第`limit - 1`个分隔符开始到最后的字符串，将作为一个整体放在数组的最后。
    - 当`limit`等于 0 时，字符串会被尽可能多地分割。
    - 当`limit`小于 0 时，效果与不设置`limit`参数（或`limit`为 0）相同，即字符串也会被尽可能多地分割，不限制数组的长度。

以下是一些示例代码：

```java
public class SplitExample {
    public static void main(String[] args) {
        String str = "welcome-to-runoob";
        System.out.println("- 分隔符返回值:");
        for (String retval : str.split("-")) {
            System.out.println(retval); 
        }

        System.out.println("");
        System.out.println("- 分隔符设置分割份数返回值:");
        for (String retval : str.split("-", 2)) {
            System.out.println(retval); 
        }

        String str2 = "www.runoob.com";
        System.out.println("转义字符返回值:");
        for (String retval : str2.split("\\.")) { 
            System.out.println(retval); 
        }

        String str3 = "acount=?and uu=?or n=?";
        System.out.println("多个分隔符返回值:");
        for (String retval : str3.split("and|or")) { 
            System.out.println(retval); 
        }
    }
}
```

上述代码的输出结果为：

```
- 分隔符返回值:
welcome
to
runoob

- 分隔符设置分割份数返回值:
welcome
to-runoob

转义字符返回值:
www
runoob
com

多个分隔符返回值:
acount=?
uu=?
n=?
```

需要注意的是，某些特殊字符在作为分隔符时需要进行转义，因为它们在正则表达式中有特殊含义，例如`.*+^$|?()[]{}\`等。如果要使用这些字符作为分隔符，需要加上相应的转义字符，如`\\.`表示点，`\\|`表示竖线等。另外，如果分隔符连续出现，可能会产生空字符串作为分割后的子字符串。在实际使用中，要根据具体需求来处理这些情况。

## Collection
![](/Leetcode%20刷题技巧总结/java_collections_overview.png)

```mermaid
---
title: Collection
---
classDiagram
    Collection <|-- List
    Collection <|-- Set
    Collection <|-- Queue
    Collection <|-- Deque
    List <|-- ArrayList
    List <|-- Vector
    List <|-- CopyOnWriteArrayList
    List <|-- LinkedList
    Vector <|-- Stack
    Queue <|-- LinkedList
    Queue <|-- PriorityQueue
    Deque <|-- LinkedList
    Deque <|-- ArrayDeque
    Set <|-- HashSet
    Set <|-- LinkedHashSet
    Set <|-- TreeSet

    class ArrayList {
        // 用法
        - 适合随机访问和遍历，查询速度快。
        - 初始化时可指定容量，避免频繁扩容。
        // 注意事项
        - 增删元素时性能相对较差。
        - 非线程安全。
    }

    class LinkedList {
        // 用法
        - 适合频繁的插入和删除操作。
        - 可用于实现队列和栈。
        // 注意事项
        - 随机访问性能较差。
    }

    class Vector {
        // 用法
        - 线程安全的列表。
        - 适合多线程环境下使用。
        // 注意事项
        - 性能相对较低。
    }

    class Stack {
        // 用法
        - 遵循后进先出原则。
        // 注意事项
        - 已不被推荐使用，建议使用 Deque 实现栈功能。
    }

    class HashSet {
        // 用法
        - 快速查找元素是否存在。
        - 不保证元素的顺序。
        // 注意事项
        - 元素的 hashCode 和 equals 方法需正确实现。
    }

    class LinkedHashSet {
        // 用法
        - 元素有序，按照插入顺序。
        // 注意事项
        - 比 HashSet 性能略低。
    }

    class TreeSet {
        // 用法
        - 元素自动排序。
        - 可用于排序和去重。
        // 注意事项
        - 元素必须实现 Comparable 接口或提供比较器。
    }

    class PriorityQueue {
        // 用法
        - 按照元素的优先级取出元素。
        // 注意事项
        - 元素需要实现 Comparable 接口或提供比较器。
    }

    class ArrayDeque {
        // 用法
        - 双端队列，两端插入和删除效率高。
        // 注意事项
        - 不支持容量限制。
    }

    class CopyOnWriteArrayList {
        // 用法
        - 适用于读多写少的并发场景。
        // 注意事项
        - 内存消耗较大，写入时复制整个数组。
    }
```

```mermaid
---
title: Map
---
classDiagram
    Map <|-- HashMap
    Map <|-- LinkedHashMap
    Map <|-- TreeMap
    Map <|-- Hashtable
    Map <|-- ConcurrentHashMap
    Hashtable <|-- Properties
    
    class HashMap {
        // 用法
        - 常用的键值对存储结构。
        - 可通过调整初始容量和负载因子优化性能。
        // 注意事项
        - 非线程安全，多线程环境下可能出现问题。
        - 键的 hashCode 和 equals 方法需正确实现。
    }

    class LinkedHashMap {
        // 用法
        - 保持元素的插入顺序。
        // 注意事项
        - 性能略低于 HashMap。
    }

    class TreeMap {
        // 用法
        - 按照键的自然顺序或指定的比较器排序。
        // 注意事项
        - 键必须实现 Comparable 接口或提供比较器。
    }

    class Hashtable {
        // 用法
        - 线程安全的键值对存储。
        // 注意事项
        - 不允许键或值为 null，性能相对较低。
    }

    class Properties {
        // 用法
        - 常用于读取和写入配置文件。
        // 注意事项
        - 键和值都是字符串类型。
    }

    class ConcurrentHashMap {
        // 用法
        - 高并发环境下的键值对存储。
        // 注意事项
        - 部分操作可能不具备原子性。
    }
```