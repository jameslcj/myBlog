---
title: DOM扩展
date: 2017-08-02 09:53:53
tags: javaScript高级程序设计笔记
---
## 选择符API
### querySelector()方法
> 可以接受一个css选择符, 返回匹配的第一个元素, 如果没有找到匹配的元素, 返回null

```
//获取body
document.querySelector("body");

//获取id为div的元素
document.querySelector("#div");
```

### querySelectorAll()方法
> 可以接受一个css选择符, 返回所有匹配的第一个元素, 如果没有找到匹配的元素, 返回null

```
//获取所有类为myClass的元素
document.querySelectorAll(".myClass");

//获取所有div下的所有p元素
document.querySelectorAll("div p");
```

### matchesSelector()方法
> 选择的元素是否与之匹配, 匹配返回true, 否则返回false; 目前不是所有浏览器支持, 需要添加浏览器标识

```
if (document.body.matchesSelector && document.body.matchesSelector("body")) {
	//do someting
} else if (document.body.webkitMatchesSelector && document.body.webkitMatchesSelector("body")) {
	//do someting
} else if (document.body.msMatchesSelector && document.body.msMatchesSelector("body")) {
	//do someting
} else if (document.body.mozMatchesSelector && document.body.mozMatchesSelector("body")) {
	//do someting
}
```

## 元素遍历
> 因为ie9以前是版本, 对于元素间的空格, 不会作为节点返回, 因此childNodes和firstChild等元素在不同浏览器下结果不一样, 所以w3c又引入了新的属性来获取元素

- childElementCount 返回子元素个数(不包含空格和注释)
- firstElementChild 返回第一个子元素
- lastElementChild 返回最后一个子元素
- previousElementSibling 返回前一个同辈元素
- nextElementSibling 返回后一个同辈元素

```
//之前的做法
var child = element.firstChild;
while (child != element.lastChild) {
	if (child.nodeType == 1) {
		processChild(child)
	}
	child = child.nextSibling;
}

//现在用新的属性, 可简化如下
var child = element.firstElementChild;
while (child != element.lastElementChild) {
	processChild(child)
	child = child.nextElementSibling;
}

```

## HTML5
### 与类相关的扩充
- getElementsByClass()方法
	+ 类名选择器
	+ 返回nodeList, 所以和其他返回nodeList的选择器一样有性能问题
	+ IE9+支持
- classList属性
	+ add(类名) 添加类名
	+ remove(类名) 删除类名
	+ contains(类名) 判断是否包含类名
	+ toggle(类名) 有对应类名就删除, 没有就添加
	+ Firefox3.6+和chrome

### 焦点管理
- document.activeElement 获取文档焦点对象

```
var btn = document.getElementById("btn");
btn.focus();
btn === document.activeElement; //true
```

- document.hasFocus() 判断文档是否获取焦点对象

```
var btn = document.getElementById("btn");
btn.focus();
document.hasFocus(); //true
```

### HTMLDocument的扩展
- readyState 属性 判断文档是否加载完毕
	+ 其值为loading, 正在加载文档
	+ 其值为complete, 已经加载完文档

```
if (document.readyState == 'complete') {
	//do something
}
```

- compatMode 渲染模式
	+ CSS1Compat 标准模式
	+ BackCompat 兼容模式

- head 属性 
```
var head  = document.head || document.getElementsByTagName("head")[0]
```

### 字符集属性
- document.charset 当前浏览器字符集
- document.defaultCharset 浏览器或系统默认字符

### 自定义数据属性
- dataset
```
<div data-id="123" data-name="hello"></div>
var div = document.getElementsByTagName("div")[0];
div.dataset.id = "321";
div.dataset.name; //"hello"
```

### 插入标记
- innerHTML 属性
- outerHTML 属性
- insertAdjacentHTML() 方法, 接受2个参数, 第一个参数要插入的位置, 第二个参数要插入的html文本
第一个参数为下:
	+ "beforebegin" 在当前元素之前插入一个紧邻的同辈元素
	+ "afterbegin" 在当前元素之后插入一个新的子元素或在第一个子元素之前再插入新的子元素
	+ "beforeend" 在当前元素之后插入一个新的子元素或在最后一个子元素之后再插入新的子元素
	+ "afterend" 在当前元素之后插入一个紧邻的同辈元素
- 内存与性能问题

> 以上方法有可能导致内存泄露, 因为替换了dom节点后, 之前的dom节点还会在内存中, 因此如果多次替换节点时, 应该先删除要被替换的节点;
当插入大量html标签时, 建议先拼接好html, 再进行innerHTML或者outerHTML属性, 不要频繁使用, 效率较低