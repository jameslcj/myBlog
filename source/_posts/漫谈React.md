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

## 组件通信
### 跨级组件通信通信
> 父级组件申明`getChildContext`方法和`childContextTypes`对象属性, 子组件申明`contextTypes`再可以通过`this.context`跨级获取对应的属性, 类似全局变量的概念, 但是这样比较混乱, react不建议这样使用

```
class Parent extends Component {
	static childContextTypes = {
	    color: PropTypes.string,
	}
	getChildContext() { 
		return {
			color: 'red', 
		};
	}

	render() {
		return <Son />
	}
}

class Son extends Component {
	static contextTypes = {
	    color: PropTypes.string,
	};
	render() {
		return {this.context.color}
	}
}

```

### 没有嵌套关系的组件通信
> 通过`EventEmitter`对象

```
import { EventEmitter } from 'events';
const emitter = new EventEmitter();
class App extends Component {
  componentDidMount() {
    this.btnClick = emitter.on('btnClick', (data) => { 
      console.log(data);
      alert(data)
    }); 
  }

  componentWillUnmount() {
    emitter.removeListener(this.btnClick);
  }
}

class OtherComp extends Component {
	constructor(props) {
    	super(props);
	}
	onClickEvent() {
	    emitter.emit('btnClick', 'click...')
	}
	render() {
		
		return <button onClick={this.onClickEvent.bind(this)}>按钮</button>
	}
}
```