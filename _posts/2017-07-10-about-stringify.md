---
layout: default
title:  "有关JSON.stringify()的小知识"
date:   2017-07-10 10:00:56 +0800
categories: jekyll update
---

>前段时间，和同事讨论代码时发现，用`JOSN.stringify()`做对象转换的时候，会丢失类型为`function`的成员，于是查了一下[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)，这才发现，平时用的这么顺手的方法竟然还有一些不为人知（也可能只有我不知道）的特性。

## 方法描述

我们通常用这个方法来将一个JSON对象转换成JSON字符串，配合`JOSN.parse()`可以快速克隆对象。
这句话是存在一些误区的，首先被转换的值不限于JSON对象，以下的一些值也可以被转换：

```javascript
JSON.stringify(1)
// "1"
JSON.stringify(false)
// "false"
JSON.stringify('hello')
// ""hello""
JSON.stringify([1, false, 'hello'])
// "[1,false,"hello"]"
```
其次，`undedined`、`function`或者`Symbol`类型的值将会被特殊处理。具体来说，对象的value是以上类型，该健值对会被忽略；数组的某个元素是以上类型，将会被转化为`null`；如果单纯地用以上类型的值做参数，则返回`undefined`:

```javascript
JSON.stringify({a: function() {}, b: undefined, c: Synbol('')})
// "{}"
JSON.stringify([function() {}, undefined, Symbol('')])
// "[null,null,null]"
```
最后，所有key类型为`Symbol`的属性和不可枚举的属性都会被忽略。

## 参数

在文档中，`JSON.stringify()`的参数有三个：
- value 
&emsp;&emsp;被转换为string的值。
- replacer（可选）
&emsp;&emsp;可以是一个function或者是包含string或者number对象的数组。如果是方法，被转换的对象的每一个被遍历的值都作为参数被传入；如果是数组，数组将会作为一个白名单，被转换的对象中不在数组内的值都将被忽略。
- space（可选）
&emsp;&emsp;可以是一个string或者number类型的值，这个值将被插入转换后的字符串的的空白位置，以增加可读性。

第一个大家都知道，第三个参数比较简单，可以自定义分隔符，使用的场景比较少，而第二个参数看起来很有搞头。

按照文档所说，如果传入一个方法，就可以接受到所有被遍历的属性：

```javascript
JSON.stringify(
    {a: function() {}, b: undefined, c: Symbol('')},
    (key, value) => {
        console.log(key, value);
        return value;
    }
)
/**
 {b: undefined, c: Symbol(), a: ƒ}
a ƒ () {}
b undefined
c Symbol()
*/

JSON.stringify(
    [function() {}, undefined, Symbol('')], 
    (key, value) => {
        console.log(key, value);
        return value;
    }
)
/**
 (3) [ƒ, undefined, Symbol()]
0 ƒ () {}
1 undefined
2 Symbol()
*/
```
不仅打印出了三个属性，连对象本身也打印出来了，对象本身的key值为空，用这个方法可以处理在转换过程中的特殊值，也可以根据需要定制转换方法。

如果传入数组，则起到了白名单的过滤效果：

```javascript
const n = {
  a: '1',
  b: '2',
  c: {
    d: '4',
    e: {
      f: '6'
    },
    g: '7'
  }
};

JSON.stringify(n, ['a', 'b', 'c', 'd', 'aa']);
// {"a":"1","b":"2","c":{"d":"4"}}
```
不仅过滤了第一层的属性，连嵌套的属性也可以被过滤。

## 特殊行为

`JSON.stringify()`在转换的时候会隐式的调用`toJOSN`方法，这个方法的返回值将作为`JSON.stringify()`的返回值：

```javascript
const n = {
  a: '1',
  b: '2',
  c: {
    d: '4',
    e: {
      f: '6'
    },
    g: '7'
  },
  toJSON: function () {
    return 'hello world';
  }
};

JSON.stringify(n, ['a', 'b', 'c', 'd', 'aa']);
// "hello world"
```
