---
title: "LeetCode 刷题技巧总结 - JAVA篇"
category: CS&Maths
#id: 57
date: 2024-7-25 20:42:32
tags: 
  - LeetCode
  - JAVA
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: code  # 展示在时间线列表中
---
记录一下常用的 LeetCode 刷题技巧（JAVA篇）。
<!--more-->

# 语法
## 字符串操作
### String
在刷 LeetCode 时，Java 的`String`类有以下一些常用操作：

#### 字符串拼接
使用`+`运算符可以方便地将多个字符串拼接在一起。
例如：
```java
String str1 = "Hello";
String str2 = "World";
String result = str1 + " " + str2; 
```

#### 获取字符串长度
通过`length()`方法获取字符串的长度。
```java
String text = "This is a test";
int length = text.length(); 
```

#### 字符提取
可以使用`charAt(int index)`方法获取指定位置的字符。
例如：
```java
String word = "Java";
char ch = word.charAt(2); 
```

#### 子字符串获取
`substring(int beginIndex)`和`substring(int beginIndex, int endIndex)`方法用于获取子字符串。
```java
String sentence = "Hello Java World";
String sub1 = sentence.substring(6); 
String sub2 = sentence.substring(6, 10); 
```

#### 字符串比较
可以使用`equals(Object anObject)`方法比较两个字符串的内容是否相等。
```java
String strA = "apple";
String strB = "apple";
if (strA.equals(strB)) { 
    // 执行相应操作
}
```

#### 字符串转换
例如将字符串转换为字符数组，使用`toCharArray()`方法。
```java
String str = "example";
char[] charArray = str.toCharArray(); 
```

反向转换：
1. 直接在构造String时建立。 

```java
char c[] = {'s', 'g', 'k'}; 
String str = new String(c);
```

2. String 有 valueOf() 方法可以直接转换。 

```java
char[] c = {'s','g','h'}; 
String str = String.valueOf(c);
```

#### 字符串拆分
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

#### 字符串合并
```java
import java.util.Arrays;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("apple", "banana", "cherry");
        String result = String.join(", ", list);
        System.out.println(result); // Outputs: apple, banana, cherry
    }
}
```

#### 删除前后空格
```java
public class Main {
    public static void main(String[] args) {
        String str = "  Hello, World!  ";
        String trimmedStr = str.trim();
        System.out.println(trimmedStr); // Outputs: "Hello, World!"
    }
}

```

### StringBuilder(非线程安全) & StringBuffer(线程安全)
在刷 LeetCode 时，`StringBuilder`和`StringBuffer`常用于高效地构建和修改字符串。

#### 初始化
```java
StringBuilder sb = new StringBuilder();
StringBuffer sf = new StringBuffer();
```

#### 添加内容
可以使用`append`方法添加各种类型的数据，如字符串、整数、浮点数等。
```java
append(boolean b)
append(char c)
append(char[] str)
append(char[] str, int offset, int len)
append(CharSequence s)
append(CharSequence s, int start, int end)
append(double d)
append(float f)
append(int i)
append(long lng)
append(Object obj)
append(String str)
append(StringBuffer sb)
```

```java
sb.append("Hello");
sb.append(123);
sf.append(3.14);
```

#### 删除部分内容
通过`delete`方法删除指定范围内的字符。
```java
sb.delete(0, 5);  // 删除从索引 0 到 4（包括 4）的字符
```

#### 插入内容
使用`insert`方法在指定位置插入数据。
```java
sb.insert(2, "World");  // 在索引 2 处插入 "World"
```

#### 替换内容
借助`replace`方法替换指定范围内的字符。
```java
sb.replace(1, 3, "Java");  // 替换索引 1 到 2 的字符为 "Java"
```

#### 反转字符串
利用`reverse`方法将字符串反转。
```java
sb.reverse();
```

