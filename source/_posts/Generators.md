---
title: Generators
date: 2017-12-02 14:17:01
tags: You Don't Know JS
---
```
function *foo(x) {
	var y = x * (yield function() {return "hello world";}); 
	return y;
}
var it = foo( 6 ); //将6作为参数调用foo方法 并返回一个迭代器对象
// start `foo(..)`
var res1 = it.next(); //开始执行函数, 直到碰到yield 就停住了, 等待下一步指令; 并返回一个迭代器对象值为 {done: false, value: ƒ}, done表示迭代器还没有结束, value表示yield后面的内容; 一般浏览器都会忽略掉第一个next传入的参数
res1.value(); //上面yield后面的内容是一个函数, 所以调用后, 返回"hello world"
var res2 = it.next( 7 ); //将7 作为yield的返回值, 并继续执行至下一个yield或是结束
res2.value; //因此 6 * 7 = 42
```
