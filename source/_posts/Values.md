---
title: Values
date: 2017-10-17 20:01:54
tags: You Don't Know JS
---
## Arrays

```
var arr = [];
arr[0] = 0;
arr[2] = 2;
arr.length; //3
```
> 如上当我们跳过1索引位, 但是数组的长度还是变成了3而不是2, 索引1位置的值为`undefined`

```
var arr = [];
arr["hello"] = "world";
arr.length; //0
arr["2"] = 2;
arr.length; //3
```
> 如上当设置为字符串为key时, 不会算在数组长度里, 但是如果设置的是一个字符串的数值, 会为强转成数值

```
Array.prototype.slice.call(arrayLike);
//等价于es6
Array.from(arrayLike)
```
> 如上是将类数组(dom对象数组或者函数的arguments参数数组)的数组转换为数组

## Strings
> 字符串, 有点像字符数组, 他们有一些公用方法, 但不完全一样, 比如字符串是无法改变自身位置, 但数组是可以的

```
var str = "hello";
str.concat("~~", "world");//hello~~world

str.join;//undefined
Array.prototype.join.call(str, '-');//h-e-l-l-o
```
> 如上我们发现我们能使用部分数组的方法, 但是不能使用有些数组方法(比如reverse), 这是为什么呢? 因为join方法是返回一个新数组的, 而reverse是改变数组本身, 又因为字符串是无法改变自身位置的, 所以reverse无法使用

##Numbers
```
var num = .42;
//等价于如下
num = 0.42

var num2 = 42.;
//等价于如下
num2 = 42.0;

var num3 = 4E2; //400

//.toFiex保留几位小数
var num4 = 42.59;
num4.toFixed(0)//43
num4.toFixed(1)//42.6
num4.toFixed(2)//42.59
num4.toFixed(4)//42.590
num4.toFixed(5)//42.5900

//.toPrecision保留几位数
var a = 42.59;
a.toPrecision( 1 ); // "4e+1"
a.toPrecision( 2 ); // "43"
a.toPrecision( 3 ); // "42.6"
a.toPrecision( 4 ); // "42.59"
a.toPrecision( 5 ); // "42.590"
a.toPrecision( 6 ); // "42.5900"
```

```
0x12//十六进制
0o12//八进制 在非严格模式下可以使用012, 但已废弃, 不建议再使用
0b11//二进制
```

```
0.1 + 0.2 == 0.30000000000000004

function numbersCloseEnoughToEqual(n1,n2) {
        return Math.abs( n1 - n2 ) < Number.EPSILON;
}
var a = 0.1 + 0.2;
var b = 0.3;
numbersCloseEnoughToEqual( a, b );
```
> 如上得知, js的小数计算结果有偏差, 所以可以使用`Number.EPSILON`来计算是否相等

```
Number.MAX_VALUE;//1.798e+308
Number.MIN_VALUE;//5e-324
Number.MAX_SAFE_INTEGER;//9007199254740991= 2^53 - 1
Number.MIN_SAFE_INTEGER;//-9007199254740991
```
> js整数最大用53位表示, 所以如果计算64位数时需要额外引入库

```
Number.isInteger( 42 );     // true
Number.isInteger( 42.000 ); // true
Number.isInteger( 42.3 );   // false
```

## undefined
> undefined是一个标识符, 而null是关键字

```
function foo() {
    "use strict";
    var undefined = 2;
    console.log( undefined ); // 2
}
foo()
```

```
var foo = void 0;
foo;//undefined
```
> 可以通过void获取undefined值, void _ 会一直返回undefined值

## NaN - Not a Number
```
typeof NaN // number

NaN == NaN; //undefined
```

```
isNaN(NaN); //true
isNaN(1); //false
isNaN('1'); //false
isNaN('hello'); //true
```
> 以上得知, isNaN是用来判别是不是数字, 如果不是数字就返回true, 否则就是true

```
Number.isNaN(NaN); //true
Number.isNaN(1); //false
Number.isNaN('1'); //false
Number.isNaN('hello'); //false
```
> es6提供的`Number.isNaN`修复了这个问题

```
if (!Number.isNaN) {
        Number.isNaN = function(n) {
            return (
                typeof n === "number" &&
                window.isNaN( n )
			); 
		};
}

if (!Number.isNaN) {
	 Number.isNaN = function(n) {
	 	//只有NaN不等于自身
        return n !== n;
    };
}
```
> 两种es6 isNaN的polyfill

### Zeros
> js里有0和-0, 它的用处不是很大, 但是在某些运动时刻, 可以通过正负值, 来判断运动方向

```
var a = 0 / -3; // -0
var b = 0 * -3; // -0

a.toString(); //"0"
a + ""; //"0"
String( a ); //"0"

JSON.stringify( a );   // "0"
JSON.parse( "-0" ); // -0

Number( "-0" );     // -0

-0 === 0;//true
```

```
function isNegZero(n) {
    n = Number( n );
    return (n === 0) && (1 / n === -Infinity);
}
isNegZero( -0 );// true
isNegZero( 0 / -3 );// true
isNegZero( 0 );// false
```
> 可以通过上面的方法判断是否为负0

### Special Equality
> es6提供的提供了`Object.is`方法可以来判断两值是否相等
```
var a = 2 / "foo";
var b = -3 * 0;

Object.is( a, NaN ); //true
Object.is( b, -0 ); //true
Object.is( b, 0 ); // false
```

```
if (!Object.is) {
        Object.is = function(v1, v2) {
            // test for `-0`
            if (v1 === 0 && v2 === 0) {
                return 1 / v1 === 1 / v2;
            }
            // test for `NaN`
            if (v1 !== v1) {
                return v2 !== v2;
            }
            // everything else
            return v1 === v2;
        };
}
```
> polyfill如上