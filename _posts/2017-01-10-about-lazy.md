---
layout: post
title:  "惰性求值的简易实现"
date:   2017-07-10 10:00:56 +0800
categories: jekyll update
---

>在编程语言理论中，惰性求值（英语：Lazy Evaluation），又译为惰性计算、懒惰求值，也称为传需求调用（call-by-need），是一个计算机编程中的一个概念，它的目的是要最小化计算机要做的工作。它有两个相关而又有区别的含意，可以表示为“延迟求值”和“最小化求值”，除可以得到性能的提升外，惰性计算的最重要的好处是它可以构造一个无限的数据类型。

## 实现方法

* range(to, from): 接受两个Number类型的参数，返回一个从to到from的数组；
* map(fn): 对每一个数组元素使用fn处理；
* filter(fn): 过滤；
* take(n): 接受一个Number类型参数，对数组截断，暂时只接受大于0小于数组长度的参数；
* join: 将所有元素收集成一个数组，当使用调用到join才开始真正计算，从而实现惰性求值。

## 实现思路

因为在JS中，表达式是会被立即计算的，所以要实现惰性求值，只能返回一个方法，等到调用到这个返回方法的时候再去求值，所以range方法大概应该长成这样：
```javascript
function range (from, to) {
    var i = from
    return function () {
        if (i++ < to) {
            console.log('range\t', i)
            return i
        }
    }
}
```
还有一需要解决的问题是如何表示结束，上面的方法中i超过边界久没有返回值的这显然是不行的，我们需要一个唯一值，当返回这个唯一值的时候表示迭代结束了，我们利用ES6新增的第七个基本类型可以做到：
```javascript
var over = Symbol()

function isOver (_over) {
    return _over === over
}

function range (from, to) {
    var i = from
    return function () {
        if (i++ < to) {
            console.log('range\t', i)
            return i
        }
        return over
    }
}
```
然后我们在后面的处理方法里只要判断isOver就可以知道是否结束了，比如map方法，如果迭代没有结束就把当前的计算值作为参数传给mapFn，否则就返回返回当前的计算值：
```javascript
function map (flow, transform) {
    return function () {
        var data = flow()
        console.log('map\t', data)
        return isOver(data) ? data : transform(data)
    }
}
```
filter和take方法也是类似的思路，join方法有一点不一样，join方法负责收集所有的结果返回一个数组：
```javascript
function join (flow) {
    const array = []
    while (true) {
        var data = flow()
        if (isOver(data)) {
            break
        }
        array.push(data)
    }
    return array
}
```
目前基本功能都实现了，调用一下：
```javascript
cosnt nums = join(take(filter(map(range(0, 20), n => n*10), n => n%3 === 0), 2))
console.log(nums)

/*
output:
range	 1
map	 1
range	 2
map	 2
range	 3
map	 3
filter	 30
range	 4
map	 4
range	 5
map	 5
range	 6
map	 6
filter	 60
[ 30, 60 ]
*/
```
可以看到，当range返回一个值后立刻被map处理了，并不是range处理了所有的元素才到map，而且，当take数量足够的时候即使结束了迭代，避免了不必要的便利，这点在处理体量大的数组的时候可以节省许多性能。

但是，看起来太丑了，一点都没有lazyjs和lodash那样优雅，我们需要用一个封装来实现链式调用：
```javascript
function _Lazy() {
    this.iterator = null

    this.range = function (from, to) {
        this.iterator = range(from, to)
        return this
    }

    this.map = function(mapFn) {
        this.iterator = map(this.iterator, mapFn)
        return this
    }

    this.filter = function (filterFn) {
        this.iterator = filter(this.iterator, filterFn)
        return this
    }

    this.take = function (n) {
        this.iterator = take(this.iterator, n)
        return this
    }

    this.join = function () {
        return join(this.iterator)
    }
}

function lazy() {
    return new _Lazy()
}
```
这时调用就变成这样，好看多了逼格也高：
```javascript
const nums2 = lazy().range(0, 20).map(n => n*10).filter(n => n%3 === 0).take(2).join()

console.log(nums2)
```

## 补充

[完整代码](https://gist.github.com/Adamwu1992/a2499710ea260a0a669c630fba5ad79a)

惰性求值使用了函数式编程的思想，对于我们这种OOP语言出身的程序猿在这一点上转变是很别扭的，但是目前越来越多的库在推崇这种编程思想，前段时间看RxJS时深有体会，虽然日常实现的业务场景可能暂时不会有这样的需求，但是人嘛，开心最重要，多折腾折腾可以延缓衰老

:）





