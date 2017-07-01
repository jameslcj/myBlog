---
title: jquery源码心得
date: 2017-01-11 19:38:17
tags: [JavaScript,Jquery]
---
# 阅读jquery源码心得
- 闭包参数传递window, 是为了寻找window时, 不用寻找到外层, 提高效率, 也为压缩代码提供了方便
- 闭包声明undefined, 但不赋值, 就是让underfined == ‘underfined' 防止ie9一下的浏览器 修改其值
- jquery = function(selector,…) {return new jquery.property.init()} 这样做是做了 我们平时$()时 就能执行init里的方法; 不用每次都要先调用init; 如何实现? jquery.property.init.property = jquery.property; 这样init 不仅有自身方法也有jquery的所有方法
- null 只与 null和undefined 相等, 其他都是false
- typeof Nan == ‘number’ 所以要用isNan 和 isFinite 来判断是不是数字
- 判断类型建议使用 [].toString.call(new Date)  {}.toString.call(new Date) 获取更新详细的结果
- obj.constructor.prototype.hasOwnProperty('isPrototypeOf’) 对象独有的属性

### mouseover mousein 结构嵌套时冒泡解决方法
- mouseover mousein 在结构嵌套的时候会有冒泡传递事件影响有些需求事件, 可以换成mouseleave mouseenter
- 其他解决方法
```
//解决方法
<script>
var oDiv1 = document.getElementById('div1');
var oDiv2 = document.getElementById('div2');
var oSpan1 = document.getElementById('span1');
oDiv1.onmouseover = function(ev){
	var ev = ev || window.event;
	var a = this;
	var b = ev.relatedTarget;
	if( !elContains(a, b) && a!=b ){
		oSpan1.innerHTML += '1';
	}
};
oDiv1.onmouseout = function(ev){
	var ev = ev || window.event;
	var a = this;
	var b = ev.relatedTarget;
	if( !elContains(a, b) && a!=b ){
		oSpan1.innerHTML += '2';
	}
};
function elContains(a, b){  //两个元素是否是嵌套关系
	return a.contains ? a != b && a.contains(b) : !!(a.compareDocumentPosition(b) & 16);
}
</script>
<body>
<style>
#div1{ width:200px; height:200px; background:red;}
#div2{ width:100px; height:100px; background:yellow;}
</style>

<div id="div1">
	<div id="div2"></div>
</div>
<span id="span1">11111111111</span>
</body>
```
### trigger() 与 triggerHandler的区别
- trigger 会触发对应事件的回调函数, 并且触发事件的默认行为(比如focus 光标会获取焦点)
- triggerHandler 会触发对应事件的回调函数, 但不会触发事件的默认行为
### `$().prop` 与 `$().attr`的区别 
- `$("#id").prop` 实现的原理是
    + 设置值时: `document.getElementById('id')['name'] = 'name'`
    + 获取值时: `document.getElementById('id')['name']`
- `$("#id").attr` 实现的原理是
    + 设置值时: `document.getElementById('id').setAttribute = 'name'`
    + 获取值时: `document.getElementById('id').getAttribute`
- 当对document等对象(没有setAttrite方法), 建议使用prop(jquery做了兼容)
- 当获取自定义或者设置自定义属性时, prop在某些浏览器上无法实现
- removeProp在某些浏览器上无法删除原生属性

### 浏览器内存泄漏
```
var div1 = document.getElementById('div')[0];
var div2 = document.getElementById('div')[1];
div1.name = div2;
div2.name = div1;
```

### $的 attr, prop, data3个方法的区别
- data不会有浏览器内存泄漏, 其他2个会有
- data适合挂载大量数据
- data是利用一个中间件将对象和属性联系起来的, 不是直接将属性挂载在对象上, 这样就避免了内存泄漏, 和挂载大量数据时影响效率
