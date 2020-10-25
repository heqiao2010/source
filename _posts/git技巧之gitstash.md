---
layout:     post
title:      "git技巧之git stash"
category:   git
date:       2019-06-13 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - git
---

## 功能
英文单词：stash的原意是储藏的意思。当我们在开发过程中，代码写了一部分，由于某种原因需要切换到另一个分支上，比如临时需要切换到另一个需求上开发另一个任务，这个时候就我们就需要某种手段临时保存手头上没有完成的任务，在另一个任务做完了之后，再恢复这个没有完成的任务继续开发。这个过程是不是有点类似，操作系统中发生函数调用时，保存现场和恢复现场？

## 示例
1. 创建存储
```  
git stash
```
运行这个命令之后，会将当前所有的修改存储起来，再次运行git statu,会发现目录恢复成clean的状态了。这个时候就可以切换到其他分支了。

2. 查看存储列表
```
git stash list
```
有时候，我们运行了stash多次，通过上面的命令，可以查看每次存储的记录；类似：
```
$ git stash list
stash@{0}: WIP on master: 049d078 added the index file
stash@{1}: WIP on master: c264051 Revert "added file_size"
stash@{2}: WIP on master: 21d80a5 added number to log
```

3. 恢复存储

```    
git stash apply #应用存储
git stash pop　 #应用并删除存储
git stash drop　＃删除存储
```

上面的命令默认对最近一次的存储进行操作，如果需要制定那个存储，可以在明后跟上存储的名称，例如：
```
git stash apply stash@{2}
```

##从储藏中创建分支
```
git stash branch [分支名]
```
有时候想要对于暂存的代码新创建一个分支，可以用上面的命令进行；一般这种情况可能是应用存储时发生冲突，或者是想要创建分支进行其他操作。

## 参考

https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%82%A8%E8%97%8F%EF%BC%88Stashing%EF%BC%89