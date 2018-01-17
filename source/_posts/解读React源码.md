---
title: 解读React源码
date: 2018-01-06 10:34:47
tags: 深入React技术栈
---
## 15.0版本源码结构
- addons:包含一系列的工具方法插件，如 PureRenderMixin、CSSTransitionGroup、Fragment、 LinkedStateMixin 等。
- isomorphic:包含一系列同构方法
- shared:包含一些公用或常用方法，如 Transaction、CallbackQueue 等
- test:包含一些测试方法等
- core/tests:包含一些边界错误的测试用例。
- renderers:是 React 代码的核心部分，它包含了大部分功能实现，此处对其进行单独分析。
    + renderers 分为 dom 和 shared 目录
        - dom:包含 client、server 和 shared
            + client:包含 DOM 操作方法(如 findDOMNode、setInnerHTML、setTextContent 等)以 及事件方法，结构如图 3-2 所示。这里的事件方法主要是一些非底层的实用性事件方法， 如 事 件 监 听 ( ReactEventListener )、 常 用 事 件 方 法 ( TapEventPlugin 、 EnterLeave- EventPlugin)以及一些合成事件(SyntheticEvents 等)。
            + server:主要包含服务端渲染的实现和方法(如 ReactServerRendering、ReactServer- RenderingTransaction 等)。
            + shared:包含文本组件(ReactDOMTextComponent)、标签组件(ReactDOMComponent)、 DOM 属性操作(DOMProperty、DOMPropertyOperations)、CSS 属性操作(CSSProperty、 CSSPropertyOperations)等。
- shared:包含 event 和 reconciler
    + event:包含一些更为底层的事件方法，如事件插件中心(EventPluginHub)、事件注册(EventPluginRegistry)、事件传播(EventPropagators)以及一些事件通用方法。React 自定义了一套通用事件的插件系统，该系统包含事件监听器、事件发射器、事 件插件中心、点击事件、进/出事件、简单事件、合成事件以及一些事件方法，如图 3-3 所示。
    + reconciler:称为协调器，它是最为核心的部分，包含 React 中自定义组件的实现 (ReactCompositeComponent)、组件生命周期机制、setState 机制(ReactUpdates、ReactUpdateQueue)、DOM diff 算法(ReactMultiChild)等重要的特性方法。

## Virtual DOM 
构建一套简易 Virtual DOM 模型并不复杂，它只需要具备一个 DOM 标签所需的基本 元素即可:
- 标签名
- 节点属性，包含样式、属性、事件等
- 子节点
- 标识 id

```js
{
    // 标签名 
    tagName: 'div', 
    // 属性 
    properties: {
        // 样式
        style: {} 
    },
    // 子节点 
    children: [], 
    // 唯一标识 
    key: 1
}
```

> Virtual DOM 中的节点称为 ReactNode，它分为3种类型 ReactElement、ReactFragment 和 ReactText。其中，ReactElement 又分为 ReactComponentElement 和 ReactDOMElement

```js
type ReactNode = ReactElement | ReactFragment | ReactText;
type ReactElement = ReactComponentElement | ReactDOMElement;
type ReactDOMElement = { 
    type : string,
    props : {
        children : ReactNodeList, 
        className : string,
        etc.
    },
    key : string | boolean | number | null, 
    ref : string | null
};
type ReactComponentElement<TProps> = { 
    type : ReactClass<TProps>,
    props : TProps,
    key : string | boolean | number | null, 
    ref : string | null
};
type ReactFragment = Array<ReactNode | ReactEmpty>; 
type ReactNodeList = ReactNode | ReactEmpty;
type ReactText = string | number;
type ReactEmpty = null | undefined | boolean;
```
### 创建React元素
```js
const Nav, Profile;
// 输入(JSX):
const app = <Nav color="blue"><Profile>click</Profile></Nav>;
// 输出(JavaScript):
const app = React.createElement(
    Nav,
    {color:"blue"}, 
    React.createElement(Profile, null, "click")
);
```

> 源码路径: /v15.0.0/src/isomorphic/classic/element/ReactElement.js 

### 初始化组件入口
- 当 node 为空时，说明 node 不存在，则初始化空组件 ReactEmptyComponent.create(instan- tiateReactComponent)。
- 当 node 类型为对象时，即是 DOM 标签组件或自定义组件，那么如果 element 类型为字 符 串 时 ， 则 初 始 化 DOM 标 签 组 件 ReactNativeComponent.createInternalComponent (element)，否则初始化自定义组件 ReactCompositeComponentWrapper()。
- 当 node 类型为字符串或数字时，则初始化文本组件 ReactNativeComponent.create InstanceForText(node)。
- 如果是其他情况，则不作处理。

> 源 码 路 径 : /v15.0.0/src/renderers/shared/ reconciler/instantiateReactComponent.js

### DOM 标签组件
> 当开发者 使用 React 时，此时的 <div> 并不是原生 <div> 标签，它其实是 React 生成的 Virtual DOM 对象， 只不过标签名称相同罢了