#### 获取字符串
通过`toString`方法将`StringBuilder`或`StringBuffer`对象转换为`String`类型。
```java
String result = sb.toString();
```

以下是 `StringBuilder` 和 `StringBuffer` 在刷 LeetCode 中可能会用到的一些其他操作：

#### 清空内容
使用 `setLength(0)` 方法可以清空 `StringBuilder` 或 `StringBuffer` 的内容。

#### 获取指定位置的字符
可以使用 `charAt(int index)` 方法获取指定索引位置的字符。

### 设置指定位置的字符
通过 `setCharAt(int index, char ch)` 方法来设置指定索引位置的字符。

#### 获取容量
`capacity()` 方法能获取当前对象的内部字符数组的容量。

#### 获取字符串长度
```java
StringBuilder sb = new StringBuilder("Java");
int length = sb.length();
```
```java
StringBuffer sbuffer = new StringBuffer("Python");
int length = sbuffer.length();
```

#### 截取子字符串
```java
StringBuilder sb = new StringBuilder("HelloWorld");
String sub = sb.substring(5); 
```
```java
StringBuffer sbuffer = new StringBuffer("JavaLanguage");
String sub = sbuffer.substring(4, 8);
```


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
### Queue
#### PriorityQueue
在Java中，`Queue` 是一个接口，而 `PriorityQueue` 是该接口的一个具体实现。

   - **`PriorityQueue`** 是 `Queue` 的一个实现类，它是一个基于优先级堆的队列。`PriorityQueue` 保证每次访问或删除的元素是队列中优先级最高的元素（根据自然顺序或提供的比较器进行排序）。

   - **`PriorityQueue`** 按元素的优先级排序，而不是插入顺序。最小的元素（根据自然顺序或比较器决定）会首先被处理。例如，数字1优先于2，字符串"A"优先于"B"。

   - **`PriorityQueue`** 底层使用的是一个动态数组，并且以二叉堆（Binary Heap）的形式存储数据。它保证在插入或删除元素时能够保持堆的结构。

   - **`PriorityQueue`** 适用于需要按优先级处理元素的场景，常用于任务的优先级调度、处理基于优先级的事件等。

在LeetCode刷题时，`PriorityQueue` 是一个非常常用的工具，尤其在处理需要动态获取最小值或最大值的问题时（如Top K问题、合并K个有序链表、滑动窗口最大值等）。`PriorityQueue` 提供了很多便捷的方法，以下是常用的操作和一些典型用法：

Java中的 `PriorityQueue` 默认是**最小堆**，也就是每次 `poll()` 或 `peek()` 时返回的是当前队列中最小的元素。假设需要在最小堆的基础上实现**最大堆**，可以传递一个自定义的比较器。


以下是一些 LeetCode 经典题目中使用 `PriorityQueue` 的场景：

1. **合并K个有序链表**
   - 题目: [LeetCode 23 - 合并K个排序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)
   - 解决思路: 使用 `PriorityQueue` 来存储每个链表的头结点，按节点值进行排序（默认是最小堆）。每次从队列中取出最小的节点，然后将其连接到结果链表，同时将该节点的下一个节点加入队列。
   
   ```java
   public ListNode mergeKLists(ListNode[] lists) {
       PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> (a.val - b.val));
       for (ListNode list : lists) {
           if (list != null) pq.offer(list);
       }
       ListNode dummy = new ListNode(0);
       ListNode curr = dummy;
       while (!pq.isEmpty()) {
           ListNode node = pq.poll();
           curr.next = node;
           curr = curr.next;
           if (node.next != null) pq.offer(node.next);
       }
       return dummy.next;
   }
   ```

