---
title: MVCandClasses
date: 2018-01-14 16:11:51
tags: JavaScript Web Applications
---
## What Is MVC?
### The Model
主要存储对象数据的地方

因为是面向对象编程, 因此我们任何数据被model对象封装后, 都可以直接调用其方法, 而不依赖全局方法, 这样可以减少很多全局方法冲突
```js
//使用全局变量的方法
var user = users["foo"]; 
destroyUser(user);

//建议封装为如下
var user = User.find("foo"); 
user.destroy();
```

### The View
主要是展示页面, 和其他逻辑解耦

```js
// helper.js
var helper = {};
helper.formatDate = function(){ /* ... */ };

// template.html 
<div>
    ${ helper.formatDate(this.date) } 
</div>
```

### The Controller
controller是model和view的中间粘合剂, 它从view中获取events和input, 再从model中获取相应数据, 返回给view展示

```js
// Use a anonymous function to enscapulate scope 
(Controller.users = function($){
var nameClick = function(){ 
    /* ... */
};
// Attach event listeners on page load $(function(){
$("#view .name").click(nameClick); });
})(jQuery);
```

## Toward Modularity, Creating Classes
js中的class是用原型链模拟的

```js
var Class = function(parent) {
      var klass = function() {
          this.init.apply(this, arguments);
      }
      if (parent) {
          var subClass = function() {};
          subClass.prototype = parent.prototype;
          klass.prototype = new subClass();
      }

      klass.fn = klass.prototype;

      klass.fn.init = function() {}

      klass.fn.parent = klass;

      klass.fn._super = klass.__proto__;

      // Adding a proxy function 
      klass.proxy = function(func) {
        var self = this; 
        return(function() {
            return func.apply(self, arguments); 
        });
      }
      // Add the function on instances too 
      klass.fn.proxy = klass.proxy;  

      klass.extend = function(obj) {
          for (var i in obj) {
              klass[i] = obj[i];
          }
      }

      klass.include = function(obj) {
          for (var i in obj) {
              klass.prototype[i] = obj[i];
          }
      }

      return klass;
}
var Animal = new Class;
Animal.include({ 
    init: function() {
        console.log('I am animal')  
    },
    breath: function(){
        console.log('breath'); 
    }
});
var animal = new Animal;
animal.breath(); //I am animal //breath
var Cat = new Class(Animal)
Cat.prototype.init = function() {
    console.log('I am Cat')
}
// Usage
var tommy = new Cat; 
tommy.breath();//I am Cat //breath
```

### bind polyfill
```js
if (!Function.prototype.bind) {
    Function.prototype.bind = function(context) {
        //self 就是bind的调用者 context是调用者想绑定上下文
        var self = this,
            slice = [].prototype.slice,
            //获取额外要传递的参数
            args = slice.call(arguments, 1),
            //用来做间接继承的
            nop = function() {};
        //将bind的调用者的原型 赋值给nop
        nop.prototype = self.prototype;
        
        //bound 会覆盖bind调用者
        var bound = function() {
            //这里是函数被调用时 会生效的逻辑
            //通过apply将上下文 指向 nop(因为继承了bind调用者的原型)或者context
            //然后函数调用时传递的参数合并到之前传递进来的参数数组里
            return self.apply(this instanceof nop ? this : (context || {}), args.concat(slice(arguments)))
        }

        //将bind调用者的原型赋值给bound的原型, 这样bound也拥有了bind调用者的属性和方法, 而且具有改变上下文指向的能力
        bound.prototype = new nop();
        
        return bound;
    }
} 
```