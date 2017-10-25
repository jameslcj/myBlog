---
title: Coercion
date: 2017-10-22 14:28:26
tags: You Don't Know JS
---
## Abstract Value Operations
### JSON.stringify
```
JSON.stringify( undefined ); //"undefined"
JSON.stringify( function(){} ); //"undefined"
JSON.stringify( null ); //"null"
JSON.stringify(
   [1,undefined,function(){},4]
); // "[1,null,null,4]"
JSON.stringify(
   { a:2, b:function(){} }
);// "{"a":2}"

```
> 从上可以看出, 不同类型的值在不同类型中, `JSON.stringify`的结果表现形式也不一样, 在数组的`undefined`, `function`会被转换成`null`, 在对象里值为`undefined, function`的属性会被过滤掉

```
var a = { 
	b: 42,
	c: o,
    d: function(){}
};
// create a circular reference inside `a`
o.e = a;

// would throw an error on the circular reference
// JSON.stringify( a );

// define a custom JSON value serialization
a.toJSON = function() {
    // only include the `b` property for serialization
    return { b: this.b };
};

JSON.stringify( a ); // "{"b":42}"
```
> 如果我们对象引用了自身, 再执行`JSON.stringify`时会报异常, 我们可以给这个对象定义一个`toJSON`的方法, 返回一个默认对象, 来避免这种情况

## Explicit Coercion
### ToNumber
```
Nubmer(true);//1
Nubmer(false);//0
Nubmer(undefined);//NaN
Nubmer(null);//0
Nubmer("");//0
Nubmer("false");//NaN
Nubmer([]);//0
Nubmer([""]);//0
Nubmer(["", ""]);//NaN
Nubmer({});//NaN
Nubmer(function(){});//NaN

var num1 = true + 1; // 1 + 1 = 2
var num2 = false + 1; //0 + 1 = 1
var num3 = undefined + 1; //NaN + 1 = NaN
var num4 = null + 1; // 0 + 1 = 1
```
> 综上可知, `true, false, undefined, null`在数学计算中, 强转后的数值不一, 格外需要注意的是`undefined`会被转换为`NaN`

```
 var a = {
        valueOf: function(){
            return "42";
        },
        toString: function(){
            return "4221";
        }
};
    var b = {
        toString: function(){
            return "42";
        }
};
var c = [4,2];
c.toString = function(){
    return this.join( "" ); // "42"
};
Number( a ); //42
Number( b ); //42
Number( c ); //42
a + '';//42
```
> 如果定义了`valueOf`, `toString`, 在强转的时候, 会优先调用`valueOf`, 如果没有`valueOf`, 再调用`toString`

### ToBoolean
```
Boolean("");//false
Boolean(NaN);//false
Boolean(undefined);//false
Boolean(null);//false
Boolean(0);//false
Boolean(-0);//false

Boolean(new Boolean( false ));//true
Boolean(new Number( 0 ));//true
Boolean(new String( "" ));//true
Boolean("0");//true
Boolean([]);//true
Boolean({});//true
```
> 只有上面几种情况会返回`false`, 其余都会返回`true`

```
Boolean(document.all)//false
```
> `Boolean(document.all)`居然返回false, 似乎和上面的结论矛盾, 其实是因为历史原因, `document.all`其实已经被废弃, 但是又为了兼容有些代码, 浏览器又不能把他删除, 但又不想让`if(document.all){}`这样的代码执行, 因此让`Boolean(document.all)`为`false`, 即使`document.all`返回的是一个类数组

### Explicitly: Parsing Numeric Strings
```
var a = "42";
var b = "42px";
Number( a );    // 42
parseInt( a );  // 42
Number( b );    // NaN
parseInt( b );  // 42
```
> `parseInt`只要解析是值, 第一个是数字就能解析, 否则就返回`NaN`

```
parseInt(1/0, 19);//18
parseInt( 0.000008 );// 0   ("0" from "0.000008")
parseInt( 0.0000008 );// 8   ("8" from "8e-7")
parseInt( false, 16 );// 250 ("fa" from "false")
parseInt( parseInt, 16 );// 15  ("f" from "function..")
parseInt( "0x10" );// 16
parseInt( "103", 2 ); // 2

```
> `parseInt()`的一些奇怪现象

## Implicit Coercion
### Implicitly: Strings <--> Numbers
```
var a = [1,2];
var b = [3,4];
a + b; // "1,23,4"
{} + [];//0
[] + {};//"[object Object]"
```
> 以上是一些比较奇怪的现象

```
 var a = {
        valueOf: function() { return 42; },
        toString: function() { return 4; }
}
a + ""; // "42"
String( a );    // "4"
```
> 以上两者转换是有区别的, `a + ""`这种形式会优先调用`valueOf`方法, 而`String()`直接调用`toString`方法

## Loose Equals Versus Strict Equals
> 我们经常会说`==`表示检测两者的值否是相等, `===`表示检测两者的值相等的同时再检测类型是否相同, 这种说法不是完全正确, 正确理解应该是, `==`会将两值转换成同一类型再进行比较, 而`===`不会将两值类型做转换而直接进行比较
> 当一个`String`类型和一个`Number`类型进行`==`比较, `String`类型会转换为`Number`

```
var num = 42;
num == true;//false ==> 42 == Number(true) ==> 42 == 1
num == false;//false ==> 42 == Number(false) ==> 42 == 0
```
> 上面的比较结果居然都是`false`, 据我们上面分析得知, 42被转换为布尔值为`true`, 但实际上, 当一个`Number`类型和一个`Boolean`类型做比较时, `Boolean`类型会被转换成`Number`类型

```
var str = "42"
num == true;//false ==> Number("42") == Number(true) ==> 42 == 1
num == false;//false ==> Number("42") == Number(false) ==> 42 == 0
```
> 当2个非数字类型的变量进行比较时, 两者都会转换成`Number`类型再进行比较