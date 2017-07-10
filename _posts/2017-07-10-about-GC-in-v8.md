---
layout: default
title:  "理解V8内存处理"
date:   2017-07-10 10:00:56 +0800
categories: jekyll update
---

## 定义

## GC是如何工作的

Garbage collection是一个进程，它回收不再被应用程序使用的对象所占用的内存。通常来说，内存分配是廉价的，但是当内存池几乎耗尽时回收时昂贵的。

当一个对象从root不可到达时，也就是不再被跟对象或其他活动对象引用时，它将成为GC的候选对象。root可以是全局对象，DOM对象或者局部变量。

堆（the heap）有两个主要片段，New Space和Old Space。New Space是新的内存分配发生的地方，在这里收集垃圾也很快，它的大小是1～8MBs。在New Space中的对象被称作Young Generation。Old Space是对象在New Space的垃圾收集中幸存下来的对象进入到的地方，这些对象被称作Old Generation。内存分配在Old Space中是迅速的，但是垃圾收集昂贵的，所以很少执行。

> **为什么垃圾收集是昂贵的？**V8引擎使用一种stop-the-world的垃圾收集机制，就意味着当GC进入到进程中时程序将被停止执行。

通常大约20%的Young Generation会进入Old Space，Old Space中的垃圾收集只有在它运行缓慢时才会开始。V8引擎有两种不同算法达到这样的效果：

* **清除收集**，它是快速的，并且是对Young Generation执行的；

* **标记-清除收集**，它比较慢，对Old Generation执行

## 参考
[Hunting a Ghost - Finding a Memory Leak in Node.js](https://blog.risingstack.com/finding-a-memory-leak-in-node-js/)
