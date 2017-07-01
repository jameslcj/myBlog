---
title: JavaScript原型和原型链
date: 2017-01-18 20:14:42
tags: JavaScript
---
### 原型继承方法
- 原型继承有如下2种方式:
```
var BaseCalculator = function() {
    this.decimalDigits = 2;
};

BaseCalculator.prototype = {
    add: function(x, y) {
        return x + y;
    },
    subtract: function(x, y) {
        return x - y;
    }
};
var Calculator = function () {
    //为每个实例都声明一个税收数字
    this.tax = 5;
};
// 原型继承有如下2种方式:
Calculator.prototype = new BaseCalculator();
Calculator.prototype = BaseCalculator.prototype;

var calc =newCalculator();
alert(calc.add(1, 1));
alert(calc.decimalDigits);
```
- 2种继承方式的区别: 
    + 第一种继承要是能够访问到calc.decimalDigits的值 
    + 而第二种无法访问到calc.decimalDigits; 
    + 因为第一种是将 BaseCalculator的实例赋值给Calculator.prototype, 所以能够访问到其对应的属性, 并且所有Calculator都公用一个BaseCalculator实例, 而第二种没有实例, 所以无法访问

### 如何判断对象是否有对应的属性比较安全
- 对象的hasOwnProperty方法, 可以用来判别是否是自己上的属性或方法, 
- 但是对象的hasOwnProperty很容易被修改, 所以我们应该使用
{}.hasOwnProperty.call(foo, 'bar’);来判断 更加可靠安全

### __proto__属性
- 每个对象都有一个`__proto__`隐式属性, 他指向该对象的原型, 既`prototype`
- 可以直接用`__proto__`属性改变实例对象的原型链

```
var a = {
  x: 10,
  calculate: function (z) {
    return this.x + this.y + z
  }
};
 
var b = {
  y: 20,
  __proto__: a
};
var c = {
  y: 30,
  __proto__: a
};
 
// 调用继承过来的方法
b.calculate(30); // 60
c.calculate(40); // 80

var d = {
    x: 20,
    calculate: function (z) {
        return this.x + this.y + z
    }
}
c.__proto__ = d;
c.calculate(40); //90
```

### with会改变作用域
- with会把原型链`__proto__`改变到`Object.prototype`上
- 每个上下文执行环境都有变量对象(variable object)，this指针(this value)，作用域链(scope chain) 这3个属性
- 在代码执行过程中，如果使用with或者catch语句就会改变作用域链。而这些对象都是一些简单对象，他们也会有原型链。这样的话，作用域链会从两个维度来搜寻。
    + 首先在原本的作用域链
    + 每一个链接点的作用域的链（如果这个链接点是有prototype的话）

```
Object.prototype.x = 10;
 
var w = 20;
var y = 30;
 
// 在SpiderMonkey全局对象里
// 例如，全局上下文的变量对象是从"Object.prototype"继承到的
// 所以我们可以得到“没有声明的全局变量”
// 因为可以从原型链中获取
 
console.log(x); // 10
 
(function foo() {
 
  // "foo" 是局部变量
  var w = 40;
  var x = 100;
 
  // "x" 可以从"Object.prototype"得到，注意值是10哦
  // 因为{z: 50}是从它那里继承的
 
  with ({z: 50}) {
    console.log(w, x, y , z); // 40, 10, 30, 50
  }
 
  // 在"with"对象从作用域链删除之后
  // x又可以从foo的上下文中得到了，注意这次值又回到了100哦
  // "w" 也是局部变量
  console.log(x, w); // 100, 40
 
  // 在浏览器里
  // 我们可以通过如下语句来得到全局的w值
  console.log(window.w); // 20
 
})();
```

### AO变量对象

- 当进入执行上下文(代码执行之前)时，VO里已经包含了下列属性
    + 函数的所有形参(如果我们是在函数执行上下文中)
    + 由名称和对应值组成的一个变量对象的属性被创建；没有传递对应参数的话，那么由名称和undefined值组成的一种变量对象的属性也将被创建。
    + 所有函数声明(FunctionDeclaration, FD)—由名称和对应值（函数对象(function-object)）组成一个变量对象的属性被创建；如果变量对象已经存在相同名称的属性，则完全替换这个属性。
    + 所有变量声明(var, VariableDeclaration)—由名称和对应值（undefined）组成一个变量对象的属性被创建；如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性。
    + 函数表达式是不在AO对象里生成变量, 但是把函数表达式赋值给一个声明的变量, 那边这个变量会以undefined的方式存放在AO对象里

### 全局变量和var变量的区别
- 关于变量，还有一个重要的知识点。变量相对于简单属性来说，变量有一个特性(attribute)：{DontDelete},这个特性的含义就是不能用delete操作符直接删除变量属性。
```
a = 10;
alert(window.a); // 10
 
alert(delete a); // true
 
alert(window.a); // undefined
 
var b = 20;
alert(window.b); // 20
 
alert(delete b); // false
 
alert(window.b); // still 20

但是这个规则在有个上下文里不起走样，那就是eval上下文，变量没有{DontDelete}特性。
eval('var a = 10;');
alert(window.a); // 10
 
alert(delete a); // true
 
alert(window.a); // undefined

```