2. **寻找数组中的第K大元素**
   - 题目: [LeetCode 215 - 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)
   - 解决思路: 可以用一个大小为 K 的**最小堆**来维护当前最大的 K 个元素。当堆中的元素超过 K 个时，移除最小的元素，最后堆顶元素就是第 K 大的元素。

   ```java
   public int findKthLargest(int[] nums, int k) {
       PriorityQueue<Integer> pq = new PriorityQueue<>();
       for (int num : nums) {
           pq.offer(num);
           if (pq.size() > k) {
               pq.poll();
           }
       }
       return pq.peek();
   }
   ```

3. **滑动窗口最大值**
   - 题目: [LeetCode 239 - 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)
   - 解决思路: 你可以使用最大堆（优先队列）来跟踪滑动窗口中的最大值。对于每个滑动窗口，向堆中添加新元素，并移除已超出窗口范围的元素。

   ```java
   public int[] maxSlidingWindow(int[] nums, int k) {
       PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> b[0] - a[0]); // 最大堆
       int n = nums.length;
       int[] res = new int[n - k + 1];
       for (int i = 0; i < n; i++) {
           pq.offer(new int[]{nums[i], i});
           if (i >= k - 1) {
               while (pq.peek()[1] <= i - k) { // 移除不在窗口中的元素
                   pq.poll();
               }
               res[i - k + 1] = pq.peek()[0]; // 当前窗口的最大值
           }
       }
       return res;
   }
   ```

4. **前K个高频元素**
   - 题目: [LeetCode 347 - 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)
   - 解决思路: 使用**最小堆**存储频率前K高的元素。遍历所有元素及其频率，当堆中的元素数量超过K时，移除堆顶元素，最终堆中的元素就是频率前K高的元素。

   ```java
   public int[] topKFrequent(int[] nums, int k) {
       Map<Integer, Integer> freqMap = new HashMap<>();
       for (int num : nums) {
           freqMap.put(num, freqMap.getOrDefault(num, 0) + 1);
       }

       PriorityQueue<Map.Entry<Integer, Integer>> pq = new PriorityQueue<>((a, b) -> a.getValue() - b.getValue());

       for (Map.Entry<Integer, Integer> entry : freqMap.entrySet()) {
           pq.offer(entry);
           if (pq.size() > k) {
               pq.poll();
           }
       }

       int[] result = new int[k];
       for (int i = 0; i < k; i++) {
           result[i] = pq.poll().getKey();
       }
       return result;
   }
   ```

4. **自定义比较器（最大堆）**
默认情况下，`PriorityQueue` 是最小堆。如果需要实现**最大堆**，可以传递一个自定义的比较器。例如：
```java
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);
```
这种方式在解决涉及最大值的问题时非常有用。



### Deque
**双端队列（Deque）**

双端队列是一种支持在两端进行元素插入和删除的线性集合。“Deque”这个名称是“double ended queue”（双端队列）的缩写，通常发音为“deck”。大多数双端队列的实现对其可能包含的元素数量没有固定限制，但这个接口也支持容量受限的双端队列以及没有固定大小限制的双端队列。

这个接口定义了访问双端队列两端元素的方法。提供了插入、删除和检查元素的方法。每种方法都有两种形式：一种在操作失败时抛出异常，另一种返回一个特殊值（根据操作，可能为 `null` 或 `false`）。插入操作的后一种形式专门用于容量受限的双端队列实现；在大多数实现中，插入操作不会失败。

上述十二种方法总结在以下表格中：

**双端队列方法总结**

|  第一个元素（头部）抛出异常  |  第一个元素（头部）特殊值  |  最后一个元素（尾部）抛出异常  |  最后一个元素（尾部）特殊值  |
| :---: | :---: | :---: | :---: |
| `addFirst(e)`  | `offerFirst(e)`  | `addLast(e)`  | `offerLast(e)`  |
| `removeFirst()`  | `pollFirst()`  | `removeLast()`  | `pollLast()`  |
| `getFirst()`  | `peekFirst()`  | `getLast()`  | `peekLast()`  |

这个接口扩展了 `Queue` 接口。

