---
title: DOM2和DOM3
date: 2017-08-05 14:18:55
tags: javaScript高级程序设计笔记
---
## 检测DOM等级方法
- document.implementation.hasFeature(属性名, 等级)
```
document.implementation.hasFeature("MouseEvents", "2.0");//检测MouseEvents是否支持DOM2.0
```

## DOM变化
### document类型的变化

- createElementNS(namespaceURI, tagName) 使用给定的tagName创建一个属于命名空间为namespaceURI的新元素
- createAtrributeNS(namespaceURI, attributeName) 使用给定的attributeName创建一个属于命名空间namespaceURI的新特性
- getElementsByTagNameNS(namespaceURI, tagName) 返回属于命名空间namespaceURI的tagName元素的NodeList

```
var svg = document.createElementNS("http://www.w3.org/2000/svg", "svg");
var attr = document.createAtrributeNS("http://wwww.somewhere.com", "random");
var elements = document.getElementsByTagNameNS("http://www.w3.org/2000/xhtml", "*");
```

- importNode(节点对象, 是否深复制) 这个方法功能和cloneNode()非常类似, 使用cloneNode()从别的文档树种拷贝的节点, 会导致错误, 所以使用importNode, 就能保证拷贝过来的新节点的onwerDocuemnt属性, 是指向当前文档树的

```
var newNode = document.importNode(oldNode, true);
document.body.appendChild(newNode);
```

- document.implementation.createHTMLDocument(新标题) 创建一个完整的html文档

```
var newHtml = document.implementation.createHTMLDocument("new title");
newHtml.title; //new title
typeof newHtml.body; //object
```

### Node 类型的变化
- isSameNode(比较节点) 节点相同, 同一个对象
- isEqualNode(比较节点) 节点相等, 具有相同的属性(nodeName, nodeValue等)

```
var div1 = document.createElement("div");
div1.setAttribute("class", "box");

var div2 = document.createElement("div");
div2.setAttribute("class", "box");

div1.isSameNode(div1);//true
div1.isEqualNode(div2);//true
div1.isSameNode(div2);//false

```

- setUserData()将数据指定给节点, 它接受3个参数, 要设置的键, 实际的值(可以是任务数据类型)和处理函数, 处理函数接受5个参数, 分别表示操作类型的数值(1表示复制, 2表示导入, 3表示删除, 4表示重命名), 数据键, 数据值, 源节点和目标节点

```
var div = document.createElement("div");
div.setUserData("name", "James", function(operation, key, value, src, desc) {
	if (operation == 1) {
		desc.setUserData(key, value, function() {})
	}
});
var newDiv = div.cloneNode(true);
newNode.getUserData("name");//James
```

### 框架变化
- contentDocument 指向表示框架内容的文档对象

```
var iframe = document.getElementsById("myIframe");
var iframeDoc = iframe.contentDocument || iframe.contentWindow.document;
```

## 样式
### 访问元素样式
- getPropertyCSSValue(propertyName)
- getPropertyValue(propertyName)
- cssText 

```
var myDiv = document.getElementsByTagName("div")[0];
myDiv.style.cssText = "font-size: 18px; background-color: red";
```

### 计算的样式
- getComputedStyle 可以获取其他样式表层叠而来并影响到当前元素的样式信息
```
<style type="text/css">
	#myDiv {
		width: 100px;
		height: 100px;
		background-color: blue;
	}
</style>
<div id="myDiv" style="background-color:red;"></div>
<script type="text/javascript">
	var myDiv = document.getElementById("#myDiv");
	var computeStyle = document.defaultView.getComputedStyle(myDiv, null);
	computeStyle.backgroundColor; //red
	computeStyle.width;//100px;
	computeStyle.height;//100px;
</script>
```

- currentStyle IE下使用此属性获取计算后的属性

```
var myDiv = document.getElementById("#myDiv");
var currentStyle = myDiv.currentStyle;
currentStyle.backgroundColor;
```

### 操作样式
- document.styleSheets 获取所有style样式
- sheet.deleteRule(索引)
- sheet.insertRule(规则文本, 插入规则的索引) 向现有样式表插入新规则

