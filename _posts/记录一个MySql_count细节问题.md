---
layout:     post
title:      "记录一个MySql count细节问题"
category:   Mysql
date:       2019-01-01 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - MySql
---

![](https://img.hacpai.com/bing/20180418.jpg?imageView2/1/w/960/h/520/interlace/1/q/100)

>下面两条SQL查询结果会不一样吗？

```
select count(1) from 
  (select distinct `handler_name`, `retry_info`, `is_disabled` from handler) as alia;

select count(distinct `handler_name`, `retry_info`, `is_disabled`) from handler;
```

### count()语法

（1）count(*)---包括所有列，返回表中的记录数，相当于统计表的行数，在统计结果的时候，**不会忽略列值为NULL的记录。**

（2）count(1)---忽略所有列，1表示一个固定值，也可以用count(2)、count(3)代替，在统计结果的时候，**不会忽略列值为NULL的记录。**

（3）count(列名)---只包括列名指定列，返回指定列的记录数，在统计结果的时候，会忽略列值为NULL的记录（不包括空字符串和0），**即列值为NULL的记录不统计在内。**

（4）count(distinct 列名)---只包括列名指定列，返回指定列的不同值的记录数，在统计结果的时候，在统计结果的时候，会忽略列值为NULL的记录（不包括空字符串和0），**即列值为NULL的记录不统计在内。**


### 总结
由于count（列名）时，为NULL时，不会统计在内，这个点，踩了个坑，这里记录一下，平时还是需要多注意下细节。


### 参考
https://blog.csdn.net/wendychiang1991/article/details/70909958/