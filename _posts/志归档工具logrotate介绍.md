---
title: 日志归档工具logrotate介绍
layout: post
category: logrotate
date: 2020-06-17 18:00:00
---

### 缘起
对于日志的归档，轮转以及清理，很多框架都已经支持，比如logback，log4j等，对于logrotate鲜有用武之地。

不过最近做一个项目的时候，客户环境存在大量的syslog，但是对这类syslog日志确没有做定时的清理和归档，于是看了下centos自带的logrotate工具，还是蛮方便的。

### 业务需求
希望能够做到日志轮转，保留一周的日志，每天的日志保存到一个文件中，一周之前的日志，自动删除；2天之前的日志能够归档压缩；

### 解决
在/etc/logrotate.d/目录新增logrotate配置，例如：
cat /etc/logrotate.d/test

```
/data/log/test.log {
        daily    #每天执行
        missingok # 如果日志不存在，则忽略
        rotate 7  # 保留7天的日志
        create 644 root root  # 新创建文件权限
        dateext  # 用日期作为文件名后缀
        dateformat -%Y%m%d.%s #日期格式
        nocompress  #不需要压缩，compress 表示用gzip压缩
        copytruncate #采用拷贝后截断的方式，归档日志，好处是避免日志归档对正在写这个log的程序出现问题 
}
```


### 参考文章
https://blog.huoding.com/2013/04/21/246

https://linux.die.net/man/8/logrotate