当双端队列用作队列时，会产生先进先出（FIFO）的行为。元素在双端队列的末尾添加，并从开头移除。从 `Queue` 接口继承的方法与双端队列方法的精确对等关系如下表所示：

**队列和双端队列方法的比较**

|  队列方法  |  等效的双端队列方法  |
| :---: | :---: |
| `add(e)`  | `addLast(e)`  |
| `offer(e)`  | `offerLast(e)`  |
| `remove()`  | `removeFirst()`  |
| `poll()`  | `pollFirst()`  |
| `element()`  | `getFirst()`  |
| `peek()`  | `peekFirst()`  |

双端队列也可以用作后进先出（LIFO）栈。应该优先使用这个接口而不是遗留的 `Stack` 类。当双端队列用作栈时，元素在双端队列的开头进行压入和弹出。栈方法与双端队列方法的精确对等关系如下表所示：

**栈和双端队列方法的比较**

|  栈方法  |  等效的双端队列方法  |
| :---: | :---: |
| `push(e)`  | `addFirst(e)`  |
| `pop()`  | `removeFirst()`  |
| `peek()`  | `peekFirst()`  |

请注意，当双端队列用作队列或栈时，`peek` 方法都能很好地工作；在任何一种情况下，元素都是从双端队列的开头获取的。

这个接口提供了两个方法来删除内部元素，`removeFirstOccurrence` 和 `removeLastOccurrence` 。

与 `List` 接口不同，这个接口不提供对元素的基于索引的访问支持。

虽然双端队列的实现并非严格禁止插入 `null` 元素，但强烈建议不要这样做。任何允许插入 `null` 元素的双端队列实现的用户，都强烈建议不要利用插入 `null` 的能力。这是因为 `null` 被各种方法用作特殊的返回值来表示双端队列为空。

双端队列的实现通常不会定义基于元素的 `equals` 和 `hashCode` 方法，而是从 `Object` 类继承基于标识的版本。

### List
在刷 LeetCode 时，Java 的`List`接口（通常使用`ArrayList`或`LinkedList`实现）有以下一些常用操作：

#### 初始化
```java
List<Integer> list = new ArrayList<>();     // 或者 new LinkedList<>()
List<Integer> list = new ArrayList<>(10);   // 无法初始化size, 只是初始化了capacity
```

#### 添加元素
- `add(E element)`：在列表末尾添加元素。
```java
list.add(1);
```
- `add(int index, E element)`：在指定索引位置添加元素。
```java
list.add(1, 2);
```

#### 获取元素
- `get(int index)`：获取指定索引位置的元素。
```java
int element = list.get(0);
```

#### 删除元素
- `remove(int index)`：删除指定索引位置的元素。
```java
list.remove(0);
```
- `remove(Object o)`：删除指定的元素。
```java
list.remove(Integer.valueOf(1));
```

#### 修改元素
- `set(int index, E element)`：将指定索引位置的元素修改为给定元素。
```java
list.set(0, 3);
```

#### 获取列表大小
- `size()`：获取列表中元素的个数。
```java
int size = list.size();
```

#### 判断是否包含元素
- `contains(Object o)`：判断列表是否包含指定元素。
```java
boolean contains = list.contains(2);
```

#### 清空列表
- `clear()`：清空列表中的所有元素。
```java
list.clear();
```

#### 遍历列表
- 使用`for`循环：
```java
for (int i = 0; i < list.size(); i++) {
    int element = list.get(i);
    // 处理元素
}
```
- 使用增强`for`循环：
```java
for (int element : list) {
    // 处理元素
}
```
- 使用迭代器`Iterator`：
```java
Iterator<Integer> iterator = list.iterator();
while (iterator.hasNext()) {
    int element = iterator.next();
    // 处理元素
}
```


### Set
在刷 LeetCode 时，Java 的`Set`接口（常见的实现类如`HashSet`和`TreeSet`）有以下常用操作：

