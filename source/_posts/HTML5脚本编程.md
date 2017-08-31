---
title: HTML5脚本编程
date: 2017-08-31 09:56:59
tags: javaScript高级程序设计笔记
---
## 跨文档消息传递
> postMessage传递的消息, 最早只支持字符串, 后来有些浏览器支持了任何数据结构, 但为了保证安全建议`JSON.stringify`转换为字符串再传递

- postMessage(一条消息, 消息接受方来自哪个域的字符串) 发送消息
- onmessage(data, origin, source) 接受消息

```
var iframe = document.getElementById("#ifrmae");
iframe.postMessage("some data", "http://test.com");

window.onmessage = function(data, origin, source) {
	console.log(data, origin, source)
}
```

## 原生拖放
### 拖放事件
- dragstart
- drag
- dragend

> 当元素拖动到另一个有效的放置目标上时, 会触发以下事件

- dragenter
- dragover
- dragleave 或 drop

### 自定义放置目标

> 元素默认是不允许放置的, 可用阻止默认事件来允许放置

```
droptarget.ondragover = function(event) {
	event.preventDefault();
}
droptarget.ondragenter = function(event) {
	event.preventDefault();
}
```

### 