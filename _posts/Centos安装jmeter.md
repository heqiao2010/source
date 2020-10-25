---
layout:     post
title:      "CentOs安装Jmeter"
category:   服务端
date:       2018-05-13 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 服务端
---

>前段时间公司生产环境，压力增长了不少，后来出过几次性能问题；为了避免高峰期服务器压力过大，导致服务不可用的情况，领导要求每次合入
影响性能的代码之后，都需要做压力测试了。
我们用jmeter做压力测试，之前是在实验室环境的window上安装图形化的jmeter进行测试，但是由于实验室带宽限制，流量达不到要求，后来就直接在服务器上
安装jmeter基于命令行打流；


#如何在CentOS中安装Jmeter
1. Install JRE

	`yum install java`

2. 下载jmeter

	`wget http://mirrors.hust.edu.cn/apache//jmeter/binaries/apache-jmeter-4.0.tgz`
	`tar zxf apache-jmeter-4.0.tgz`

1. 拷贝到系统目录

	`mv apache-jmeter-4.0 /usr/local/`

1. 修改profile

	`vim /etc/profile`
	
	然后追加：
	
	`#/usr/local/apache-jmeter-4.0`
	`export JMETER=/usr/local/apache-jmeter-4.0/`
	`export CLASSPATH=$JMETER/lib/ext/ApacheJMeter_core.jar:$JMETER/lib/jorphan.jar:$JMETER/lib/logkit-2.0.jar:$CLASSPATH`
	`export PATH=$JMETER/bin/:$PATH`

	然后source生效
	`source /etc/profile`

1. 验证

`[?]# jmeter -v`
`May 12, 2018 9:44:30 AM java.util.prefs.FileSystemPreferences$1 run`

`INFO: Created user preferences directory.`
`_ ____ _ ____ _ _ _____ _ __ __ _____ _____ _____ ____`
`/ \ | _ \ / \ / ___| | | | ____| | | \/ | ____|_ _| ____| _ \`
`/ _ \ | |_) / _ \| | | |_| | _| _ | | |\/| | _| | | | _| | |_) |`
`/ ___ \| __/ ___ \ |___| _ | |___ | |_| | | | | |___ | | | |___| _ <`
`/_/ \_\_| /_/ \_\____|_| |_|_____| \___/|_| |_|_____| |_| |_____|_| \_\ 4.0 r1823414`
`Copyright (c) 1999-2018 The Apache Software Foundation`

1. 如何基于命令行运行

	`jmeter -n -t test1.jmx -l result.log 2>/dev/null`