#### 初始化
```java
Set<Integer> set = new HashSet<>();  // 或者 new TreeSet<>()
```

#### 添加元素
```java
set.add(1);
```

#### 判断元素是否存在
```java
boolean exists = set.contains(1);
```

#### 删除元素
```java
set.remove(1);
```

#### 获取集合大小
```java
int size = set.size();
```

#### 清空集合
```java
set.clear();
```

#### 遍历集合
- 使用增强`for`循环：
```java
for (int element : set) {
    // 处理元素
}
```

#### 集合运算
- 求交集：
```java
Set<Integer> set1 = new HashSet<>();
Set<Integer> set2 = new HashSet<>();
// 求交集并存入新的集合
Set<Integer> intersection = new HashSet<>(set1);
intersection.retainAll(set2);
```
- 求并集：
```java
Set<Integer> union = new HashSet<>(set1);
union.addAll(set2);
```
- 求差集：
```java
Set<Integer> difference = new HashSet<>(set1);
difference.removeAll(set2);
```

### Map
在刷 LeetCode 时，Java 的`Map`接口（常见的实现类如`HashMap`和`TreeMap`）有以下常用操作：

#### 初始化
```java
Map<String, Integer> map = new HashMap<>();  // 或者 new TreeMap<>()
```

#### 添加键值对
```java
map.put("key1", 10);
```

#### 获取值
```java
int value = map.get("key1");
int value = map.getOrDefault("key1", "key2");
```

#### 判断键是否存在
```java
boolean containsKey = map.containsKey("key1");
```

#### 删除键值对
```java
map.remove("key1");
```

#### 获取键集合
```java
Set<String> keys = map.keySet();
```

#### 获取值集合
```java
Collection<Integer> values = map.values();
```

#### 判断是否为空
```java
boolean isEmpty = map.isEmpty();
```

#### 清空
```java
map.clear();
```

#### 遍历
- 通过键遍历：
```java
for (String key : map.keySet()) {
    int value = map.get(key);
    // 处理键值对
}
```
- 通过键值对遍历：
```java
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    String key = entry.getKey();
    int value = entry.getValue();
    // 处理键值对
}
```

#### computeIfAbsent 方法

`computeIfAbsent` 方法是 Java 8 在 `java.util.Map` 接口中引入的一个默认方法。它主要用于简化在 Map 中根据键获取值的代码逻辑。如果指定的键在 Map 中不存在，它将计算指定键的值并将其放入 Map 中。

方法签名如下：
```java
default V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction)
```

参数说明：

- `key`：Map 中的键。
- `mappingFunction`：一个函数，接受一个键并返回一个值。


如果键存在，则返回当前映射的值；如果键不存在，则返回计算后的值。

**使用场景：**
`computeIfAbsent` 方法通常用于避免重复的检查和插入操作，简化代码，提升可读性。例如，当你需要从 Map 中获取某个值，如果该值不存在，则计算并插入新的值时，这个方法非常有用。

**示例：**

假设我们有一个 Map 用来存储学生的成绩，当我们需要获取一个学生的成绩时，如果该学生的成绩不存在，我们会默认给他一个分数：

```java
import java.util.HashMap;
import java.util.Map;

public class ComputeIfAbsentExample {
    public static void main(String[] args) {
        Map<String, Integer> studentScores = new HashMap<>();
        
        // 给定的默认分数计算函数
        studentScores.put("Alice", 85);

        // 使用computeIfAbsent方法，如果“Bob”不存在于Map中，则添加并返回默认分数70
        Integer score = studentScores.computeIfAbsent("Bob", k -> 70);
        System.out.println("Bob's score: " + score); // 输出：Bob's score: 70

        // 再次获取Bob的分数，这次直接从Map中获取
        score = studentScores.get("Bob");
        System.out.println("Bob's score: " + score); // 输出：Bob's score: 70
    }
}
```

#### entrySet 方法

