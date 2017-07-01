---
title: css
date: 2017-03-02 21:25:22
tags: css
---
### border
- border-width, outline, box-shadow, text-shadow等不支持百分比
- border-width默认medium(3px)
- chrome/firefox 虚线比例3:1; IE 2:1
- border-style:dotted 是原型的, 所以利用此功能实现圆形
```
.box {
    width: 150px;
    height: 150px;
    /*超出区域隐藏, 只显示一个圆*/
    overflow: hidden;
}
.dotted {
    width: 100%;
    height: 100%;
    border: 149px dotted #cd0000;
}
```

### 选择器
#### 包含选择符 与 子选择符的区别
- 包含选择器只要是后代就生效
- 子选择必须是儿子(IE7+)
```
// 包含选择器
div p
{
    color: red;
}
//子选择器
div > p
{
    color: red;
}
```
#### 相邻选择器 
- 只有相邻才生效(IE7+)
```
div + p
{
    color: blue;
}
```
#### 属性选择符：
- E[attr]、E[attr=“value”]、E[attr~=“value”]、E[attr|=“value”]、E[attr^=“value”]

#### 伪类选择器
- 可以在目前前面或后面添加内容, 但在dom节点上看不到相应的内容
- hover这些可以配合before,after使用, 但是必须先申明hover
```
.test:hover::after {
    content: ' hello ';
    color: red;
}
// 可以使用自身的属性
.test:hover::after {
    content: attr(title);
    color: red;
}
<div class="test" title="hello">aaa</div>
```
#### CSS样式采用的优先顺序：
- 标有!important关键字声明的属性。
- html中的CSS样式属性。
- 作者编辑的CSS文件样式属性。
- 用户设置的样式。
- 浏览器默认的样式。

#### 选择符优先级积分：
- 标签选择符、伪类及伪对象：优先级积分为1。
- 类选择符、属性选择符：优先级积分为10。
- ID选择符：优先级积分为100。
- style属性：优先级积分为1000。
- 其他选择符，如通配符选择符等：优先级积分为0。

#### 清楚浮动
1. clear属性--常用clear: both;
2. 添加额外标签: <div style="clear:both;"></div>
3. 使用 br标签和其自身的 html属性: <br clear="all">
4. 父元素设置: overflow: hidden; *zoom:1; (在IE6中还需要触发 hasLayout ，例如 zoom：1)
5. 父元素设置: display: table;
6. 父元素设置 :after 伪类 (推荐)：
```
.clearFix:after {
    clear: both; /* 清除伪类层以上的浮动 */
    display: block;   /* 使生成的元素以块级元素显示,占满剩余空间; */
    visibility: hidden; /* 设置伪类层内容为块级元素且不可见 */   
    height: 0;
    line-height: 0; /* 设置伪类层中的高度和行高为0 */
    content: " "; /* 将伪类层中的内容清空 */
}
.clearFix {
    zoom: 1; /* 针对IE浏览器产生hasLayout效果清除浮动 */
}

/* 更简洁的写法 */
.clearFix:before,
.clearFix:after {
    content: "";
    display: table;
}

.clearFix:after {
    clear: both;
}
.clearFix{
    zoom:1;
}
```

#### 滚动条
- 默认的滚动条是html的 (body的滚动条会与边框相差8个像素)

#### 元素宽度
- 元素宽度 = width + borderWidth + padding + margin


### 布局经验
- 浮动的父级尽量设置高度, 这样不会整体上移
- 块级元素, 左右居中可以用`margin: 0 auto`; 但是上下居中只能靠`margin/padding`调整
- 当div漂浮起来后, 这行高度就变化了, 不在按这个div高度来算了
- 如何让文字居中, 设置宽度, 再设置 `text-align:center`
