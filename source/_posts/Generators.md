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

```
function* test() {
	yield 1;
	yield 2;
	yield 3;
}

var it = test();
for (var v of it) {
	console.log(v)
}
```
> `for (var v of it)` 会自动执行迭代器的next方法, 直到返回的对象`done: true`

```
for (var v of it) {}
//可以转化为如下
for (var ret; (ret = it.next()) && !ret.done; ) {}
```

```
var arr = [1, 3, 5, 7];
for (var v of arr) {
	console.log(v);//1 3 5 7
}
// 等价于如下
var it = arr[Symbol.iterator]()
it.next();//{value: 1, done: false}
it.next();//{value: 3, done: false}
```

```
function* test() {
	try {
		var nextVal = 1;
		while (true) {
			nextVal += 3;
			yield nextVal;
		}
	} finally {
		console.log('finally...')
	}
}

var it = test();
for (var v of it) {
	console.log(v)
	if (v > 30 ) {
		var result = it.return("hello world")
		console.log("result: ", result)
	}
}
/**
 * 输出结果: 1 4 .... finally... result: {value: "hello world", done: true}
 */
```
> 当我们调用迭代器it的return方法时, 迭代器会将it.next的结果done设置为true, 来终端迭代器迭代下去, 然后会执行迭代器里的finally方法(如果有), 最后返回的value的值等于我们return传递进去的值