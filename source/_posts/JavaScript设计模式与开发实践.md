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

### 用单例模式代替once
```js
//让div的click事件只触发一次
var bindEvent = function(){
    $( 'div' ).one( 'click', function(){
        alert ( 'click' ); 
    });
};
var render = function(){ 
    console.log( '开始渲染列表' ); 
    bindEvent();
};
render();
render();

//下面利用单例模式实现
var bindEvent = getSingle(function(){ 
    document.getElementById( 'div1' ).onclick = function(){
        alert ( 'click' ); 
    }
    return true; 
});

var render = function(){ 
    console.log( '开始渲染列表' ); 
    bindEvent();
};

render(); 
render();
```

## 策略模式
> 策略模式指的是定义一系 列的算法，把它们一个个封装起来。将不变的部分和变化的部分隔开是每个设计模式的主题，策 略模式也不例外，策略模式的目的就是将算法的使用与算法的实现分离开来。

```js
//案例
var calculateBonus = function( performanceLevel, salary ){
    if ( performanceLevel === 'S' ){ 
        return salary * 4;
    }
    if ( performanceLevel === 'A' ){ 
        return salary * 3;
    }
    if ( performanceLevel === 'B' ){ 
        return salary * 2;
    } 
};

calculateBonus( 'B', 20000 ); // 输出:40000
calculateBonus( 'S', 6000 ); // 输出:24000

//使用策略模式
var strategies = {
    "S": function( salary ){
        return salary * 4; 
    },
    "A": function( salary ){ 
        return salary * 3;
    },
    "B": function( salary ){
        return salary * 2; 
    }
};
var calculateBonus = function( level, salary ){ 
    return strategies[ level ]( salary );
};
console.log( calculateBonus( 'S', 20000 ) ); // 输出:80000 
console.log( calculateBonus( 'A', 10000 ) ); // 输出:30000
```

## 代理模式
> 代理模式包括许多小分类，在 JavaScript 开发中最常用的是虚拟代理和缓存代理

```js
//不用代理模式  下面代码只能实现预加载, 如果哪天不需要预加载了 整个代码都需要改变
var MyImage = (function(){
    var imgNode = document.createElement( 'img' ); 
    document.body.appendChild( imgNode );
    var img = new Image;
    img.onload = function(){ 
        imgNode.src = img.src;
    };
    return {
        setSrc: function( src ){
            imgNode.src = 'file:// /C:/Users/svenzeng/Desktop/loading.gif';
            img.src = src; 
        }
    } 
})();

MyImage.setSrc( 'http:// imgcache.qq.com/music/photo/k/000GGDys0yA0Nk.jpg' );

//使用代理模式 
var myImage = (function(){
    var imgNode = document.createElement( 'img' ); 
    document.body.appendChild( imgNode );
    return {
        setSrc: function( src ){
        imgNode.src = src; }
    }
})();
var proxyImage = (function(){ 
    var img = new Image; 
    img.onload = function(){
        myImage.setSrc( this.src ); 
    }
    return {
        setSrc: function( src ){
            myImage.setSrc( 'file:// /C:/Users/svenzeng/Desktop/loading.gif' );
            img.src = src; 
        }
    } 
})();

proxyImage.setSrc( 'http:// imgcache.qq.com/music/photo/k/000GGDys0yA0Nk.jpg' );
```

### 缓存代理
```js
/**************** 计算乘积 *****************/ 
var mult = function(){
    var a = 1;
    for (var i = 0, l = arguments.length; i < l; i ++) {
        a *= arguments[i];
    }

    return a;
}
 /**************** 计算加和 *****************/
var plus = function(){
    var a = 1;
    for (var i = 0, l = arguments.length; i < l; i ++) {
        a += arguments[i];
    }

    return a;
}

/**************** 创建缓存代理的工厂 *****************/ 
var createProxyFactory = function( fn ){
    var cache = {};
    return function(){
        var args = Array.prototype.join.call( arguments, ',' );
        if ( args in cache ){
            return cache[ args ]; 
        }
        return cache[ args ] = fn.apply( this, arguments ); 
    }
};

var proxyMult = createProxyFactory( mult ), 
proxyPlus = createProxyFactory( plus );

alert ( proxyMult( 1, 2, 3, 4 ) );//输出:24
alert ( proxyPlus( 1, 2, 3, 4 ) );//输出:10
```