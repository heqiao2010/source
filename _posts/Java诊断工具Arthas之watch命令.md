---
layout:     post
title:      "Java诊断工具Arthas之watch命令"
category:  Arthas
date:       2019-07-01 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Arthas
---

>Arthas是一个开源的Java诊断工具,详见:https://alibaba.github.io/arthas/index.html. 今天第一次在项目中排查问题使用到它,发现其功能确实很强大.这里记录一下watch命令的使用.

### 安装
安装JDK然后以jar包的形式运行即可.
```
wget https://alibaba.github.io/arthas/arthas-boot.jar
java -jar arthas-boot.jar
```

### 利用watch诊断方法调用

通过Arthas的watch方法可以在没有打印日志的情况下,看到方法的入参和返回值.这点在没有打印debug或者重启会破坏现场的情况下是非常有用的。

在运行arthas-boot.jar之后，会列出当前系统中的所有java进程，然后输入需要诊断的进程的序号，就进入了arthas的命令模式了。这里以官网上的java -jar arthas-demo.jar为例，毕竟为了信息安全，把在公司的debug信息透出不大好。

1. 首先可以利用sc命令，查询对应的类名。
```
$ sc *Math*
demo.MathGame
io.netty.util.internal.MathUtil
java.lang.Math
Affect(row-cnt:3) cost in 17 ms.
```

2. 然后再用sm命令，查询对应的方法。
```
$ sm demo.MathGame*
demo.MathGame <init>()V
demo.MathGame primeFactors(I)Ljava/util/List;
demo.MathGame main([Ljava/lang/String;)V
demo.MathGame run()V
demo.MathGame print(ILjava/util/List;)V
Affect(row-cnt:5) cost in 9 ms.
```

3. 利用watch命令，监听指定的方法，-x 参数可以指定打印入参和返回值的深度。

```
$ watch demo.MathGame primeFactors "{params,returnObj}" -x 2
Press Q or Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 49 ms.
ts=2019-07-03 14:29:41; [cost=1.778091ms] result=@ArrayList[
    @Object[][
        @Integer[-136776],
    ],
    null,
]
ts=2019-07-03 14:29:42; [cost=0.105508ms] result=@ArrayList[
    @Object[][
        @Integer[47524],
    ],
    @ArrayList[
        @Integer[2],
        @Integer[2],
        @Integer[109],
        @Integer[109],
    ],
]
```
我就是利用这个工具，诊断出系统中一个很奇怪的问题。最终原因是因为PHP程序调用Java接口在时序上有略微的差异导致的。
