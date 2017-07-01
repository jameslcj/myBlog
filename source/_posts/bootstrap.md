---
title: bootstrap
date: 2017-03-02 21:24:09
tags: css
---
### 栅格
- 要被div.row包裹起来
- 一行只有12个
- col-lg col-md col-sm col-xs 这些是根据不同的分辨率展现对应的个数 可以嵌套使用
- col-lg-offset 向右偏移
- col-lg-push 向右偏移和offset类似 但不完全一样
- col-lg-pull 向左偏移
- visible-lg-block 在对应的分辨率下可见 其他情况隐藏
- hidden-sm 在对应的像素下隐藏
- pull-right/left 贴在左边或右边
- hidden/visible-print-block 打印显示或隐藏
### 样式
- default primary success info warning error
- btn
    + btn-link 有连接
    + btn-block 独占一行
    + 也有 btn-lg lg md sm xs 样式大小
    + disable 禁用样式
    + active 选中样式
- btn-group 按钮变为一组
    + btn-group-justified 充满一行 默认只对a标签生效 input和button需要再套一层div.btn-group
    + btn-group-vertical
- bg
- text
- alert
- panel
    + panel-heading
    + panel-body
- input
- form-control
- caret 下拉箭头
    + dropup 箭头向上
### 下拉
- data-* 跟js有关 data-toggle
    + data-target 操作对象
- aria-* 残疾人辅助功能
- divider 分割线
```
<div class="container">
        <div class="bg-primary">aaaa</div>
        <div class="dropdown">
            <div class="btn-group dropdown-toggle" data-toggle="dropdown" data-target=".dropdown">
                <div class="btn btn-primary">aaa</div>
                <div class="btn btn-primary"><span class="caret"></span></div>
            </div>
            <ul class="dropdown-menu">
                <li>1</li>
                <li>2</li>
                <li>3</li>
                <li>4</li>
            </ul>
        </div>
    </div>
    <div class="container">
        <div class="bg-primary">aaaa</div>
        <div class="dropdown">
            <div class="btn-group dropdown-toggle" data-toggle="dropdown">
                <div class="btn btn-primary">aaa</div>
                <div class="btn btn-primary"><span class="caret"></span></div>
            </div>
            <ul class="dropdown-menu">
                <li>1</li>
                <li>2</li>
                <li>3</li>
                <li>4</li>
            </ul>
        </div>
    </div>
```
### 导航 nav
- nav 
- nav-tabs
- nav-jusified 等宽
- nav-pills
- nav-stackted 垂直的
- tab-content 导航内容
- tab-pane 
```
<ul class="nav nav-tabs">
    <li class="active"><a href="#a" data-toggle="tab">one</a></li>
    <li><a href="#b" data-toggle="tab">one</a></li>
    <li><a href="#c" data-toggle="tab">one</a></li>
    <li><a href="#d" data-toggle="tab">one</a></li>
</ul>
<ul class="tab-content">
    <li class="tab-pane active" id="a">aaa</li>
    <li class="tab-pane" id="b">bbb</li>
    <li class="tab-pane" id="c">ccc</li>
    <li class="tab-pane" id="d">ddd</li>
</ul>
```
- navbar 
- navbar-default narbar-* 
- navbar-inverse 不同的风格
- navbar-fixed-top/bottom 固定
- navbar-header
    + navbar-brand
```
<nav class="navbar navbar-default nav-inverse">
            <div class="navbar-header">
                <a href="" class="navbar-brand">logo</a>
            </div>
            <ul class="nav nav-tabs">
                <li class="active"><a href="#a" data-toggle="tab">one</a></li>
                <li><a href="#b" data-toggle="tab">one</a></li>
                <li><a href="#c" data-toggle="tab">one</a></li>
                <li><a href="#d" data-toggle="tab">one</a></li>
            </ul>
        </nav> 
```
- navbar-left 内容靠左 
- navbar-right
- navbar-btn
- navbar-link
- navbar-text
- navbar-toggle
- navbar-collapse 导航折叠
```
<nav class="navbar navbar-default nav-inverse">
            <div class="navbar-header">
                <a href="" class="navbar-brand">logo</a>
                <btton class="navbar-toggle" data-toggle="collapse" data-target="#myCollapse">
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </btton>
            </div>
            <div class="collapse navbar-collapse" id="myCollapse">
                <ul class="nav nav-tabs navbar-left">
                    <li class="active"><a href="#a" data-toggle="tab">one</a></li>
                    <li><a href="#b" data-toggle="tab">one</a></li>
                    <li><a href="#c" data-toggle="tab">one</a></li>
                    <li><a href="#d" data-toggle="tab">one</a></li>
                </ul>
                <button class="navbar-nav navbar-btn">aa</button>
                <p class="nav navbar-nav navbar-text navbar-right">bbb</p>
                <form action="" class="navbar-form navbar-left">
                    <input type="text" class="form-control">
                    <input type="text" class="btn btn-primary" value="搜索">
                </form>
                <ul class="nav navbar-nav  navbar-right">
                    <li><a href="">登录</a></li>
                </ul>
            </div>
        </nav> 
```
- data-spy="scroll" 监听滚动条
- data-target="#"
- data-offset="200px" 偏移距离