在 Java 中，`entrySet` 方法是 `java.util.Map` 接口的一部分。它返回该 Map 中所有键值对 (Map.Entry) 的集合视图。通过 `entrySet`，你可以遍历 Map 的所有键值对，并对其进行操作。

**方法签名：**

```java
Set<Map.Entry<K, V>> entrySet()
```

**使用场景：**

`entrySet` 通常用于遍历 Map 或者在 Map 的键值对上进行批量操作。

**示例：**

以下示例展示了如何使用 `entrySet` 遍历一个 Map 并打印出所有的键值对：

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class EntrySetExample {
    public static void main(String[] args) {
        Map<String, Integer> studentScores = new HashMap<>();
        studentScores.put("Alice", 85);
        studentScores.put("Bob", 70);
        studentScores.put("Charlie", 90);

        // 获取Map的entrySet视图
        Set<Map.Entry<String, Integer>> entries = studentScores.entrySet();

        // 遍历Map的所有键值对
        for (Map.Entry<String, Integer> entry : entries) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }
    }
}
```

**常见操作：**

1. **遍历 Map：**

   使用 `entrySet` 遍历 Map 时，每个元素都是一个 `Map.Entry` 对象，你可以通过 `entry.getKey()` 和 `entry.getValue()` 来获取键和值。

2. **修改 Map：**

   在遍历时，可以直接通过 `Map.Entry` 对象修改 Map 中的值。例如：

   ```java
   for (Map.Entry<String, Integer> entry : entries) {
       if (entry.getKey().equals("Alice")) {
           entry.setValue(95); // 修改Alice的分数
       }
   }
   ```

3. **删除元素：**

   使用 `Iterator` 时，可以在遍历过程中安全地删除元素：

   ```java
   Iterator<Map.Entry<String, Integer>> iterator = entries.iterator();
   while (iterator.hasNext()) {
       Map.Entry<String, Integer> entry = iterator.next();
       if (entry.getKey().equals("Bob")) {
           iterator.remove(); // 删除Bob的记录
       }
   }
   ```

#### Pair
java 默认不提供 Pair，需要我们自行提供。

```java
class Solution {
    // Simple Pair class as JavaFX Pair is not available by default
    static class Pair<K, V> {
        private K key;
        private V value;

        public Pair(K key, V value) {
            this.key = key;
            this.value = value;
        }

        public K getKey() {
            return key;
        }

