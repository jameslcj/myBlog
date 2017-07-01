---
title: JavaScript坑总结
date: 2016-12-22 19:36:38
tags: JavaScript
---
### 作用域
- 通过函构造函数创建的函数的[[scope]]属性总是唯一的全局对象
```JavaScript
var x = 10;
 
function foo() {
 
  var y = 20;
 
  function barFD() { // 函数声明
    alert(x);
    alert(y);
  }
 
  var barFE = function () { // 函数表达式
    alert(x);
    alert(y);
  };
 
  var barFn = Function('alert(x); alert(y);');
 
  barFD(); // 10, 20
  barFE(); // 10, 20
  barFn(); // 10, "y" is not defined
 
}
 
foo();
```
- Function构造不同的是eval()可以干扰作用域链，而Function()更安分守己些。不管你在哪里执行 Function()，它只看到全局作用域。所以其能很好的避免本地变量污染
```
var global = “global";
(function () {
   var local = 1;
   Function("console.log(typeof local, global);")(); // undefined global
}());
```

- 如果一个属性在对象中没有直接找到, 会现在全局查找, 如果还没找到，查询将在原型链中继续
```JavaScript
function foo() {
  alert(x);
}
 
Object.prototype.x = 10;
 
foo(); // 10

// -----

function foo() {
  alert(x);
}

var x = 12 
Object.prototype.x = 10;
 
foo(); // 12
```

- with 声明临时调整的变量, 在跳出with后不会生效, 其他变量在with中修改有效
```JavaScript
var x = 10, y = 10;
 
with ({x: 20}) {
 
  var x = 30, y = 30;
 
  alert(x); // 30
  alert(y); // 30
}
 
alert(x); // 10
alert(y); // 30
```
### this指向谁
```JavaScript
var foo = {
    bar: function(){return this.a},
    a: 1
}
var f = foo.bar();
foo.bar() // 1
f() //undefined
```
- foo.bar() 调用者是foo 所以this指向foo的a
- f() 调用者是window 所以是undefined ===> window.f() 

```JavaScript
var foo = {
  bar: function () {
    console.log(this);
  }
};
 
foo.bar(); // Reference, OK => foo
(foo.bar)(); // Reference, OK => foo
 
(foo.bar = foo.bar)(); // global?
(false || foo.bar)(); // global?
(foo.bar, foo.bar)(); // global?
```

### 是不是实例
```JavaScript
function f(){}
new f() instanceof f  // true

function f(){ return f;}
new f() instanceof f  // false 
```

### 函数的length
- 函数的形参就是length的值
- arguments 是实参的个数
```JavaScript
function test(a, b, c){}
test.length//3
```

### with关键字
- with的作用就是临时改变作用域
- 将`__proto__`属性临时指向`Object.prototype`
- 性能较差
```JavaScript
var x = 20, y = 20, z = 30;
var obj = {
    x: 1,
    y: 2,
}
with(obj) {
    console.log(x, y, z) //1, 2, 30
}
```
### 内存泄漏
- 这个多余的g函数就死在了返回函数的闭包中了，因此内存问题就出现了。这是因为if语句内部的函数与g是在同一个作用域中被声明的。这种情况下 ，除非我们显式断开对g函数的引用，否则它一直占着内存不放。
```
var f = (function(){
    var f, g;
    if (true) {
      f = function g(){};
    }
    else {
      f = function g(){};
    }
    // 设置g为null以后它就不会再占内存了
    g = null;
    return f;
  })();
```
### Object.preventExtensions() Object.seal() Object.freeze()的区别
    Object.preventExtensions() 对象不可扩展, 即不可以新增属性或方法, 但可以修改/删除
    Object.seal() 在上面的基础上，对象属性不可删除, 但可以修改
    Object.freeze() 在上面的基础上，对象所有属性只读, 不可修改
### 数组
- slice()方法实现的是浅拷贝
    + MDN上的解释: For object references (and not the actual object), slice copies object references into the new array. Both the original and new array refer to the same object. If a referenced object changes, the changes are visible to both the new and original arrays.
- push() 
  + 对一个数组push数组, 应该先使用concat或者slice数组在push进去, 不然只是引用关系
```JavaScript
var arr = [{a: 1}]
var arr2 = arr.slice()
arr[0].a = 2
arr2[0].a // 2
//---
var arr = [1, 2];
var allArr = [];
allArr.push(arr);
arr.push(3);
allArr[0] // [1, 2, 3]

var arr = [1, 2];
var allArr = [];
allArr.push(arr.concat();
arr.push(3);
allArr[0] // [1, 2]
```

- 数组引用
```JavaScript
    var arr = [1, 2, 3];
    //这样不进行拷贝 只是引用关系
    var arr2 = arr;
    arr2.push(4);
    console.log(arr) //1, 2, 3, 4
    console.log(arr2) //1, 2, 3, 4
    
    //这样arr 与 arr2就没有引用关系了
    arr2 = [1, 2, 3];
    console.log(arr) //1, 2, 3, 4
    console.log(arr2) //1, 2, 3
```

- Array() 与 Array.of() 的区别
```JavaScript
Array(2) // [, ,]
Array(2,3) //[2, 3]

Array.of(2) //[2]
Array.of(2,3 ) // [2, 3]
```
- map与forEach 的区别
    + map会根据返回的值改变数组对应的值
    + forEach只是单纯的执行一遍函数不会修改数组
```JavaScript
var arr = [1, 2, 3, 4, 5];
function test(num) {
    return num *2;
}
arr.forEach(test); //[1, 2, 3, 4, 5]
arr.map(test); // [2, 4, 6, 8, 10]
```

### 正则
- (?=) 前向声明
```JavaScript
//需求: 匹配到ab 才能将a换成* 这时就需要用到前向声明了
var str = 'abacadab';
var reg = /a(?=b)/g
str.replace(reg, '*')//*bacad*b
```
- (?!) 反前向声明
```JavaScript
//需求: 匹配到a 但后面不是b的情况下 才能将a换成* 这时就需要用到反前向声明了
var str = 'abacadab';
var reg = /a(?!b)/g
str.replace(reg, '*')//ab*c*dab
```
- 前向声明和反前向声明的应用
```JavaScript
function test4(str) {
    // 声明后面的 (?!\b) 不能以空格开头 必须是3位数字为一组一直到结束
	var reg = /(?=(?!\b)(\d{3})+$)/g;
	return str.replace(reg, ',')
}
test('1234567') //1,234,567
```

