---
layout:     post
title:      "Java中锁的使用"
category:   服务端
date:       2018-05-13 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Java
---

>工作3年多了，写过一点点Java代码，用过sychronized，不过确实没有用过Lock这个东西，今天抽时间看看。


其实Lock是个接口：

```
public interface Lock {
	 // 获取锁，如果锁被占用，则等待；
    void lock();
    // 
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```

