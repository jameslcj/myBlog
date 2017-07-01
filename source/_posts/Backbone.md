---
title: Backbone
date: 2017-03-02 21:21:31
tags: JavaScript
---
### backbone依赖
- 依赖`jquery`库 进行dom操作
- 依赖`underscore` 进行数据存储
```
<script type="text/javascript" src="jquery-2.0.3.min.js"></script>
<script type="text/javascript" src="underscore-min.js"></script>
<script type="text/javascript" src="backbone-min.js"></script>
```
### `model` 与 `collection`的区别
- `collection`是多条`model`对象的集合
```
var collect = new Backbone.Collection();
    var model = new Backbone.Model();
    model.set('name', 'bbb')
    model.set({'age': 18})
    collect.add(model)
    alert(JSON.stringify(collect))
```

### model的扩展
- 第一个对象是扩展对象方法
- 第二个对象是扩展静态方法
```
var M = Backbone.Model.extend({
        aaa: function() {
            alert('aaa')
        }
    }, {
        bbb: function() {
            alert('bbb')
        }
    })
    M.bbb();
    var model = new M
    model.aaa();
```

### 初始化默认值
- defaults 默认参数
- initialize 默认方法
```
var M = Backbone.Model.extend({
        defaults: {
            name: 'lc'
        },
        initialize: function() {
        //当有变化时触发 change后面可以添加参数 指定某个参数变化 触发 如change:name
            this.on('change', function() {
                alert('changed')
            })
        }
    })
    var model = new M();
    alert(model.get('name'))
```

### 继承
```
var M = Backbone.Model.extend({
        defaults: {
            name: 'lc'
        }
    })
    var childModel = M.extend();
    var model = new childModel();
    alert(model.get('name'))
```

### 与view结合
```
var M = Backbone.Model.extend({
        defaults: {
            name: 'lc'
        }
    })
    var V = Backbone.View.extend({
        initialize: function() {
            this.listenTo(this.model, 'change', this.show);
        },
        show: function(model) {
            $('body').append('<div>' + model.get('name') + '</div>')
        }
    })
    var m = new M();
    var view  = new V({model: m})
    setTimeout(function() {
        m.set({name: 'll'})
    }, 1000)
```

### save 与 sync联系
```
Backbone.sync = function(method, model) {
        alert(method + ':' + JSON.stringify(model))
        //将method方法改成update
        model.id = 1
    }
    var M = Backbone.Model.extend({
        defaults: {
            name: 'lc'
        },
        url: '/url'
    })
    var model = new M()
    model.save()
    model.save({name: 'll'})
```

### fetch 获取服务器资源
```
Backbone.sync = function(method, model) {
        alert(method + ':' + JSON.stringify(model))
        //将method方法改成update
        // model.id = 1
    }
    var C = Backbone.Collection.extend({
        initialize: function() {
            this.on('reset', function() {
                alert(123)
            })
        },
        url: 'http://www.baidu.com'
    })
    var model = new C()
    model.fetch()
```
### Router
```
var m = Backbone.Router.extend({
        routes: {
            'index': 'index',
            'getData/:id': 'getData',
            'setData/:id/:data': 'setData'
        },
        index: function() {
            alert('index')
        },
        getData(id) {
            alert(id)
        },
        setData(id, data) {
            alert(id + ':' + data )
        }
    })
    var model = new m();
    Backbone.history.start()
```
### 事件委托 
- 对dom操作
```
var V = Backbone.View.extend({
        //模板
        el: $('body'),
        events: {
            //监听事件 dom对象 回调方法
            'click li': 'test'
        },
        test: function() {
            alert('aaa')
        }
    })
    var view = new V
```
### 模板
```
<script type="text/template" id="template">
        <% for (var i = 0; i < 5; i ++) {%>
            <div><%= name %></div>
        <% } %>
    </script>
    
var M = Backbone.Model.extend({
        defaults: {
            name: 'lc'
        }
    })
    var V = Backbone.View.extend({
        initialize: function() {
            this.listenTo(this.model, 'change', this.show);
        },
        show: function(model) {
            //将数据转成json 传递给模板生成
            $('body').append(this.template(this.model.toJSON()))
        },
        // 拿到模板 并获取数据填充
        template: _.template($("#template").html())
    })
    var m = new M();
    var view  = new V({model: m})
    setTimeout(function() {
        m.set({name: 'll'})
    }, 1000)
```
