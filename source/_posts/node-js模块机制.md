---
title: node.js模块机制
date: 2018-01-13 15:53:23
tags: 深入浅出Node.js
---
## 模块编译

module的定义
```js
function Module(id, parent) { 
    this.id = id;
    this.exports = {}; 
    this.parent = parent;
    if (parent && parent.children) {
         parent.children.push(this);
    }
    this.filename = null; 
    this.loaded = false; t
    his.children = [];
}
var module = new Module()
```

node会对引入的不同扩展类型的文件进行不同方式解析
- .js文件. 通过fs模块同步读取文件后编译执行
- .node文件. 这是C/C++编写的扩展文件, 通过dlopen()方法加载最后编译生成的文件
- .json文件. 通过JSON.parse获取结果
- 其他扩展类型. 它们都被当做js文件引入
```js
Module._extensions['.json'] = function(module, filename) {
    var content = NativeModule.require('fs').readFileSync(filename, 'utf8');
    try {
        module.exports = JSON.parse(stripBOM(content));
    } catch (err) {
        err.message = filename + ': ' + err.message;
        throw err;
    }
}

//最后Module._extensions 会被赋值给require, 这样require可以在引入文件的时候, 根据不同类型进行解析
require.extensions = Module._extensions

//如果想对特殊的扩展进行处理, 可进行如下操作
require.extensions['.ext'] = function() {
    //...do something
}
```

### node中的module, require等变量从何而来?
其实node对获取数据通过字符的形式拼接如下, 然后再调用vm原生模块的`runInThisContext()`方法(类似eval, 只是具有明确的上下文, 不污染全局), 返回一个function对象, 最后将`exports, require, module, __filename, __dirname`通过参数传入, 这样每个模块就可以独立作用域了
```js
(function (exports, require, module, __filename, __dirname) {
    var math = require('math');
    exports.area = function (radius) {
    return Math.PI * radius * radius; };
})(exports, require, module, __filename, __dirname)
```

### 为什么需要同时传入exports和module?
因为`exports = module.exports`, `exports`只是一个引用, 当赋值一个对象时, 这个引用就会被覆盖掉, 因此只能使用`module.exports`