ReactDOMComponent 针对 Virtual DOM 标签的处理主要分为以下两个部分:
- 属性的更新，包括更新样式、更新属性、处理事件等
- 子节点的更新，包括更新内容、更新子节点，此部分涉及 diff 算法。

1. 更新属性
当执行 mountComponent 方法时，ReactDOMComponent 首先会生成标记和标签，通过 this. createOpenTagMarkupAndPutListeners(transaction) 来处理 DOM 节点的属性和事件。
- 如果存在事件，则针对当前的节点添加事件代理，即调用 enqueuePutListener(this, propKey, propValue, transaction)。
- 如果存在样式，首先会对样式进行合并操作 Object.assign({}, props.style)，然后通过 CSSPropertyOperations.createMarkupForStyles(propValue, this) 创建样式。
- 通过 DOMPropertyOperations.createMarkupForProperty(propKey, propValue) 创建属性。
- 通过 DOMPropertyOperations.createMarkupForID(this._domID) 创建唯一标识。

> _createOpenTagMarkupAndPutListeners 方法的源码如下(源码路径:/v15.0.0/src/renderers/ dom/shared/ReactDOMComponent.js):

## 详解React生命周期
### 使用createClass创建自定义组件
createClass 是创建自定义组件的入口方法，负责管理生命周期中的 getDefaultProps。该方法法在整个生命周期中只执行一次，这样所有实例初始化的 props 将会被共享。通过 createClass 创建自定义组件，利用原型继承 ReactClassComponent 父类，按顺序合并mixin，设置初始化 defaultProps，返回构造函数。当使用 ES6 classes 编写 React 组件时，class MyComponent extends React.Component 其实就 是调用内部方法 createClass 创建组件

> 源码路径(src/isomorphic/classic/ class/ReactClass.js#L802)

### 阶段一:MOUNTING
mountComponent 负责管理生命周期中的 getInitialState、componentWillMount、render 和componentDidMount。
由于 getDefaultProps 是通过构造函数进行管理的，所以也是整个生命周期中最先开始执行 的。而 mountComponent 只能望洋兴叹，无法调用到 getDefaultProps。这就解释了为何 getDefaultProps只执行一次
由于通过 ReactCompositeComponentBase 返回的是一个虚拟节点，所以需要利用 instantiateReactComponent 去得到实例，再使用 mountComponent 拿到结果作为当前自定义元素的结果。

其实，mountComponent 本质上是通过递归渲染内容的，由于递归的特性，父组件的 componentWillMount 在其子组件的 componentWillMount 之前调用，而父组件的 componentDidMount 在其子组件的 componentDidMount 之后调用。

updateComponent 本质上也是通过递归渲染内容的，由于递归的特性，父组件的 componentWillUpdate 是在其子组件的 componentWillUpdate 之前调用的，而父组件的 componentDidUpdate 也是在其子组件的 componentDidUpdate 之后调用的

禁止在 shouldComponentUpdate 和 componentWillUpdate 中调用 setState，这会造成循环 调用，直至耗光浏览器内存后崩溃。
 > mountComponent源码: /v15.0.0/src/renderers/shared/reconciler/ReactCompositeComponent.js

 ## setState 异步更新
 setState 通过一个队列机制实现 state 更新。 当执行 setState 时，会将需要更新的 state 合并 后放入状态队列，而不会立刻更新 this.state，队列机制可以高效地批量更新 state。
 当调用 setState 时，实际上会执行 enqueueSetState 方法，并对 partialState 以及_pendingStateQueue 更新队列进行合并操作，最终通过 enqueueUpdate 执行 state 更新。
 而 performUpdateIfNecessary 方法会获取 _pendingElement、_pendingStateQueue、_pendingForceUpdate，并调用 receiveComponent 和 updateComponent 方法进行组件更新。
 如 果 在 shouldComponentUpdate 或 componentWillUpdate 方 法 中 调 用 setState ， 此 时 this._pendingStateQueue != null，则 performUpdateIfNecessary 方法就会调用 updateComponent 方法进行组件更新，但 updateComponent 方法又会调用 shouldComponentUpdate 和 componentWill- Update 方法，因此造成循环调用，使得浏览器内存占满后崩溃

 ## vitrul diff
 > 源码: /v15.0.0/src/renderers/shared/reconciler/ReactMultiChild.js

 ## React Patch 方法
 所谓 Patch，简而言之就是将 tree diff 计算出来的 DOM 差异队列更新到真实的 DOM 节点上，最终让浏览器能够渲染出更新的数据。而且，React 并不是计算出一个差异就去执 行一次 Patch，而是计算出全部差异并放入差异队列后，再一次性地去执行 Patch 方法完成真实 DOM 的更新。
 
 >Patch 方 法 的 源 码 如 下 ( 源 码 路 径 : /v15.0.0/src/renderers/dom/client/utils/DOMChildrenOperations.js)