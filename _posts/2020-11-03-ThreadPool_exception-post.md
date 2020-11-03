---
layout: post
title: 多线程异常不打印?
date: 2020-11-03 11:00:00 +0800
category: debug
thumbnail: /style/image/debug.png
icon: code
---

* content
{:toc}

# Thread pool
线程池的接触和使用
## 遇到问题
使用以下方法创建可缓存的线程池
```Java
Executors.newCachedThreadPool();
```
使用CountDownLatch()等待多线程执行完.

在多线程内部使用了ArrayList.get(int index)方法
当list为空或者index > list.size()时,抛出NullPointerException

但这块在实现时没有考虑到异常情况,
遇到问题时输出 list.get(i).toString(); 

但控制台没有输出.程序也没有继续往下执行(由于CountDownLatch)


导致表象上看 程序停在这里不会往下走

使用IDEA进行断点调试,并将Suspend改成Thread
并打印Thread信息, 发现多线程执行时, 异常信息在子线程中进行打印...

