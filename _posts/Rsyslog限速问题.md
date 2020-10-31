---
title: Rsyslog限速问题
top: false
cover: false
toc: true
mathjax: true
date: 2019-06-11 12:28:52
password:
summary:
tags: 限速
categories: syslog
---

**问题：**
今天遇到一个syslog间歇性缺失的问题，排查了一圈发现是限速导致的。

```
cat /var/log/message | grep rate-limit
```

有如下信息：

```
Feb  9 10:22:32 localhost rsyslogd: imuxsock lost 432 messages from pid 9832 due to rate-limiting
```

**解决：**

1.修改配置文件
对于centos6，修改/etc/rsyslog.conf新增
```
$SystemLogRateLimitInterval 0
$SystemLogRateLimitBurst 0
$IMUxSockRateLimitBurst 0
$IMUXSockRateLimitInterval 0
$IMUxSockRateLimitSeverity 7 
```
对于centos7，修改/etc/rsyslog.conf新增
```
$SystemLogRateLimitInterval 0
$SystemLogRateLimitBurst    0
# 在$ModLoad imjournal之后新增
$ImjournalRateLimitInterval 0
```
修改：/etc/systemd/journald.conf
注释掉：
```
RateLimitInterval=5s
RateLimitBurst=30000
```

2.重启服务
```
systemctl restart systemd-journald
systemctl restart rsyslog
```

**参考资料**
https://www.thegeekdiary.com/imuxsock-lost-messages-from-pid-due-to-rate-limiting-rsyslog-rate-limiting-in-linux/
https://www.rsyslog.com/how-to-use-rate-limiting-in-rsyslog/