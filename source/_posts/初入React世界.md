---
title: 初入React世界
date: 2017-10-10 08:47:18
tags: 深入React技术栈
---
## React生命周期
- constructor
- static getDerivedStateFromProps
- componentWillMount / UNSAFE_componentWillMount
- render
- componnetDidMount
- componentWillReceiveProps / UNSAFE_componentWillReceiveProps
- static getDerivedStateFromProps
- shouldComponentUpdate
- componentWillUpdate / UNSAFE_componentWillUpdate
- render
- getSnapshotBeforeUpdate
- componnetDidUpdate
- componentWillUnmount
- componentDidCatch

### componnetDidMount
> 绑定事件, ajax请求尽量在这个阶段调用; 此阶段虽然在render之后被调用, 实质上仅仅生成了dom树, 还没有被渲染到页面上, 因此在调用setState, 页面上不会出现重复渲染

### shouldComponentUpdate
> 此阶段控制页面是否重新渲染; 可以通过继承 `React.PureComponent`, 它实现了prop和state的浅比较; react不推荐我们在这阶段进行深度比较, 或是通过json.stringify进行比较, 这样很耗性能;

### getSnapshotBeforeUpdate
```js
class ScrollingList extends React.Component {
  listRef = React.createRef();

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // Are we adding new items to the list?
    // Capture the current height of the list so we can adjust scroll later.
    if (prevProps.list.length < this.props.list.length) {
      return this.listRef.current.scrollHeight;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // If we have a snapshot value, we've just added new items.
    // Adjust scroll so these new items don't push the old ones out of view.
    if (snapshot !== null) {
      this.listRef.current.scrollTop +=
        this.listRef.current.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```

### componentDidCatch

```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error, info) {
    // Display fallback UI
    this.setState({ hasError: true });
    // You can also log the error to an error reporting service
    logErrorToMyService(error, info);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

## ReactDom
### ReactDOMServer.renderToString(element) 
在服务端渲染react的html, 加速首屏渲染和SEO优化

```js
import ReactDOMServer from 'react-dom/server';
ReactDOMServer.renderToString(element) 
```

### ReactDOM.hydrate(element, container[, callback])
可以对服务器渲染的容器挂载事件


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