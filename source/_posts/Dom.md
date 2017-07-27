---
title: DOM
date: 2017-07-14 09:43:27
tags: javaScript高级程序设计笔记
---
## 节点层次
> 如下, 最外层的节点<html>, 称为文档元素, 每个文档只能有一个文档元素;
在html页面中, 文档元素始终都是<html>元素;

```
<html>
  <head>
    <title>Home</title>
  </head>
  <body>
    <p>hello wolrd!</p>
  </body>
</html>
```

### Node类型
> 节点类型如下:

```
Node.ELEMENT_NODE(1);
Node.ATTRIBUTE_NODE(2);
Node.TEXT_NODE(3);
Node.CDATA_SECTION_NODE(4);
Node.ENTITY_REFERENCE_NODE(5);
Node.ENTITY_NODE(6);
Node.PROCESSING_INSTRUCTION_NODE(7);
Node.COMMENT_NODE(8);
Node.DOCUMENT_NODE(9);
Node.DOCUMENT_TYPE_NODE(10);
Node.DOCUMENT_FRAGMENT_NODE(11);
Node.NOTATION_NODE(12);

//可以通过如下方式判断
if (someNode.nodeType == 1) {
}
```
### nodeName和nodeValue属性
> `nodeName`元素的标签名, `nodeValue`节点值

```
var someNode = document.getElementById('html');
someNode.nodeName;//"html"
someNode.nodeValue;//null
```

### 节点关系
> 每个节点都有一个childNodes属性, 保存着一个NodeList对象, NodeList是一种类数组对象(有length属性, 但不是数组);
可以通过previousSibling和nextSibling属性, 可以访问同一列表中的相邻其他节点.
firstChild获取第一个节点, lastChild获取最后一个节点;
parentNode可以获取父节点;
hasChildNodes()可以判断是否有子节点;
所有节点都有一个属性是ownerDocument, 该属性指向整个文档的文档节点, 即: document;
以下为获取NoeList中的节点方法

```
var firstChild = someNode.childNodes[0];
var secondChild = someNode.childNodes.item(1);

firstChild.nextSibling == secondChild;//true
secondChild.previousSibling == firstChild;//true
someNode.firstChild == firstChild//true
someNode.lastChild == someNode.childNodes[someNode.childNodes.length - 1]; //true
someNode.firstChild.parentNode == someNode;//true
```

> 由于NodeList是类数组, 可以通过Array.prototype.slice.call(NodeList, 0)方式转换为数组, 但是需要IE8以上, 因为IE8及一下是DOM是用COM对象实现的, 不是JScript;

```
function convertToArray(nodes) {
  var array = [];
  try {
    array = Array.prototype.slice.call(nodes, 0);//针对非IE浏览器
  } catch (ex) {
    for (var i=0, len=nodes.length; i < len; i++) {
      array.push(nodes[i]);
    }
  }

  return array;
}
```

### 操作节点
> appendChild()想childNodes列表的末尾添加一个节点;
insertBefore(要插入的节点, 参照的节点); 可以把节点放在childNodes列表中某个特定的位置上, 如果参照节点为null, 则insertBefore与appendChild执行相同的操作;
replaceChild(要插入的节点, 替换节点);
removeChild(要移除的节点);

```
var returnedNode = someNodes.appendChild(newNode);
returnedNode == newNode; //true
someNodes.lastChild == newNode; //true

//在末尾插入
someNodes.insertBefore(newNode, null);
someNodes.lastChild == newNode; //true

//插入到第一个
someNodes.insertBefore(newNode, newNode.firstChild);
someNodes.firstChild == newNode; //true

//替换第一个节点
someNodes.replaceChild(newNode, someNodes.firstChild);

//移除第一个节点
someNodes.removeChild(someNodes.firstChild);
```

### 其他方法
> cloneNode(是否深复制子节点)克隆节点; 当传递为true时, 会复制节点自身和它的子节点, 否则只复制自身, 复制后返回的节点副本属于文档所有, 但并没有为它指定父节点. 需要通过appendChild(), insertBefore()或replaceChild()将它添加到文档中. cloneNode()不会复制DOM节点中的JavaScript属性, 例如事件处理程序等.IE在此存在一个bug, 即它会复制事件处理程序, 所以我们建议在复制之前最好先移除事件处理程序;
normalize()处理文档树中的文本节点, 删除节点的后代节点中的空文本节点, 或是将相邻的文本节点, 合并为一个文本节点.

