---
layout: Program Performance
title: Performance
date: 2017-12-30 16:06:39
tags: You Don't Know JS
---
## Web Worker
> JavaScript是一门单线程语言, 我们可以通过浏览器的`Work`, 启动多线程来提高性能来处理io密集型计算, 大数据处理等; 但是主进程和子线程是隔离的, 只能通过监听事件和postMessage的方式进行传递消息

```
//在主线程启动线程 注意有跨域问题 而且必须是js可执行脚本
var w1 = new Worker( "http://127.0.0.1:8080/test.js" );
//监听事件, 就可以收到子线程的发送消息
w1.addEventListener("message", function(e) {
    console.log("main process receive: ", e.data)
})
//对子线程发送信息
w1.postMessage("hello world")

// test.js
console.log("test.js....", navigator, location, JSON)
addEventListener( "message", function(evt){
    // evt.data
    console.log("receive: ", evt.data)
    postMessage( "message has received" );
} );
importScripts("test2.js")
```

### Work 中可以使用如下全局变量

- navigator
- location
- JSON
- importScripts //同步阻塞引入其他脚本

### Shared Workers

通浏览器同时打开多个tab(同一个页面), 就会同时打开多个work, 此时我们可以用共享work来减少性能资源消耗
```
var w1 = new SharedWorker( "http://127.0.0.1:8080/mySharedWorker.js" );

// 通过port对象作为唯一标识
w1.port.addEventListener( "message", handleMessages );
w1.port.postMessage( "something cool" );

//port connect initialized
w1.port.start();
```

在shared worker 内部给来的请求分配一个端口
```
// inside the mySharedWorker.js
addEventListener("connect", function (event) {
    //分配端口给这个链接
    var port = event.ports[0];

    port.addEventListener("message", function (event) {
        // ....
        port.postMessage( .. );
    })

    // initialize the port connection
    port.start();
})
```