        public V getValue() {
            return value;
        }
    }
    ...
}
```

### Collection 嵌套

```java
Map<Integer, List<Integer>> map = new HashMap<Integer, ArrayList<Integer>>();   // 错误
Map<Integer, List<Integer>> map = new HashMap<Integer, List<Integer>>();        // 正确
map.add(new ArrayList<Integer>());
```
# 技巧

## 基础类型转换
在 Java 中，刷 LeetCode 时常用的不同类型之间互相转换的方法有以下几种：

**整数与字符串之间的转换**：
- `Integer.toString(int)` ：将整数转换为字符串。例如：`Integer.toString(123)` 将返回字符串 `"123"` 。
- `Integer.parseInt(String)` ：将字符串转换为整数。例如：`Integer.parseInt("123")` 将返回整数 `123` 。

**字符与整数之间的转换**：
- `(int)char` ：将字符转换为对应的 ASCII 码值。例如：`(int)'A'` 将返回 `65` 。
- `char(int)` ：将整数（代表 ASCII 码值）转换为对应的字符。例如：`char(65)` 将返回字符 `'A'` 。

**浮点数与字符串之间的转换**：
- `Double.toString(double)` ：将双精度浮点数转换为字符串。例如：`Double.toString(3.14)` 。
- `Double.parseDouble(String)` ：将字符串转换为双精度浮点数。例如：`Double.parseDouble("3.14")` 。

**字符串与其他基本数据类型的转换**：
- 除了上述提到的整数和浮点数，对于其他基本数据类型（如 `short` 、 `long` 、 `float` 等），也有相应的 `parseXXX` 和 `toString` 方法进行转换。

## List 和数组之间的相互转换

**数组转 `List`**：

使用 `Arrays.asList()` 方法，但需要注意它返回的 `List` 是不可变的。
```java
int[] arr = {1, 2, 3};
List<Integer> list = Arrays.asList(1, 2, 3);
```

如果要得到一个可变的 `List` ，可以手动遍历数组添加元素到新的 `List` 中。
```java
int[] arr = {1, 2, 3};
List<Integer> list = new ArrayList<>();
for (int num : arr) {
    list.add(num);
}
```

**`List` 转数组**：

使用 `toArray()` 方法。
```java
List<Integer> list = Arrays.asList(1, 2, 3);
Integer[] arr = list.toArray(new Integer[0]);
```

或者指定数组的长度。
```java
List<Integer> list = Arrays.asList(1, 2, 3);
Integer[] arr = list.toArray(new Integer[list.size()]);
```

## String 和 char 数组之间的相互转换

在 Java 中，`String` 和 `char` 数组之间的相互转换有多种方法，以下为您详细介绍：

**`String` 转换为 `char` 数组**：
```java
String str = "Hello";
char[] charArray = str.toCharArray();
```
例如，如果 `str` 的值为 `"World"` ，转换后的 `charArray` 就是 `['W', 'o', 'r', 'l', 'd']` 。

**`char` 数组转换为 `String`**：
```java
char[] charArray = {'H', 'e', 'l', 'l', 'o'};
String str = new String(charArray);
```
再比如，`charArray` 为 `['J', 'a', 'v', 'a']` ，转换得到的 `str` 就是 `"Java"` 。

还可以通过 `StringBuilder` 或 `StringBuffer` 来实现转换：
```java
char[] charArray = {'H', 'e', 'l', 'l', 'o'};
StringBuilder stringBuilder = new StringBuilder();
for (char c : charArray) {
    stringBuilder.append(c);
}
String str = stringBuilder.toString();
```

## 基本数据类型传引用

基本数据类型无法作为引用形式传递，但是可以通过只有一个元素的数组来实现传引用。

```java
// n皇后
class Solution {
    boolean check(int i, int j, int n, int[][] t){
        for(int k=0; k<n; k++){
            if(t[i][k]==1||t[k][j]==1){
                return false;
            }
        }
        for (int k = 0; k < n; k++) {
            for (int l = 0; l < n; l++) {
                if ((i - j == k - l || i + j == k + l) && t[k][l] == 1) {
                    return false;
                }
            }
        }
        return true;
    }
    void totalNQueens(int i, int n, int[][] t, int[] res){
        if(i==n){
            res[0]++;
        }else{
            for(int j=0; j<n; j++){
                if(check(i,j,n,t)){
                    t[i][j]=1;
                    totalNQueens(i+1, n, t, res);
                    t[i][j]=0;
                }
            }
        }
    }