```
<ul>
  <li>item 1</item>
  <li>item 2</item>
  <li>item 3</item>
</ul>
var deepList = myList.cloneNode(true);
console.log(deepList.childNodes.length);//3(IE<9)或7(其他浏览器)

var shallowList = myList.cloneNode(false);
console.log(shallowList.childNodes.length);//0
```

## document类型
> document是HTMLDocument的一个实例

```
document.nodeType//9
document.nodeName //#document
document.nodeValue//null
document.parentNode//null
document.ownerDocument//null
```

### 文档的子节点
- document.documentElement
- document.body
- document.doctype(只读), 各浏览器对其是否解析为document的子节点各不相同

```
<html>
  <head></head>
  <body></body>
<html>

document.childNodes[0] === document.documentElement //true
```

### 文档信息
- document.title 获取标题信息, 修改标题
- document.URL(只读) 当前地址
- document.referrer(只读) 来源网站的URL
- document.domain

> 由于安全方面的限制, 这个值只能设置为URL中的子域名, 比如:www.test.com只能设置为test.com;
这个值的用处是, 当一个页面嵌套一个同域子页面时, 如果要想通过JavaScript进行通信, 则必须将两个页面的document.domain设置为同一个域, 才可以进行通信;

- namedItem
> 对`HTMlCollectio`n集合调用数字索引值时, 就会调用`item()`方法, 如果调用字符串索引值就会调用`namedItem()`方法

```
<img src="myimage.gif" name="myImage">
// 获取HTMLCollection集合
var images = document.getElementByTagName("img")
images.namedItem("myImage");
//等价于
images["myImage"]
```

> `document.getElementByTagName('*')`可以获取所有元素, 但是在ie下注释的元素也会被获取

### 特殊集合
- document.anchors 所有带`name`标签的<a>元素
- document.applets 所有<applets>元素
- document.forms 所有<form>元素
- document.images 所有<img> 元素
- document.links 所有带`href`特性的<a>元素
- document.write() 将输出流写入网页
- document.writeln() 将输出流写入网页并换行

> `document.write/writeln` 这2个方法会重写`整个页面`, 还有如果输出`</script>`一定要转译成`<\/script>`, 否则会报错

## Element类型
- nodeType值为1
- nodeName/tagName值为元素的标签名
- nodeValue值为null
- parentNode值为Document或Element

> nodeName一般为大写, 所以先进行转大小写, 再进行比较

```
<a>我是a标签</a>
var a = document.getElementsByTagName("a")[0]
a.nodeName //A
a.nodeName === a.tagName//true
if (a.nodeName.toLowerCase == 'a') {
  //do something
}
```

### html元素
> 每个html元素都有如下属性

- id
- title
- className
- lang 元素内容的语言代码, 很少使用
- dir 语言的方法, 值为 "ltr" 或 "rtl"
```
<div id="div1" class="myClass" title="myTitle" lang="en" dir="ltr"></div>
var div = document.getElementById("div1");
div.id;//div1
div.className;//myClass
div.title;//myTitle
div.lang;//en
div.dir;//ltr
```

### 取得特性
- getAttribute() 获取属性, 获取的属性名与实际相同, 所以获取class时, 使用getAttribute('class')就可以而不是className

> 只有公认的属性会添加到DOM对象上(ie除外), 其他的自定义属性需要使用`getAttribute()`获取;
获取`style`属性时会返回一个对象或是`null`, 获取事件属性时会返回函数或者`null`

```
<div id="div1" class="myClass" my_attr="myAttr"></div>
var div = document.getElementById("div1");
div.my_attr;//undefined(ie除外)
div.getAttribute("my_attr");//myAttr
div.getAttribute("class");//myClass
```

### 设置特性
- setAttribute() 设置属性

```
<div id="div1" class="myClass" my_attr="myAttr"></div>
var div = document.getElementById("div1");
div.setAttribute("title", "myTitle");
div.className = "newClass";
//如果这样直接设置属性, 在大多数浏览器下, 这个属性不会自动变成元素的特性, 因此用getAttribute("myColor")结果为null
div.myColor = "red";
```

