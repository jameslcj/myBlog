---
title: api
date: 2017-09-24 20:52:11
tags: javaScript高级程序设计笔记
---
## Page Visibility
- document.hidden 表示页面是否隐藏
- document.visibilityState: 有4个状态的值
- visibiliteChagne事件

```
document.addEventListener('visibiliteChagne', function(event) {
	if (document.hidden) {
		//do something
	}
})
```

## Geolocation API
> 获取用户地理位置

```
navigator.geolocation.getCurrentPosition(function(position) {
	console.log(position.coords.latitude, position.coords.longitude);
}, function(error) {
	console.log(error)
}, {
	enableHighAccuracy: true, //尽可能使用精确的位置信息
	timeout: 5000,//等待位置信息的最长时间
	maximumAge: 25000//上一次获取的坐标信息的有效时间
})
```

- watchPosition方法和getCurrentPosition类似, 但是它会定时调用getCurrentPosition的效果相同
- clearWatch 取消监听

```
var watchId = navigator.geolocation.watchPosition方法和getCurrentPosition类似(function(position) {
	console.log(position.coords.latitude, position.coords.longitude);
}, function(error) {
	console.log(error)
})

//取消监听
clearWatch(watchId);

```

## FileReader 类型
> 可以异步读取上传文件中的数据

- readAsText(file, encoding) 以纯文本形式读取文件
- readAsDataURL(file) 读取文件以URI的形式保存
- readAsBinaryString(file) 获取字符串, 每个字符表示一个字节
- readAsArrayBuffer(file) 获取ArrayBuffer类型的内容

![应用](https://img.alicdn.com/tfs/TB1b_bwggoQMeJjy0FoXXcShVXa-1224-1480.png)

## 对象URL
> 可以不必将数据内容读取到JavaScript中而直接使用, 可以使用`window.URL.createObjectURL()`方法, 返回字符串, 指向内存地址

```
var url = window.URL.createObjectURL(files[0])
var output.innerHTML = '<img src="' + url + '">';

//释放内存
window.URL.revokeObjectURL(url);
```

## 读取拖放的文件
> html5可以直接拖入文件, 可以通过`files = event.dataTransfer.files;`获取拖入文件并获取文件信息

```
var droptarget = document.getElementById("droptarget");
function handleEvent(event) {
	var info = "",
		output = document.getElemntById("output"),
		files, i, len;

	EvenUtil.preventDefault(event);
	if (event.type == drop) {
		files = event.dataTransfer.files;
		i = 0;
		len = files.length;

		while (i < len) {
			//获取文件信息
		}
	}
}
EvenUtil.addHandle(droptarget, "dropenter", handleEvent);
EvenUtil.addHandle(droptarget, "dropover", handleEvent);
EvenUtil.addHandle(droptarget, "drop", handleEvent);
```
