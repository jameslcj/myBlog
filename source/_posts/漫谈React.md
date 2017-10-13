---
title: 漫谈React
date: 2017-10-11 09:01:17
tags: 深入React技术栈
---
## 事件系统
### 事件委派
> 它并不会把事件处理函数直接绑定到 真实的节点上，而是把所有事件绑定到结构的最外层，使用一个统一的事件监听器，这个事件监 听器上维持了一个映射来保存所有组件内部的事件监听和处理函数

### 在React中使用原生事件
```
class NativeEventDemo extends Component { 
	componentDidMount() {
		this.refs.button.addEventListener('click', e => { this.handleClick(e);}); 
	}
	handleClick(e) { 
		console.log(e);
	}
	componentWillUnmount() { 
		this.refs.button.removeEventListener('click');
	}
	render() {
		return <button ref="button">Test</button>;
	} 
}
```