```
var sheet = document.styleSheets[0];
sheet.insertRule("body {max-width: 1000px;}", 0);
```

## 元素大小
### 偏移量
- offsetHeight 垂直空间上的占用像素, 包括元素的高度, (可见的)水平滚动条高度, 上边框高度和下边框高度
- offsetWidth 水平方向上的占用像素, 包括元素的宽度, (可见的)垂直滚动条宽度, 左边框和右边框的宽度
- offsetLeft 元素左外边框至包含元素的左内边框之间的像素距离
- offsetTop 元素上外边框至包含元素的上内边框之间的像素距离
![偏移量](https://img.alicdn.com/tfs/TB1DpvFSFXXXXcqXXXXXXXXXXXX-1294-770.png)

> 元素的offsetParent和parentNode不一定相等

> 计算一个元素在页面上的偏移量

```
function getElementLeft(element) {
	var actualLeft = element.offsetLeft;
	var currParent = element.offsetParent;

	while (currParent != null) {
		actualLeft += currParent.offsetLeft;
		currParent = currParent.offsetParent;
	}
}

function getElementTop(element) {
	var actualTop = element.offsetTop;
	var currParent = element.offsetParent;

	while (currParent != null) {
		actualTop += currParent.offsetTop;
		currParent = currParent.offsetParent;
	}
}

```

### 客户区大小
- clientWidth 内容区宽度+左右内边距的宽度
- clientHeight 内容区高度+上下内边距的高度

![客户区大小](https://img.alicdn.com/tfs/TB1L8O2SFXXXXbQaXXXXXXXXXXX-1190-762.png)

> 浏览器视口大小

```
function getViewport() {
	//兼容IE7以前
	if (document.compatMode == "BackCompat") {
		return {
			width: document.body.clientWidth,
			height: document.body.clientHeight
		}
	} else {
		return {
			width: document.documentElement.clientWidth,
			height: document.documentElement.clientHeight
		}
	}
}
```

### 滚动大小
- scrollHeight 没有滚动条的情况下, 元素内容的总高度
- scrollWidth 没有滚动条的情况下, 元素内容的总宽度
- scrollLeft 被隐藏在内容左侧的像素数, 可以通过改变此值来改变元素的滚动位置
- scrollTop 被隐藏的内容区域上方的像素数, 可以通过改变此值改变元素的滚动位置

> 当页面大小没有超过视口大小时, `scrollHeight`与`clientHeight`, `scrollWidth`与`clientWidth`值相等;

```
//chrome 下测试结果  各浏览器结果不一致, 为了兼容性 如果想获取文档高度, 应该取两值的最大值
document.documentElement.scrollHeight;// 2874 最个文档高度
document.documentElement.clientHeight;// 911 视口高度

document.body.scrollHeight;// 2874 最个文档高度
document.body.clientHeight;// 2874 最个文档高度
```

### 确定元素大小
- getBoundingClientRect() 获取元素left, right, top, bottom

```
document.documentElement.getBoundingClientRect()
```

## 遍历
### NodeIterator

- document.createNodeIterator(起始节点, 要访问哪些节点的代码, nodeFilter对象, 是否要扩展实体引用)

```
var div = document.getElementById("div1");
var iterator = document.createNodeIterator(div, NodeFilter.SHOW_ELEMENT, null, false);
var node = iterator.nextNode();
while (node !== null) {
	console.log(node.tagName);
	node = iterator.nextNode();
}
```

### TreeWalker

- document.createTreeWalk(起始节点, 要访问哪些节点的代码, nodeFilter对象, 是否要扩展实体引用)

```
var div = document.getElementById("div1");
var filter = function(node) {
	return node.tagName.toLowerCase() == 'li' ? NodeFilter.FILTER_ACCEPT : NodeFilter.FILTER_SKIP;
}
var iterator = document.createTreeWalk(div, NodeFilter.SHOW_ELEMENT, filter, false);
var node = iterator.nextNode();
while (node !== null) {
	console.log(node.tagName);
	node = iterator.nextNode();
}
```