---
title: Natives
date: 2017-10-20 09:53:15
tags: You Don't Know JS
---
## Array
```
new Array(1, 2, 3);// [1, 2, 3]
Array(1, 2, 3);// [1, 2, 3]
[1, 2, 3];//[1, 2, 3]
```
> 以上是3种, 初始化数组的方式

```
var arr = new Array(3);
arr.length;//3
arr;//[empty x 3]
```
> 以上是一个奇怪的现象, 当我们给数组初始化时, 只传递一个数值时, 它不是初始化一个数组包含这个值, 而是初始化了长度为这个值的数组

```
var a = new Array( 3 );//[empty x 3]
var b = [undefined, undefined, undefined ];//[undefined, undefined, undefined ]
var c = [];
c.length = 3; //[empty x 3]
var d = [, , ,]; //[empty x 3]
```
> 以上数组, 看似好像都是3个位置的数组, 但其实他们的值不全一样, 请看下面

```
a.map(function(v, key){ return key; }); // [ undefined x 3 ]
b.map(function(v, key){ return key; }); // [ 0, 1, 2 ]
```
> 以上, 我们发现a居然没有key值, 所以以后要undefined填充的数组使用b的方式声明

```
var a = Array.apply( null, { length: 3 } );
a; // [ undefined, undefined, undefined ]
```
> 以上是快速获取由undefined填充的数组

> 总结: 我们应该尽量避免使用对象形式创建数组, 应该使用 `var arr = []` 这种字面量形式

## Object(..), Function(..), and RegExp(..)
> 字面量的/正则表达式/比 `new RegExp`方式性能会好很多, 因为js引擎会对其进行提前编译和缓存

```
function isThisCool(vals,fn,rx) {
    vals = vals || Array.prototype;
    fn = fn || Function.prototype;
    rx = rx || RegExp.prototype;
    return rx.test(
        vals.map( fn ).join( "" )
	); 
}

isThisCool();       // true

isThisCool(
        ["a","b","c"],
		function(v){ return v.toUpperCase(); },
		/D/
		);// false
```
> 如上, 是一种初始化类似的方式, 比`[], function(){},  /(?:)/`在性能上会好一点, 但是尽量避免有修改属性的操作, 因为那样有可能影响到元素属性, 导致bug