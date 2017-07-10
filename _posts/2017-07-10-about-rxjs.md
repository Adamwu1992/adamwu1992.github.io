---
layout: default
title:  "使用RxJS"
date:   2017-07-10 10:00:56 +0800
categories: jekyll update
---

> Reactive Extensions (Rx)是一个用来处理异步任务的库，基于事件驱动(event-based)，使用可观察的序列和LINQ-style的查询操作。

在从web page慢慢转变到web app的过程中，前端开发面临着越复杂的逻辑处理和数据状态管理，传统的管理数据方式难以为继。在试玩Angular2时接触到这个RxJS这个库，感觉它的理念非常先进。

Reactive Extensions 是微软（研究院）提出的一个函数响应式编程抽象，最早用于.Net中，之后也被大量移植到其它语言，比如 RxJS 就是 Rx 的 JavaScript 移植版本。

## 什么是Reactive Programming？

响应式编程（姑且这么翻译吧）就是基于异步数据流编程，它处理的对象都是数据流，事件总线和或者是典型的点击事件都是事件流，你可以观察它并且对他做一些处理。任何东西都可以当作流来处理：变量、用户输入、属性、缓存等，最重要的是，RxJS提供了一系列的接口来联合、创建、过滤这些流。

流（stream）是一系列正在发生的事件按事件排成的序列，它可以发射三种状态：value、error和完成标志。如果将点击事件转化成流的话，它的完成标志将在这个button（或者其他元素）所在的窗口被关闭时发射。

我们只能异步的捕获这个被发射的状态，然后定义一个函数去处理它，有时候后两种状态可以被忽略，我们只要专注于监听被发射的value，这个监听流的过程叫订阅(subscribing)，我们定义的处理函数叫观察者(observers)，流就是被观察的subject，这就是喜闻乐见的观察者模式了。

## 如何使用RxJS处理一个点击事件

学习RxJS最重要的就是用上述的思想分析问题。假设我们面临一个需求：**如何统计页面上一个按钮被点击的次数？**

哈哈是不是非常简单，老夫用jq一分钟写十个。让我们尝试用流的思想来分析一下。

在RxJS的库中，提供了很多方法可以处理流，比如```map```，```filter```，```scan```等。当你调用其中的方法是，比如```clickStream.map(fn)```时，它会基于点击事件流返回一个新的流，并不会对原始流有任何影响，这个特性叫不变性(immutability)。我们可以用链式调用处理这个问题：
```javascript
clickStream.map(f).scan(g)

/*
  clickStream: ---c----c--c----c------c-->
               map(c become 1)
               ---1----1--1----1------1-->
               scan(+)
counterStream: ---1----2--3----4------5-->
*/
```
以上只用两行代码就能实现，不能说不简单，但是场景太简单还不能显示出RxJS的优势，我们再升级一下问题：**如何统计页面上一个按钮被多次点击的次数，250ms内算一次点击区间？**

有点复杂了，jq如何实现呢？如我我来写，可能会用到定时器，没250ms去统计一次按钮被点击的次数，大于一次的在计数器上加一。但是长时间不点击页面这个事件也是要不断轮询的，有点性能浪费呢。

但是在RxJS中问题将在4行代码之内被解决：
```javascript
clickStream.buffer(dblClick$.throttleTime(250)).map(f).filter(g)

/*
  clickStream: --c----c-c----c-c-c-----c-c----c-->
               buffer(clickStream.throttleTime(250))
               --c-----cc------ccc------cc----c-->
               map(get length)
               --1------2--------3-------2----1-->
               filter(length >= 2)
counterStream: ---------2--------3-------2------->
*/
```
使用buffer函数将点击事件收集起来，使用map取到每个list的长度，再用filter过滤掉不合格的。先忘记掉代码实现，审视一下这个分析的过程，我们因该发现RxJS的优势是我们可以把一套方法应用在不同类型的数据上，换言之我们拿到初始数据可以根据需要做任何转变。

## 参考
[RxJS Design Guidelines](http://xgrommx.github.io/rx-book/content/guidelines/introduction/index.html)
[API文档](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html)

## 实现代码

以上例子的具体实现
```javascript
<body>
<button type="button" id='btn'>Click Me</button>
<label id="counter">0</label>
<script src="https://cdn.bootcss.com/rxjs/5.4.0/Rx.min.js"></script>
<script>
    const $btn = document.querySelector('#btn')
    const $counter = document.querySelector('#counter')
    const renderCounter = v => {
        $counter.innerHTML = v
    }

    //统计单击次数
    /*const click$ = Rx.Observable
            .fromEvent($btn, 'click')
            .map(v => 1)
            .scan((prev, cur) => {
                return prev + cur
            }, 0).subscribe(renderCounter)*/


    //统计多击次数
    const multiClick$ = Rx.Observable
            .fromEvent($btn, 'click')

    multiClick$.buffer(multiClick$.throttleTime(1000))
            .map(arr => {
                console.log(arr)
                return arr.length
            })
            .filter(l => {
                console.log(l)
                return l > 1
            })
            .map(() => 1)
            .scan((prev, cur) => {
                return prev + cur
            }, 0)
            .subscribe(renderCounter)


</script>
</body>
```
