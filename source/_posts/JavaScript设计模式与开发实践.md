---
title: JavaScript设计模式与开发实践
date: 2018-02-23 20:54:49
tags: JavaScript设计模式与开发实践
---
## 高阶函数实现AOP
> AOP(面向切面编程)的主要作用是把一些跟核心业务逻辑模块无关的功能抽离出来，这些 跟业务逻辑无关的功能通常包括日志统计、安全控制、异常处理等

```js
Function.prototype.before = function( beforefn ) {
    var __self = this; // 保存原函数的引用
    return function(){ // 返回包含了原函数和新函数的"代理"函数
        beforefn.apply( this, arguments ); // 执行新函数，修正 this
        return __self.apply( this, arguments );// 执行原函数
    }
}
Function.prototype.after = function( afterfn ){ 
    var __self = this;
    return function(){
        var ret = __self.apply( this, arguments ); //执行前面的函数
        afterfn.apply( this, arguments ); //调用最后的函数
        return ret;
    } 
};
var func = function(){ 
    console.log( 2 );
};
func = func.before(function(){ 
    console.log( 1 );
}).after(function(){ 
    console.log( 3 );
}); 
func();
```

## uncurrying
```js
Function.prototype.uncurrying = function(){
    var self = this; 
    return function(){
        var obj = Array.prototype.shift.call( arguments );
        return self.apply( obj, arguments );
    }
};
var push = Array.prototype.push.uncurrying();
(function(){
    push( arguments, 4 );
    console.log( arguments ); // 输出:[1, 2, 3, 4]
})( 1, 2, 3 );

```
```js
//第二种实现
Function.prototype.uncurrying = function(){
    var self = this; 
    return function(){
        return Function.prototype.call.apply( self, arguments ); 
    }
};
```

## 函数节流
> 一次性渲染太多影响性能, 所以分批进行渲染
```js
var timeChunk = function( ary, fn, count ){
    var obj, t;
    var len = ary.length; 
    var start = function(){
        for( var i = 0; i < Math.min( count || 1, ary.length ); i++ ){ 
            var obj = ary.shift();
            fn( obj );  
        } 
    };
    
    return function(){
        t = setInterval(function(){
        if ( ary.length === 0 ){ // 如果全部节点都已经被创建好
            return clearInterval( t ); }
            start();
        }, 200 ); // 分批执行的时间间隔，也可以用参数的形式传入
    }; 
 };

var ary = [];
for ( var i = 1; i <= 1000; i++ ){ 
    ary.push( i );
};
var renderFriendList = timeChunk( ary, function( n ){ 
    var div = document.createElement( 'div' ); 
    div.innerHTML = n;
    document.body.appendChild( div );
}, 8 ); 
renderFriendList();
```

## 单列模式
```js
var getSingle = function( fn ){
    var result;
    return function(){
        return result || ( result = fn .apply(this, arguments ) );
    }
};
var createLoginLayer = function() {}//创建登陆框的逻辑
var createSingleLoginLayer = getSingle( createLoginLayer );
```