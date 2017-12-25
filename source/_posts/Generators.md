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


```
function ajax(delay) {
    return new Promise(function(resolve, reject) {
        //延迟返回 模拟异步
        setTimeout(() => {
            resolve("ajax data");
        }, delay)
    })
}

function *test(arg) {
    var ajaxRes = yield ajax(arg);
    return yield ajaxRes;
}
/**
 * 模拟co库写的一个迭代器递归执行方法
 **/
function co(gen) {
    //获取初始化参数
    var args = [].slice.call(arguments, 1);
    //使用当前作用域初始化gen获取迭代器对象it, 下面it会作为一个闭包对象一直被递归调用
    var it = gen.apply(this, args);
    //返回一个处理好的promise
    return Promise.resolve().then(function handleNext(value) {
        //将上一次promise返回的结果作为参数传递给迭代器, 第一次value为undefined, 一般浏览器会忽略
        var next = it.next(value);
        console.log("next: ", next);
        
        //函数自运行
        return (function handleResult(next) {
            //判断迭代器是否完成 如果完成返回最后结果
            if (next.done) {
                return next.value;
            } else if (typeof next.value == 'function') {
				//处理thunk类型的回调
				//返回一个promise
				return new Promise(function(resolve, reject) {
					//给thunk传递回调函数
					next.value(function(err, msg) {
						if (err) {
							reject(err);
						} else {
							resolve(msg);
						}
					})
				}).then(function() {
                    //如果promise返回成功, 就递归调用handleNext来处理it迭代器至完成
                    handleNext,

                    //异常处理
                    function handleErr(err) {
                        return Promise.resolve(
                            //处理异常
                            it.throw(err)
                        ).then(
                            //继续处理异常处理的结果
                            handleResult
                        )
                    }

				})
			} else {
                //否则就递归调用迭代器
                //通过promise处理next.value, 如果next.value是非promise对象就会直接进入then, 否则就等待promise回调then
                //这里的next.value一般都是异步处理, 例如ajax操作
                return Promise.resolve(next.value).then(
                    //如果promise返回成功, 就递归调用handleNext来处理it迭代器至完成
                    handleNext,
					
                    //异常处理
                    function handleErr(err) {
                        return Promise.resolve(
                            //处理异常
                            it.throw(err)
                        ).then(
                            //继续处理异常处理的结果
                            handleResult
                        )
                    }
                );
            }
        })(next)
    })
}
co(test, 3000).then(function(res) {
    console.log("res: ",res)
});
```
> 上面是一个自动递归处理迭代器的方法, 方法类似co库

```
funcion *foo() {
	var r1 = yield ajax('url1');
	var r2 = yield ajax('url2');

	var r3 = yield ajax(r1+r2);
	return r3;
}
co(foo);

//可以优化为下面
funcion *foo() {

	var p1 = ajax('url1');
	var p2 = ajax('url2');
	var r1 = yield p1
	var r2 = yield p2

	var r3 = yield ajax(r1+r2);
	return r3;
}
co(foo);
// 上面那方法可以理解如下
funcion *foo() {
	var results = yield Promise.all([ajax('url1'), ajax('url2)]);
	var r1 = results[0];
	var r2 = results[1];

	var r3 = yield ajax(r1+r2);
	return r3;
}
co(foo);
```
> 上面第一个demo, 按照迭代器的方式, 必须等r1的ajax返回结果后, 才会调用r2的ajax, 这样效率就会很低, 所以应该优化为下面那方法, 这样就可以先直接发送2个ajax请求并发, 处理结果使用迭代器