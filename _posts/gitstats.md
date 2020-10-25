---
title:      "GitStats不错"
category:   git
date:       2018-02-15 12:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 代码统计
typora-root-url: ../../source
---

> 缘起：2017年老板说要搞一个年终总结，写了一个PPT，老板看了，觉得太虚，需要一些数据展示。而基于git原生的一些代码统计显得太单调了，发现网上大神写的gitstats工具不错。


## 安装
1.	安装依赖：Git，Python，Gnuplot。
2. `git clone git://github.com/hoxu/gitstats.git`

* 小插曲：
  在mac上安装Gnuplot的时候报错：
  `invalid active developer path (/Library/Developer/CommandLineTools)`

* 解决方法：
  在终端中执行如下命令：
  `xcode-select --install`
  <img src="/images/8.png" alt="安装提示" style="zoom:80%;" />

  

  系统弹出下载xcode相关插件，大概1分钟安装完毕，然后重试OK。

## 使用
gitstats下载下来之后，解压一共有如下几个文件：

```
-rw-r--r--@  1 heqiao  staff   1.0K  1  9  2016 Makefile 
-rw-r--r--@  1 heqiao  staff    73B  1  9  2016 arrow-down.gif
-rw-r--r--@  1 heqiao  staff    71B  1  9  2016 arrow-none.gif
-rw-r--r--@  1 heqiao  staff    73B  1  9  2016 arrow-up.gif
drwxr-xr-x@ 10 heqiao  staff   320B  1  9  2016 doc
-rwxr-xr-x@  1 heqiao  staff    47K  1  9  2016 gitstats
-rw-r--r--@  1 heqiao  staff   1.6K  1  9  2016 gitstats.css
-rw-r--r--@  1 heqiao  staff   9.3K  1  9  2016 sortable.js
```



其中最重要的就是gitstats，这个文件，实际上是个python脚本，运行：
`./gitstats [git仓库] [统计文件输出文件夹]`得到如下输出：

```
[0.01650] >> gnuplot --version
Output path: /Users/heqiao/Downloads/hoxu-gitstats-55c5c28/result
Git path: solo/
Collecting data...
[0.11579] >> git shortlog -s HEAD | wc -l
[0.00874] >> git show-ref --tags
[0.00897] >> git log "3580b4353004c88c8b8e6283aea1889be149074a" --pretty=format:"%at %aN" -n 1
[0.00794] >> git log "0473873adfc4820e1371f112325b8d733aec11f8" --pretty=format:"%at %aN" -n 1
[0.00774] >> git log "f302f95132b561b2d906b47c64cb5a2c8f6775f9" --pretty=format:"%at %aN" -n 1
[0.00895] >> git log "d78c8dc9e1e9f8bdfa047f2e4435daaf941c0d5a" --pretty=format:"%at %aN" -n 1
[0.01002] >> git log "80b05fb7bd09f860a8e79ceee61cf787aa6682ca" --pretty=format:"%at %aN" -n 1
[0.00767] >> git log "93f81cd81508f2c8a8fb64b6e4286062c023885e" --pretty=format:"%at %aN" -n 1
[0.00758] >> git log "1326eddf83578d992a4f08e45a5ddf3c9dd82431" --pretty=format:"%at %aN" -n 1
[0.00756] >> git log "a6356478a743cb681f70649d4791cc9586b5698e" --pretty=format:"%at %aN" -n 1
[0.00843] >> git log "d7a38706a7489edf997c9142ab2bf2e62ebc7ca8" --pretty=format:"%at %aN" -n 1
[0.00825] >> git log "c457ef0f91a880da238873790505e522d802484c" --pretty=format:"%at %aN" -n 1
[0.01002] >> git log "d8608b5ce71aaa44a3afd8d34d730dadc7dd63ce" --pretty=format:"%at %aN" -n 1
[0.00878] >> git log "869a4500faaffe27d3b075d293d7246ef4f98740" --pretty=format:"%at %aN" -n 1
...
```



生成的统计文件后，用浏览器打开目录中的index.html即可。SOLO项目的代码统计：
![统计文件](/images/7.png)



此外，为了让一些前端同学（不熟悉python和linux环境）也能用这个工具，自己写了一个能够用web访问的小工具：

见：https://github.com/heqiao2010/webGitstats