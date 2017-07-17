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
所有节点都有一个属性是ownerDocument, 该属性指向整个文档的文档节点, 即: docuemnt;
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