### 删除属性
- removeAttribute() 删除属性

```
<div id="div1" class="myClass" my_attr="myAttr"></div>
var div = document.getElementById("div1");
div.removeAttribute("class");
```

### attributes属性
- getNamedItem(name) 获取名为name的属性
- removeNamedItem(name) 删除名为name的属性
- setNamedItem(node) 向列表中添加节点, 以节点的nodeName为索引
- item(pos) 返回位于数字pos位置的节点

```
var id = element.attributes.getNamedItem("id").nodeValue;
//等价于如下
var id = element.attributes["id"].nodeValue;
```

### 创建元素
- document.createElement()

> 由于在ie7以下, 某些动态创建的元素会有问题, 建议使用完整的html创建, 其他浏览器就使用标签创建

```
if (client.browser.ie && client.browser.ie <= 7) {
  var iframe = document.createElement("<iframe name=\"myName\"></iframe>");
} else {
  var iframe = document.createElement("iframe");
}
```
### 元素子节点
> 除ie浏览器外, 都会计算子节点下的空白节点, 因此判断子节点时候, 对nodeType进行判断是否为1

```
<ul>
  <li></li>
  <li></li>
  <li></li>
</ul>

var element = document.getElementsByTagName('ul')[0];
element.childNodes.length; //非ie为3, 其他浏览器为7


//-----------------
<ul><li></li><li></li><li></li></ul>

var element = document.getElementsByTagName('ul')[0];
element.childNodes.length; //所有浏览器都为3

for (var i = 0; i < element.childNodes.length; i++) {

  if (element.childNodes[i].nodeType == 1) {
    //do something
  }

}
```

## 文本节点
- nodeType值为3
- nodeName值为"#text"
- nodeValue值为节点所包含的内容
- parentNode是一个元素节点
- 不支持子节点
- appendData(text) 将text添加到节点末尾
- deleteData(offset, count) 从offset开始删除count个字符
- insertData(offset, text) 从offset开始插入text
- replaceData(offset, count, text) 从offset开始, 替换count个字符为text
- splitText(offset) 从offset位置开始讲文本分成两个文本节点
- substringData(offset, count) 获取从offset到offset+count为止的字符串

```
<span>hello</span>

var spanText = document.getElementsByTagName("span")[0].childNodes[0];
spanText.appendData(" world");// hello world
spanText.deleteData(5, 6);// hello

//默认情况下, 至多有一个文本节点
<span></span> //没有文本节点
<span> </span> //有空格 所以有文本节点
```

### 创建文本节点
- document.createTextNode

```
<span>hello</span>

var ele = document.getElementsByTagName("span")[0];
var textNode = document.createTextNode(" world")
ele.appendChild(textNode);
```

### 规范化文本节点
- normalize() 可以将多个子文本节点变成一个
```
<span>hello</span>

var ele = document.getElementsByTagName("span")[0];
var textNode = document.createTextNode(" world")
ele.appendChild(textNode);
ele.childNodes.length;//2
ele.normalize();
ele.childNodes.length;//1
```

### 分割文本节点
- splitText(offset) 指定位置分割文本节点, 功能和normalize()相反

## comment类型
> 注释节点 功能与text节点相似, 就少一个splitText方法

- nodeType值为8
- nodeName值为"#comment
- nodeValue值为注释内容
- parentNode值为Document或Element
- 没有子节点
- document.createComment() 创建注释

### DocumentFragment类型
> DocumentFragment类型节点类似一个节点仓库, 如果直接在节点树里添加新节点, 会导致页面重绘, 所以我们可以将多次添加的节点, 
先放到DocumentFragment节点里先, 再添加到dom树中, 这样只需重绘一次就够了.

- nodeType值为11
- nodeName值为"#document-fragment
- nodeValue值为null
- parentNode值为null
- document.createDocumentFragment()

```
var fragment = document.createDocumentFragment();
var ul = document.getElementById("myUl");

for (var i=0; i < 3; i++) {
  var li = document.createElement('li');
  fragment.appendChild(li);
}
ul.appendChild(fragment);
```
