---
title: "AddressSanitizer"
category: CS&Maths
#id: 57
date: 2024-1-3 09:00:00
tags: 
  - LLVM
  - Clang
  - GCC
  - Compiler
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: article  # 展示在时间线列表中
---
昨天晚上写了个LeetCode的第5题，最长回文子串，本地测试了发现结果是正确的之后就提交了，但提交之后却出了问题。

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>
char* longestPalindrome(char* s) {
    int ssize=strlen(s);
    bool (*vec)[ssize]=(bool*)malloc(sizeof(bool)*ssize*ssize);
    memset(vec,0,ssize*ssize);
    for(int i=0;i<ssize;i++){
        vec[i][i]=true;
    }
    int length=1;
    int pos=0;
    for (int L=2;L<=ssize;L++){
        for(int i=0;i+L-1<ssize;i++){
            if(L<3){
                vec[i][i+L-1]=s[i]==s[i+L-1];
            }else{
                vec[i][i+L-1]=s[i]==s[i+L-1]&&vec[i+1][i+L-2];
            }
            if(vec[i][i+L-1]){
                length=L;
                pos=i;
            }
        }
    }

    char *res=(char*)(malloc((length)*sizeof(char)));
    for(int i=0;i<length;i++)
        res[i]=s[pos+i];
    
    return res;
}
int main(){
    char s[]="aaaaa";
    char *res=longestPalindrome(s);
        printf("%s",res);
    free(res);
}
```
![ERROR: AddressSanitizer: heap-buffer-overflow](/AddressSanitizer/image1.png)

倒是新奇，查询一番得知这是因为LeetCode开启了AddressSanitizer[^1]，在编译时加上`-fsanitize=address`即可开启。开启后再执行就会报一样的错误了。


## 原理
由于是内存检测工具，其需要对每一次内存读写操作进行检查：
`*address = ...; // or: ... = *address;`

进行如下的逻辑判断：

```C
if (IsPoisoned(address)) {
  ReportError(address, kAccessSize, kIsWrite);
}
*address = ...;  // or: ... = *address;
```
如果指针读写异常，则统计及打印异常信息，可见整个工具的关键在于 `IsPoisoned` 如何实现，该函数需要快速而且准确。



### 内存映射

其将内存分为两块：

- 主内存：程序常规使用
- 影子内存：记录主内存是否可用等meta信息

如果有个函数 `MemToShadow` 可以根据主内存地址获取到对应的影子内存地址，那么内存检测的实现，可以改写为：
```C
shadow_address = MemToShadow(address);
if (ShadowIsPoisoned(shadow_address)) {
  ReportError(address, kAccessSize, kIsWrite);
}
```
#### 影子内存
AddressSanitizer 用 1 byte 的影子内存，记录主内存中 8 bytes 的数据。

为什么是 8 bytes ，因为malloc分配内存是按照 8 bytes 对齐。

这样，8 bytes 的主内存，共构成 9 种不同情况：

- 8 bytes 的数据可读写，影子内存中的value值为 **0**
- 8 bytes 的数据不可读写，影子内存中的value值为 **负数**
- 前 k bytes 可读写，后 (8 - k) bytes 不可读写，影子内存中的value值为 **k**

如果 `malloc(13)`，根据 8 bytes 字节对齐的原则，需要 2 bytes 的影子内存，第一个byte的值为 0，第二个byte的值为 5。

这时，整个判断流程，可改写为：
```C
byte *shadow_address = MemToShadow(address);
byte shadow_value = *shadow_address;
if (shadow_value) {
  if (SlowPathCheck(shadow_value, address, kAccessSize)) {
    ReportError(address, kAccessSize, kIsWrite);
  }
}

// Check the cases where we access first k bytes of the qword
// and these k bytes are unpoisoned.
bool SlowPathCheck(shadow_value, address, kAccessSize) {
  last_accessed_byte = (address & 7) + kAccessSize - 1;
  return (last_accessed_byte >= shadow_value);
}
```

### 例子

如何检测数组访问越界：
```C
void foo() {
  char a[8];
  ...
  return;
}
```
通过插桩(Instrumentation)和 hook 动态运行库(Run-time library)，
`AddressSanitizer` 将其改写为：
```C
void foo() {
  char redzone1[32];  // 32-byte aligned
  char a[8];          // 32-byte aligned
  char redzone2[24];
  char redzone3[32];  // 32-byte aligned
  int  *shadow_base = MemToShadow(redzone1);
  shadow_base[0] = 0xffffffff;  // poison redzone1
  shadow_base[1] = 0xffffff00;  // poison redzone2, unpoison 'a'
  shadow_base[2] = 0xffffffff;  // poison redzone3
  ...
  shadow_base[0] = shadow_base[1] = shadow_base[2] = 0; // unpoison all
  return;
}
```

插桩主要是对访问内存的操作(store，load，alloc等)，将它们进行处理。
hook动态运行库主要提供将malloc，free等系统调用函数hook住。

该算法的思路是：如果想防住Buffer Overflow漏洞，只需要在每块内存区域两端加一块区域（RedZone），使RedZone的区域的影子内存（Shadow Memory）设置为不可写即可。


hook malloc/free函数。在malloc函数中额外的分配了Redzone区域的内存，将与Redzone区域对应的影子内存加锁，主要的内存区域对应的影子内存不加锁。
free函数将所有分配的内存区域加锁，并放到了隔离区域的队列中(保证在一定的时间内不会再被malloc函数分配)，可检测Use after free类的问题。

[^1]: [https://github.com/google/sanitizers/wiki/AddressSanitizerAlgorithm](https://github.com/google/sanitizers/wiki/AddressSanitizerAlgorithm)