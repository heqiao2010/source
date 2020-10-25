---
layout:     post
title:      "VirtualBox虚拟机和宿主机实现网络互通配置"
category:   虚拟机
date:       2019-06-19 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - VirtualBox
---

>由于想要在本地测试一下syslog以及安装jenkins等需求，所以想在本地安装一个虚拟机，并且能够在宿主机上访问，所以想利于virtualbox上安装一个linux来实现，尝试了几次，其实配置挺简单的，这里记录一下。

## 接入方式对比

VirtualBox的提供了四种网络接入模式

1. NAT 网络地址转换模式(NAT,Network Address Translation) ：
宿主机做nat转换，对外外部网络来说，虚拟机是不可见的，因为宿主机代理了虚拟机的所有请求。
2. Bridged Adapter 桥接模式 ：
对于虚拟机，外部网络可见，虚拟机和宿主机存在于同一个网段。
3. Internal 内部网络模式 ：创建一个隔离的虚拟网络，在这个网络中的虚拟机之间可以相互访问，虚拟机不能访问外部网络，外部网络也不能访问内部虚拟机。
4. Host-only Adapter 主机模式 ：　也可以通过配置实现，下次有机会再看看。

各个接入方式对比：

|   | NAT | Bridged    | Internal  |Host-only  |
|-------|:---:|-----------|--------|-------:|
| 虚拟机－>宿主机  | √ | √     | × |默认不能，需设置|
| 宿主机－>虚拟机 | × | √      | × |默认不能，需设置|
| 虚拟机－>其他主机  | √ | √ | × |默认不能，需设置|
| 其他主机－>宿主机  | ×   | √ | × |默认不能，需设置|
| 虚拟机之间　  | ×   | √ | 同网络名下可以 |√|


## 配置
从对比中可以看出桥接方式是最优的方案，但是配置之后，宿主机和虚拟机之间能够互通，但是在虚拟机中不能上外网了，因为配置的宿主机DNS不能解析外部域名，而虚拟机中也无法ping通诸如114.114.114.114的地址。

如果做NAT的话，虚拟机是可以访问外网的，所以可以给虚拟机设置两个网卡。一个做桥接一个做NAT就达到目的了。

![](https://raw.githubusercontent.com/heqiao2010/heqiao2010.github.io/master/img/2019/virtualbox-netconfig.png )

需要注意的是桥接网卡在虚拟机中需要手动配置一个和宿主机同网段的IP，网关和DNS最好和宿主机配置一致。


## 参考

https://blog.csdn.net/chaishen10000/article/details/82984811

