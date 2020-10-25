---
layout:     post
title:      "Shell 实现的几个Http交互的方法"
category:   服务端
date:       2018-06-14 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Java
---

![](https://img.hacpai.com/bing/20171206.jpg?imageView2/1/w/960/h/520/interlace/1/q/10) 

>由于测试需求，需要测试一些http接口是否正常，作为后端开发可以用脚本语言来实现比如python,shell之类的，这里分享一下，用shell如何实现。

* URLencode方法
既然是http接口，必定会用到urlencode来将参数放入URL中。
```
url_encode(){ 
  echo  "$1" | tr -d  '\n' | xxd -plain | sed 's/\(..\)/%\1/g' | tr -d  '\n'  
  return  0 
}
```

* URLdecode方法
```
url_decode(){
        printf $(echo -n $t | sed 's/\\/\\\\/g;s/\(%\)\([0-9a-fA-F][0-9a-fA-F]\)/\\x\2/g')
        return 0
}
```

* 解析JSON数据，从JSON对象中获取某个属性
```
parse_json(){
    json=`echo $1 | sed 's/\"//g'`; #remove quotation mark
    echo $json | sed 's/.*'$2':\([^,}]*\).*/\1/'
    return 0
}
```

* 解析URL参数，从URL中获取某个参数的值
```
parse_uri_paras(){
    echo $1 | sed 's/.*'$2'=\([[:alnum:]]*\).*/\1/'
    return 0
}
```

* 获取重定向地址
```
request_redirect_url(){
    echo `curl -i "$1" 2>/dev/null  | sed -n 's/^Location://p'`
    return 0
}
```

* 发送简单的GET请求
```
http_get(){
    get_data=`curl -X GET "$1" 2>/dev/null`
    if [ "$get_data" =  "" ]; then #出错了
            echo "出错了，试试：curl -X GET \"$1\""
            exit 1
    else 
        echo $get_data
        return 0
    fi
}
```

* 发送简单的POST请求
```
http_post(){
    post_data=`curl -X POST -d "$1" $2 2>/dev/null`
    if [ "$post_data" =  "" ]; then #出错了
                echo "出错了，试试：curl -X POST \"$1\" \"$2\""
                exit 1
        else
                echo $post_data
                return 0
        fi
}
```