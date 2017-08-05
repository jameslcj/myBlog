---
title: DOM2和DOM3
date: 2017-08-05 14:18:55
tags: javaScript高级程序设计笔记
---
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