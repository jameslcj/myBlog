---
title: BOM
date: 2017-07-11 10:30:24
tags: javaScript高级程序设计笔记
---
## window
### 全局作用域
> 所有全局作用域下声明的变量和函数都会变成window的属性和方法

```
var test = 'test';
function getTest() {
  console.log(this.test);
}
console.log(window.test);//"test"
getTest();//"test"
window.getTest();//"test"
```

> 全局变量与直接在window上定义的变量的还是有区别的;
直接定义在window上的变量可以被delete, 但全局变量不能;
根本原因是因为通过var声明的变量有一个名为[Configurable]的特性, 这个特性的值被设置为false, 所以无法删除;

```
var test1 = 'hello';
window.test2 = 'world';
delete window.test1; //IE < 9 直接报错, 其他返回false;
delete window.test2; //IE < 9 直接报错, 其他返回true;
console.log(window.test1);//"hello"
console.log(window.test2);//undefined
```

> 如果直接访问一个没有声明的变量会报错, 但是我们可以通过window的属性方式来访问, 避免报错

```
console.log(someVar);//ERROR
console.log(window.someVar);//undefined
```
