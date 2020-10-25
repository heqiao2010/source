---
layout:     post
title:      "Syslog基础"
category:   syslog
date:       2019-06-10 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - linux
---

# Syslog基础

Syslog是类Unix操作系统中，用于记录系统日志（产生自本地或者远程操作系统）到本地磁盘的一套日志格式以及对应的程序。完整的syslog日志中包含产生日志的程序模块（Facility）、严重性（Severity或 Level）、时间、主机名或IP、进程名、进程ID和正文。


# Syslog日志格式

Syslog日志格式比较松散，一般分为PRI，HEADER以及MSG三个部分，例如：
```text
<30>Oct 9 22:33:20 hlfedora auditd[1787]: The audit daemon is exiting.
```

## PRI部分
PRI部分由尖括号包含的一个数字构成，这个数字包含了程序模块（Facility）、严重性（Severity），这个数字是由Facility乘以 8，然后加上Severity得来。也就是说这个数字如果换成2进制的话，低位的3个bit表示Severity，剩下的高位的部分右移3位，就是表示Facility的值。Facility的定义如下：

|Numerical Code |         Facility|
-|-|-
|           0   |         kernel messages|
|           1   |         user-level messages|
|           2   |         mail system|
|           3   |         system daemons|
|           4   |         security/authorization messages (note 1)|
|           5   |         messages generated internally by syslogd|
|           6   |         line printer subsystem|
|           7   |         network news subsystem|
|           8   |         UUCP subsystem|
|           9   |         clock daemon (note 2)|
|          10   |         security/authorization messages (note 1)|
|          11   |         FTP daemon|
|          12   |         NTP subsystem|
|          13   |         log audit (note 1)|
|          14   |         log alert (note 1)|
|          15   |         clock daemon (note 2)|
|          16   |         local use 0  (local0)|
|          17   |         local use 1  (local1)|
|          18   |         local use 2  (local2)|
|          19   |         local use 3  (local3)|
|          20   |         local use 4  (local4)|
|          21   |         local use 5  (local5)|
|          22   |         local use 6  (local6)|
|          23   |         local use 7  (local7)|
          

Severity的定义：


|Numerical  Code |      Severity|
-|-|-
|           0    |   Emergency: system is unusable|
|           1    |   Alert: action must be taken immediately|
|           2    |   Critical: critical conditions|
|           3    |   Error: error conditions|
|           4    |   Warning: warning conditions|
|           5    |   Notice: normal but significant condition|
|           6    |   Informational: informational messages|
|           7    |   Debug: debug-level messages|


## HEADER部分
HEADER部分包括两个字段，时间和主机名（或IP）。
时间紧跟在PRI后面，中间没有空格，格式必须是“Mmm dd hh:mm:ss”，不包括年份。“日”的数字如果是1～9，前面会补一个空格（也就是月份后面有两个空格），而“小时”、“分”、“秒”则在前面补“0”。月份取值包括：Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec；时间后边跟一个空格，然后是主机名或者IP地址，主机名不得包括域名部分。

## MSG部分
MSG部分又分为两个部分，TAG和Content。其中TAG部分是可选的。在前面的例子中（“<30>Oct 9 22:33:20 hlfedora auditd[1787]: The audit daemon is exiting.”），“auditd[1787]”是TAG部分，包含了进程名称和进程PID。PID可以没有，这个时候中括号也是没有的。进程PID有时甚至不是一个数字，例如“root-1787”，解析程序要做好容错准备。TAG后面用一个冒号隔开Content部分，这部分的内容是应用程序自定义的。



# 参考
https://www.cnblogs.com/skyofbitbit/p/3674664.html

http://www.ietf.org/rfc/rfc3164.txt

http://www.ietf.org/rfc/rfc3195.txt