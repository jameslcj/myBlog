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

### dataTransfer 对象
> 在触发拖放事件时, 可以通过这个对象读取或者设置数据

- dataTransfer.setData(text/URL, 'str');
- dataTransfer.getData(text/URL)

```
event.dataTransfer.setData("text", "some text");
var text = event.dataTransfer.getData("text");

event.dataTransfer.setData("URL", "http://test.com");
var url = event.dataTransfer.getData("URL");

// 为了浏览器兼容 已写成如下
var dataTransfer = event.dataTransfer;
var url = dataTransfer.getData("url") || dataTransfer.getData("text/uri-list");
var text = dataTransfer.getData("Text");
```

### dropEffect 与 effectAllowed
![dropEffect 与 effectAllowed](https://img.alicdn.com/tfs/TB1Zg9sa_ZRMeJjSspkXXXGpXXa-1992-1412.png)

### 可拖动
> 设置`draggable="true"`

```
<div style="width: 100px;height: 100px;background-color:red;" draggable="true"></div>
```

### 其他成员

![其他成员方法](https://img.alicdn.com/tfs/TB1YLiBa3MPMeJjy1XbXXcwxVXa-1938-1418.png)

## 媒体元素
```
//视频
<video src="video.mpg"></video>
//音频
<audio src="song.mp3"></audio>
```

> 不是所有浏览器都支持所有格式 可以使用`<source>`

```
<video>
	<source src="video.webm" type="video/webm">
	<source src="video.ogv" type="video/ogg">
</video>
```

### 属性
![属性](https://img.alicdn.com/tfs/TB1nzyya.gQMeJjy0FiXXXhqXXa-1408-1478.png)

### 事件
![事件](https://img.alicdn.com/tfs/TB1WR9Da3MPMeJjy1XbXXcwxVXa-1448-1310.png)

## 历史状态管理
> 现在很多页面都是利用hashchange事件, 改变页面, 所以我们可以通过监听此事件, 向history添加历史, 这样历史的前进和后退又可以用了

- history.pushState(状体对象, 新状态的标题, 可选的相对url) 添加历史
- history.popstate() 后退
- history.replaceState(状体对象, 新状态的标题) 更新替换
