---
layout:     post
title:      "如何获取用户真实IP"
category:   服务端
date:       2019-06-14 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 服务端
---

>在web开发过程中，我们经常需要获取用户客户端的真实IP。比如我们想知道客户的地理位置分布；比如服务端需要将会话和IP地址绑定，以提高安全性等。但是一般在分布式系统中，为了提高系统的可靠性和性能，都会采用代理来分发用户的请求，导致获取用户真实IP变得有些麻烦。

## 获取调用方IP的方法
直接调用HttpServletRequest中的request.getRemoteAddr()方法，获取的IP地址，是当前服务的上游服务的IP地址，在没有代理的情况下是准确的，不过一般都会有一个或者多个代理，这种方式一般不适用。一般可能用如下几个字段

### X-FORWARDED-FOR
X-FORWARDED-FOR这个http头是个调试参数，在代理服务器上做相关的配置，能够实现，在经过每一个代理的时候，会往后追加上代理的IP地址；这样通过这个字段就可以知道，这个请求被代理了多少次，每次代理是由那个IP处理的；所以取列表第一个IP就是用户的真实IP地址了。

nginx支持X-FORWARDED-FOR的配置：
```
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

### Proxy-Client-IP
用apache http做代理时一般会加上Proxy-Client-IP请求头

### WL-Proxy-Client-IP
weblogic插件加上的头

### HTTP_CLIENT_IP
有些代理服务器会加上此请求头。

### X-Real-IP
在nginx中可以把上游调用的真实IP加到HTTP请求头中，这个IP从remote_addr变量中获取是从TCP链接中获取的真实IP。

```
proxy_set_header X-Real-IP $remote_addr;
```

## 某些特殊情况
正常我们会按照上面的顺序按照优先级获取真实的IP地址。网上一般都会把X-FORWARDED-FOR这个参数作为第一优先级来处理。但是这个参数实际上是不安全的。
X-FORWARDED-FOR这个参数的值是个列表，每次经过代理都是往后追加，而不是覆盖的。所以不能确保IP是可信的。客户端可以先伪造一个IP放到头部，真实IP就会在伪造IP之后了。比如：

客户端伪造
```
X-FORWARDED-FOR：　1.1.1.1
```
经过两次代理（192.168.1.2,192.168.1.3）之后：
```
X-FORWARDED-FOR：　1.1.1.1,108.10.10.123,192.168.1.2,192.168.1.3
```
这样获取到的IP地址是1.1.1.1，而不是真正的108.10.10.123。其实真实IP已经在列表中了，怎么避免获取伪造的IP？

nginx有个realip模块，可以通过set_real_ip_from命令制定代理服务器的IP,因为代理服务器的IP地址都是已知的，所以可以通过从右至左依次获取第一个不在代理IP列表中的合法IP就是用户真实IP地址了。
```
set_real_ip_from  192.168.1.2;
set_real_ip_from  192.168.1.3;
real_ip_header    X-Forwarded-For;
real_ip_recursive on;
```
安装realip模块需要重新编译nginx，比较麻烦，也可以直接通过最上层代理（第一层代理）服务器获取用户真实IP，写入http头部，后端服务优先获取此参数即可。比如：第一层代理是nginx,将remote_addr参数写入X-Real-IP，后端优先获取此参数，其他代理不要覆盖此参数即可；由于参数是覆盖写入的，就算客户端伪造一个X-Real-IP参数头，也会被代理服务器复写为真实的IP.

## 参考

https://www.jianshu.com/p/1e0124de8e02
http://nginx.org/en/docs/http/ngx_http_realip_module.html