---
layout:     post
title:      "PropertyUtilsBean"
category:   Spring
date:       2018-08-25 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Java
---


![](https://img.hacpai.com/bing/20181129.jpg?imageView2/1/w/960/h/520/interlace/1/q/100) 

### beanutils包
beanutils，顾名思义，是java bean的一个工具类，可以帮助我们方便的读取(get)和设置(set)bean属性值、动态定义和访问bean属性；
细心的话，会发现其实JDK已经提供了一个java.beans包，同样可以实现以上功能，只不过使用起来比较麻烦，所以诞生了apache commons beanutils；
看源码就知道，其实apache commons beanutils就是基于jdk的java.beans包实现的。


### [AopCacheUtil](https://github.com/heqiao2010/AopCacheUtil)
之前在工作中，写了一个简单的自定义注解，后来自己凭印象写了个AopCacheUtil（还不完整），便于拦截DAO或者Controller层的方法调用，做缓存处理；当中就用到了PropertyUtilsBean，这个类。
在解析Bean属性的时候，挺方便的。
```
org.apache.commons.beanutils.PropertyUtilsBean.getProperty(Object, String)
```

### 参考
https://www.cnblogs.com/chenpi/p/6917499.html