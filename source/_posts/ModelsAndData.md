---
title: Models And Data
date: 2018-02-03 15:05:14
tags: JavaScript Web Applications
---
## ORM
```js
Math.guid = function(){
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
        var r = Math.random()*16|0, v = c == 'x' ? r : (r&0x3|0x8);
        return v.toString(16); 
    }).toUpperCase();
};
var Model = {
    //记录实例对象
    records: {},
    inherited: function(){},
    created: function(){
        //让records不共享
        this.records = {}; 
    },
    find: function(id){
        var record = this.records[id];
        if ( !record ) throw("Unknown record"); 
        //每次返回都重新复制一个对象 防止修改影响到records里的数据
        return record.dup();
    },
    //实例所具有的方法
    prototype: {
        newRecord: true,
        init: function(atts) {
            if (atts) this.load(atts);
        },
        load: function(attributes){ 
            for(var name in attributes)
                this[name] = attributes[name]; 
        },
        create: function(){
            if ( !this.id ) this.id = Math.guid(); 
            this.newRecord = false; 
            this.parent.records[this.id] = this.dup();
        },
        dup: function(){
            return $.extend(true, {}, this);
        },
        destroy: function(){
            delete this.parent.records[this.id];
        },
        update: function(){
            this.parent.records[this.id] = this.dup();
        },
        save: function(){
            this.newRecord ? this.create() : this.update();
        }
    },
    create: function(){
        var object = Object.create(this);
        object.parent = this;
        object.prototype = object.fn = Object.create(this.prototype);
        object.created(); 
        this.inherited(object); 
        return object;
    },
    init: function(){
        var instance = Object.create(this.prototype); 
        instance.parent = this; 
        // console.log("init: ", instance.init)
        instance.init.apply(instance, arguments); 
        return instance;
    },
    include: function(o){
        var included = o.included; 
        $.extend(this.prototype, o); 
        if (included) included(this);
    },
    extend: function(o){
        var extended = o.extended; 
        $.extend(this, o);
        if (extended) extended(this);
    }
};

//获取一个User model
var User = Model.create();

//获取一个user1实例
var user1 = User.init();
user1.id = 1;
user1.name = 'hello';
user1.save();

//获取一个user2实例
var user2 = User.init({
    id: 2,
    name: 'world',
});
user2.save();

var Goods = Model.create();
var goods1 = Goods.init({
    id:1,
    name: 'computer'
})
goods1.save();
var goods2 = Goods.init({
    id:2,
    name: 'mobile'
})
goods2.save()

```