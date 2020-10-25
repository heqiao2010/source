---
layout:     post
title:      "TCP链接数优化记录"
category:   服务端
date:       2018-06-15 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 服务端
---

![](https://img.hacpai.com/bing/20171207.jpg?imageView2/1/w/960/h/520/interlace/1/q/10) 

>之前在开发无线wifi认证系统的时候，遇到一次性能问题，表现就是前端3台nginx返回大量502和504，所有接口返回慢，包括html，css以及js等资源文件的响应也很慢。在后端服务器上，用`netstat`查看发现有大量数据阻塞在接收队列，服务器处理不过来，同时TCP链接数很多，不过后端服务器的CPU以及内存正常，所以之前怀疑是TCP链接数过多导致的，所以后续采取了一些方法来，减少后端服务器和前端nginx之间的链接数。


### 查看TCP链接
1. 查看TCP链接总数
总体来说，TCP连接数偏多

```
[root@portal-nginx2 ~]# netstat -an | grep ES| wc -l
6326
```

2. 查看TCP链接队列情况
如果接受队列阻塞了，说明后端服务处理请求太慢了，有性能问题；如果发送队列包很多，说明接收端有问题，或者虚机网络问题。


```
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
...

tcp        0  10369 192.168.1.10:10080      100.125.64.100:16916    ESTABLISHED
tcp        0  13169 192.168.1.10:10080      100.125.64.97:16809     ESTABLISHED
tcp        0  13568 192.168.1.10:10080      100.125.64.96:17071     ESTABLISHED
tcp        0  13595 192.168.1.10:10080      100.125.64.97:17074     ESTABLISHED
tcp        0  13596 192.168.1.10:10080      100.125.64.98:17070     ESTABLISHED
tcp        0  14600 192.168.1.10:10080      100.125.64.109:23059    ESTABLISHED
tcp        0  14851 192.168.1.10:10080      100.125.64.103:23019    ESTABLISHED
tcp        0  16157 192.168.1.10:10080      100.125.64.82:16914     ESTABLISHED
tcp        0  17680 192.168.1.10:10443      100.125.64.104:18713    ESTABLISHED
tcp        0  18653 192.168.1.10:10080      100.125.64.126:56441    ESTABLISHED
tcp        0  19766 192.168.1.10:10443      100.125.64.114:18725    ESTABLISHED
tcp        0  32120 192.168.1.10:10080      100.125.64.117:23044    ESTABLISHED
tcp        0  45716 192.168.1.10:10443      100.125.64.113:18725    ESTABLISHED
tcp        0  47955 192.168.1.10:10080      100.125.64.116:23033    ESTABLISHED
tcp        0  54595 192.168.1.10:10080      100.125.64.90:16961     ESTABLISHED
tcp        0  54600 192.168.1.10:10080      100.125.64.85:16961     ESTABLISHED
tcp        0  62454 192.168.1.10:10080      100.125.64.149:46132    ESTABLISHED
tcp        0  63689 192.168.1.10:10080      100.125.64.127:56506    ESTABLISHED
```



### 系统参数设置
这几个系统参数，印象最深的是`net.ipv4.tcp_fin_timeout`,系统默认值是60，表示链接断开后，链接保持fin_wait的时间（我记得阿里的java开发规范里有提到这个）。

```
[root@XXX ~]# cat /etc/sysctl.conf
# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).
net.ipv4.icmp_echo_ignore_broadcasts = 1 #避免放大攻击
net.ipv4.tcp_max_syn_backlog=8192  #表示SYN队列的长度，默认为1024
net.core.netdev_max_backlog=30000   #进入包的最大设备队列.默认是1000
net.core.somaxconn=8192 #挂起请求的最大数量.默认是128.
net.ipv4.ip_local_port_range = 3000     65000  #端口范围，可用端口数
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_max_tw_buckets=5000 #表示系统同时保持TIME_WAIT套接字的最大数量
```

修改完后，`sysctl -p`生效。

### 反向代理设置
nginx.conf中的upstream配置增加keepalive参数，keepalive后的参数是以秒为单位的时间。

```
upstream portal {
            server 192.168.1.128:7878;
            server 192.168.1.60:7878;
            server 192.168.1.82:7878;
            server 192.168.1.69:7878;
            server 192.168.1.253:7878;
            server 192.168.1.160:7878;
            server 192.168.1.33:7878;
            keepalive 500;
    }
```

这里谈到keepalive就想到了TCP的keepalive了，TCP的keepalive机制实际上是在没有数据传输的一个保活机制。TCP在3次握手之后，就可以开始传输数据了，但是当没有数据可传输的时候，为了知道对端是否还保持这个链接，TCP协议会定期发送一个数据为空的探测报文，看看对端是否会响应这个报文，如果不响应，则说明这个链接已经丢失了。TCP的keepalive可以设置TCP协议在没有数据传输后多久开始发送探测报文，以及发送探测报文重试次数等等。

实际上上面nginx.conf中，这个keepalive不是配置TCP探测报文的（nginx.conf中真正TCP探测报文的配置是通过`so_keepalive`控制的），这个keepalive的官方解释如下：

1.  The connections parameter sets the maximum number of idle keepalive connections to upstream servers connections（设置到upstream服务器的空闲keepalive连接的最大数量）
2.  When this number is exceeded, the least recently used connections are closed. （当这个数量被突破时，最近使用最少的连接将被关闭）
3.  It should be particularly noted that the keepalive directive does not limit the total number of connections to upstream servers that an nginx worker process can open.（特别提醒：keepalive指令不会限制一个nginx worker进程到upstream服务器连接的总数量）


http的传输层协议是TCP，一个http请求是通过TCP传输的数据，当一个http请求发出，然后得到响应之后，TCP链接可能会关闭，下一次发起http请求时，再重新创建TCP链接，这个过程，必然会有TCP多次创建的过程，而创建过程会消耗额外资源。如果能够在一段时间内复用之前已经创建的TCP链接，则可以提升效率。这个超时时间可以通过nginx.conf中http部分的`keepalive_timeout`来控制。
```
keepalive_timeout timeout [header_timeout];
```

参考1：https://www.cnblogs.com/xiaoleiel/p/8308514.html 
参考2：https://blog.csdn.net/dream_flying_bj/article/details/54709549