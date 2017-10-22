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

## 高阶组件
### 属性代理
```
import React, { Component } from 'React';
const MyContainer = (WrappedComponent) => 
	class extends Component {
	render() {
		const newProps = {
			text: newText, 
		};
		return <WrappedComponent {...this.props} {...newProps} />; 
	}
}
```

```
import React, { Component } from 'React';
const MyContainer = (WrappedComponent) =>
	class extends Component {
		constructor(props) { 
			super(props); 
			this.state = {
		  		name: '',
		  	};
			this.onNameChange = this.onNameChange.bind(this); 
		}

		onNameChange(event) { 
			this.setState({
				name: event.target.value, 
			})
		}
		render() {
			const newProps = {
				name: {
					value: this.state.name,
					onChange: this.onNameChange,
				}, 
			}
			return <WrappedComponent {...this.props} {...newProps} />; 
		}
}

@MyContainer
class MyComponent extends Component {
	render() {
		return <input name="name" {...this.props.name} />;
	} 
}
```
> 通过以上的封装，我们就得到了一个被控制的 input 组件。

### 反向代理
> 高阶组件集成传递进来的组件WrappedComponent, 然后通过super去反向调用

```
const MyContainer = (WrappedComponent) => 
	class extends WrappedComponent {
		render() {
			return super.render();
		} 
	}
}
```

## 组件性能优化
### PureRender
```
<Account style={{ color: 'black' }} />

//优化为如下

const defaultStyle = {{color: 'black'}};
<Account style={this.props.style || defaultStyle} />
```

> 如上, 我们知道，每次调用 React 组件其实都会重新创建组件。就算传入的数组或对象的值没有改变, 但它引用的地址改变, 因此对象和数组类型的数据, 应该用一个变量传入

```
class NameItem extends Component { 
	render() {
		//翻译成jsx为: <Item children={React.createElement('span', {}, 'Arcthur')}/>
		return ( <Item>
			<span>Arcthur</span> <Item/>
		 )
	}
}

import PureRenderMixin from 'react-addons-pure-render-mixin';
class NameItem extends Component {
	constructor(props) {
		super(props);
		this.shouldComponentUpdate = PureRenderMixin.shouldComponentUpdate.bind(this); 
	}
	render() { 
		return ( <Item>
			<span>Arcthur</span> </Item>
		); 
	}
}
```
> 如上, Item 组件不论什么情况下都会重新渲染。那么，怎么避免 Item 组件的重复渲染呢? 很简单，我们给 NameItem 设置 PureRender

### Immutable
> Immutable提供了很多类似es6工具方法, 它最大的亮点是提供了Map(对象), List(数据)这2个方法, 它可以得到不会被改变的对象和数组, 就可以防止数据被在某些黑盒里修改

```
let a = Map({
	select: 'users',
	filter: Map({ name: 'Cam' }),
});
let b = a.set('select', 'people');
a === b // false

let map1 = Immutable.Map({a:1, b:1, c:1}); 
let map2 = Immutable.Map({a:1, b:1, c:1}); 
map1 === map2; // => false
Immutable.is(map1, map2); // => true
```
> 以上为Immutable的map简单用法, Immutable.is 比较的是两个对象的 hashCode 或 valueOf(对于 JavaScript 对象)。由于 Immutable 内部使用了 trie 数据结构来存储，只要两个对象的 hashCode 相等，值就是一样的。 这样的算法避免了深度遍历比较，因此性能非常好。

```
import React, { Component } from 'react'; 
import { is } from 'immutable';

class App extends Component { 
	shouldComponentUpdate(nextProps, nextState) {
		const thisProps = this.props || {};
		const thisState = this.state || {};

		if (Object.keys(thisProps).length !== Object.keys(nextProps).length || Object.keys(thisState).length !== Object.keys(nextState).length) {
			return true; 
		}

		for (const key in nextProps) {
			if (nextProps.hasOwnProperty(key) && !is(thisProps[key], nextProps[key])) { 
				return true;
			} 
		}

		for (const key in nextState) {
			if (nextState.hasOwnProperty(key) && !is(thisState[key], nextState[key])) { 
				return true;
			} 
		}
		
		return false; 
	}
}
```
> 如上优化shouldComponentUpdate, Immutable.js提供了简洁、高效的判断数据是否变化的方法，只需 === 和 is 比较就能知 道是否需要执行 render，而这个操作几乎零成本，所以可以极大提高性能

### key
> 我们迭代渲染数组时, 如果不给组件设置key或是设置了相同的key会有警告, 但是设置的key, 尽量根据组件自身属性而产生的唯一性key, 比如id, 尽量避免使用数组的key, 这样效率会很低, 因为所有组件基本都会重新渲染


```
import React from 'react';
import createFragment from 'react-addons-create-fragment';
function Rank({ first, second }) { 
	const children = createFragment({
		first: first,
		second: second, 
	});
	return ( <ul>{children} </ul>);
}
```
> 关于 key，我们还需要知道的一种情况是，有两个子组件需要渲染的时候，我们没法给它们
设 key。这时需要用到 React 插件 createFragment 来解决, 如上

### react-addons-perf 性能检测工具
> 通过 Perf.start() 和 Perf.stop() 两个 API 设置 开始和结束的状态来作分析

- Perf.printInclusive(measurements):所有阶段的时间。 
- Perf.printExclusive(measurements):不包含挂载组件的时间，即初始化 props、state，调用 componentWillMount 和 componentDidMount 方法的时间等。
- Perf.printWasted(measurements):监测渲染的内容保持不变的组件(可以查看哪些组件
没有被 shouldComponentUpdate 命中)。