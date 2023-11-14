---
title: "Perl Learning Notes"
category: CS&Maths
#id: 57
date: 2023-9-24 09:00:00
tags: 
  - Perl
#toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
#timeline: article  # 展示在时间线列表中
---
<!--more-->

## debug 的方法
### 利用 perl 命令

```perl -d xxx.pl xxx.parameter```

> h 帮助
> n 下一步next，跳过sub子函数；
> s 单步调试，可以进入sub子函数；r 跳出子函数调试；
> p 打印表达式的结果，也可以显示变量的值，比如p $aaa；
> w 监视表达式。
> x 显示变量结果；比p支持的数据类型更多。
> V 支持正则表达式方法匹配变量。
> c line_num continue到line_num行
> b 行号；断点设置。B 行号；断点去除。L；查看断点。
> q 退出。
