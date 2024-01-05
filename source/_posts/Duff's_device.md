---
title: "Duff's device"
category: CS&Maths
#id: 57
date: 2024-1-5 09:00:00
tags: 
  - C/C++
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---
达夫设备（英文：Duff's device）是串行复制（serial copy）的一种优化实现。

<!--more-->

```C
send(to, from, count)
register short *to, *from;
register count;
{
    register n = (count + 7) / 8;   /* 假定了count > 0 */
    switch (count % 8) {
        case 0:	do { *to = *from++;
        case 7:	     *to = *from++;
        case 6:	     *to = *from++;
        case 5:	     *to = *from++;
        case 4:	     *to = *from++;
        case 3:      *to = *from++;
        case 2:      *to = *from++;
        case 1:      *to = *from++;
                } while (--n > 0);
    }
}
```
第一次看到这玩意儿我是懵逼的。

**这也能运行！？？？**

![](/emoji/confused_cat.png)

这段代码的功能是将数组元素复制进存储器映射输出寄存器中。
其实上面这个代码等价于下面这个：

```C
send(to, from, count)
register short *to, *from;
register count;
{
    do {                /* 假定了count > 0 */
        *to = *from++;    
    } while (--count > 0);
}
```

由于每复制一次都要执行一次判断，极大的浪费了时间。

因此可以展开：
```C
send(to, from, count)
register short *to, *from;
register count;
{
    register n = count % 8;
    while (n-- > 0) {
        *to = *from++;
    }
    n = count / 8;
    if (n == 0) return;     
    do {
        *to = *from++;
        *to = *from++;
        *to = *from++;
        *to = *from++;
        *to = *from++;
        *to = *from++;
        *to = *from++;
        *to = *from++;
    } while (--n > 0)
}
```

可以看到，这样子还得处理非8的倍数的情况。此时我们再看达夫设备的本体，其使用 switch - case 来实现了处理不能被8整除的逻辑，同时在 switch - case 中嵌入了一个 do - while 循环来实现主逻辑，再借助 switch - case 的 fall through 特性，即case语句后面如果不加 break, 则程序会依次执行下去而不是退出 switch 语句块。最后又巧妙地使用了 case 语句不会改变控制流，即 case 之后遇到 while 会再次进入 while 的循环控制流中的特点，完美地实现了达夫设备本身！

达夫设备亦有其局限性。在某些环境下，利用switch/case语句处理零余数据项，有时并非最优选择；相对应的，若采用双循环机制可能反倒更快（实现一个展开后的循环复制8*n个数据项，和另一循环复制零余数据项）。究其肇因，则常是由于编译器无法针对达夫设备进行优化，但亦可能是因某些架构的流水线与分支预测机制有所差异。

为啥叫Duff's device呢，因为是当时供职于卢卡斯影业的汤姆·达夫于1983年11月发明的，这可能是迄今为止利用C语言switch语句特性所作的最巧妙的实现，所以就叫Duff's device了。