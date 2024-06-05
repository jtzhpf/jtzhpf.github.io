---
title: "SQL 刷题技巧总结"
category: CS&Maths
#id: 57
date: 2024-6-5 20:42:32
tags: 
  - SQL
toc: true
#sticky: 1 # 数字越大置顶优先级越高。数字都需要大于 0。
#cover: /images/about.jpg # 指定封面图片的 URL
timeline: code  # 展示在时间线列表中
---
记录一下常用的 SQL 操作。
<!--more-->

# 时间处理
## DATEDIFF
用于计算两者的日期差
```SQL
DATEDIFF(date1,date2)
```
`date1` 和 `date2` 参数是合法的日期或日期/时间表达式。

注释：只有值的日期部分参与计算。
```SQL
DATEDIFF('2007-12-31','2007-12-30');   # 1
DATEDIFF('2010-12-30','2010-12-31');   # -1
```


## DATE_ADD
给日期添加指定的时间间隔

```SQL
DATE_ADD(date,INTERVAL expr type)
```
`date` 参数是合法的日期表达式。`expr` 参数是您希望添加的时间间隔。
`type` 参数可以是下列值：
| type |
|--|
MICROSECOND
SECOND
MINUTE
HOUR
DAY
WEEK
MONTH
QUARTER
YEAR
SECOND_MICROSECOND
MINUTE_MICROSECOND
MINUTE_SECOND
HOUR_MICROSECOND
HOUR_SECOND
HOUR_MINUTE
DAY_MICROSECOND
DAY_SECOND
DAY_MINUTE
DAY_HOUR
YEAR_MONTH
```SQL
# 最小 event_date +3 天作为 second_date
SELECT DATE_ADD(MIN(event_date), INTERVAL 3 DAY) AS second_date 
```
## DATE_SUB
从日期减去指定的时间间隔

# 其他
## ROUND
对某个数值域进行指定小数位数的四舍五入
```SQL
ROUND(c,decimals)
```

## IF
```SQL
IF(condition, value_if_true, value_if_false)
```
```SQL
SELECT IF(500<1000, "YES", "NO");
```

## IFNULL
```SQL
IFNULL(expression, alt_value)
```
当 `expression` 为 NULL，返回 `alt_value`
```SQL
SELECT IFNULL(NULL, "W3Schools.com");
```

##
```SQL
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    WHEN conditionN THEN resultN
    ELSE result
END;
```