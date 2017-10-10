---
title: 初入React世界
date: 2017-10-10 08:47:18
tags: 深入React技术栈
---
## React生命周期
- componentWillMount
- render
- componnetDidMount
- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate
- render
- componnetDidUpdate
- componentWillUnmount

## React 与 DOM
- findDOMNode 需要react节点

```
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
class App extends Component { 
 componentDidMount() {
 	// this 为当前组件的实例
	const dom = ReactDOM.findDOMNode(this); 
 }
 render() {} 
}
```

- unmountComponentAtNode 卸载react节点

## ReactDOM 的不稳定方法
- render: ReactMount._renderSubtreeIntoContainer(null, nextElement, container, callback)
- unstable_renderSubtreeIntoContainer: ReactMount._renderSubtreeIntoContainer(parentComponent,
nextElement, container, callback)

> unstable_renderSubtreeIntoContainer 与 render 方法很相似，但 render 方法缺少一个插入某个节点的参数