    public int totalNQueens(int n) {
        int[] res={0};
        int[][] t = new int[n][n];
        totalNQueens(0, n, t, res);
        return res[0];
    }
}
```

## 取随机数
在 Java 中，你可以使用多种方法来生成随机数。以下是几种常用的方法：

1. **使用 `Math.random()`**:
   ```java
   double randomValue = Math.random(); // 生成一个 [0.0, 1.0) 之间的随机数
   ```

   如果你需要生成一个特定范围的随机整数，可以这样做：
   ```java
   int min = 1;
   int max = 10;
   int randomInt = min + (int)(Math.random() * (max - min + 1));
   ```

2. **使用 `java.util.Random` 类**:
   ```java
   import java.util.Random;

   Random random = new Random();
   int randomInt = random.nextInt(10); // 生成 [0, 10) 之间的随机整数
   ```

   你还可以生成指定范围内的随机整数：
   ```java
   int min = 1;
   int max = 10;
   int randomIntInRange = min + random.nextInt(max - min + 1);
   ```

3. **使用 `java.util.concurrent.ThreadLocalRandom`** (适用于并发环境):
   ```java
   import java.util.concurrent.ThreadLocalRandom;

   int randomInt = ThreadLocalRandom.current().nextInt(10); // 生成 [0, 10) 之间的随机整数
   ```

   生成指定范围内的随机整数：
   ```java
   int min = 1;
   int max = 10;
   int randomIntInRange = ThreadLocalRandom.current().nextInt(min, max + 1);
   ```

## 获得某一数据类型的最大值/最小值
```java
Integer.MAX_VALUE
Integer.MIN_VALUE
```

# 常用算法
## 快速排序
经过测试，发现与pivot比较判等对性能影响极大。

方案一：前后需要swap，先交换再处理后交换，用时35ms
```java
class Solution {
    public int[] sortArray(int[] nums) {
        quicksort(nums,0,nums.length-1);
        return nums;
    }
    void swap(int[] nums, int i, int j){
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    void quicksort(int[] nums, int left, int right){
        if(left<right){
            int random = new Random().nextInt(right+1-left)+left;
            swap(nums,left,random);
            int pivot=nums[left];
            int l=left+1, r=right;
            while(l<=r){
                while (l<=r && nums[l] < pivot)l++;
                while (l<=r && nums[r] > pivot)r--;
                if (l<=r) swap(nums, l++, r--);
            }
            swap(nums,left,r);
            quicksort(nums, left, r);
            quicksort(nums, r+1, right);
        }
    }
}
```

方案二：前后无需swap，利用do-while，用时50ms
```java
class Solution {
    public int[] sortArray(int[] nums) {
        quicksort(nums,0,nums.length-1);
        return nums;
    }
    void swap(int[] nums, int i, int j){
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    void quicksort(int[] nums, int left, int right){
        if(left>=right) return;
        int random = new Random().nextInt(right-left)+left;
        int pivot = nums[random];
        int l = left-1;
        int r = right+1;
        while(l<r){
            do l++;while(nums[l]<pivot);
            do r--;while(nums[r]>pivot);
            if(l<r)swap(nums,l,r);
        }
        quicksort(nums,left,r);
        quicksort(nums,r+1,right);
    }
}
```

方案三：严蔚敏版，交错替换，与pivot比较判等，用时1800ms
```java
class Solution {
    public int[] sortArray(int[] nums) {
        quicksort(nums,0,nums.length-1);
        return nums;
    }
    void swap(int[] nums, int i, int j){
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    void quicksort(int[] nums, int left, int right){
        if(left<right){
            int random = new Random().nextInt(right-left+1)+left;
            swap(nums, random, left);
            int l=left,r=right;
            int pivot = nums[l];
            while(l < r){
                while(l < r && nums[r]>=pivot)r--;
                nums[l] = nums[r];
                while(l < r && nums[l]<=pivot)l++;
                nums[r] = nums[l];
            }
            nums[l] = pivot;
            quicksort(nums,left,l);
            quicksort(nums,l+1,right);
        }
    }
}
```

方案四：快慢指针，用时2500ms
```java
class Solution {
    public int[] sortArray(int[] nums) {
        quicksort(nums,0,nums.length-1);
        return nums;
    }
    void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    void quicksort(int[] nums, int left, int right){
        if(left<right){
            int random = new Random().nextInt(right - left + 1) + left;
            swap(nums, left, random);

            int pivot = left;
            int index = pivot+1;
            for (int i = index; i <= right; i++) {
                if (nums[i] < nums[pivot]) {
                    swap(nums, i, index);
                    index++;
                }
            }
            swap(nums, pivot, index-1);
            quicksort(nums,left,index-2);
            quicksort(nums,index,right);
        }
    }
}
```