---
layout: post
title:  "对于Js事件轮询的思考"
date:   2017-07-10 10:00:56 +0800
categories: jekyll update
---

Javascript是单线程的语言，因为如果是多线程的话，在操作DOM是会变得难以掌控
    
setTimeout和setInterval这两个方法会在任务队列里加入任务，内部实现原理一致，这里只讨论setTimeout

```javascript
    window.setTimeout(function () {
        console.log('l1')
    }, 0)
    window.setTimeout(function () {
        var s = Date.now();
        console.log('l2')
    }, 0)
    console.log('l3')
```
以上的结果是l3、l1、l2。

尽管被设置为0秒后执行，但是l3在主线程里，**l1和l2在0ms之后被加入任务队列里**，只有主线程执行完毕才调用任务队列里第一个任务，也就是l1，任务队列是先进先出，所以l1在l2前面被执行

```javascript
    window.setTimeout(function () {
        console.log('l1')
    }, 10)
    window.setTimeout(function () {
        var s = Date.now();
        console.log('l2')
    }, 0)
    console.log('l3')
````
以上结果是l3、l2、l1。
当主线程执行完毕后，如果执行时间小于10ms，那么l1还没有被加入任务队列，l2先执行

```javascript
    window.setTimeout(function () {
        console.log('l1')
    }, 50)
    window.setTimeout(function () {
        console.log('l2')
    }, 0)

    console.log('l3')
    for (var i=0; i<1000000000; i++) {}
```
以上结果是l3、l2、l1，且l2和l1几乎同时输出。
按道理l1延迟50ms加入任务队列，l2立即加入任务队列，两个方法的执行间隔应该最小50ms左右，但是由于主线程力有一个同步任务执行时间大大超过了50ms，所以等主线程闲置下来后，l2和l1早已被添加进任务队列，所以这两个任务会没有间隔连续执行
