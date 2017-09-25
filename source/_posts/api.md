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