---
title: Vue
date: 2017-03-06 09:10:03
tags: JavaScript
---
### Vue对象
- data 绑定数据
```
 data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      items: JSON.parse(localStorage.getItem('data')) || [],
      newItem: ''
    }
  },
```
- methods vue对象方法
```
methods: {
      clickEvent: function (item) {
          item.flag = !item.flag
      },
      addNew: function() {
          console.log(this.newItem)
          this.items.push({
              message: this.newItem,
              flag: false
          })
          this.newItem = ''
      }
  }
 ```
- watch 监听某个变量, 当发生变化时, 回调函数
```
watch: {
     items: {
         handler: function (val, oldVal) {
             localStorage.setItem('data', JSON.stringify(val))
         },
         deep: true
     } 
  },
```
- props 注册的属性, 不注册既是传递了属性在 组件里也无法获取
- component 需要注册的组件, 不注册只引入组件是无效的
    + 在模板里, 组件的名字需要被反驼峰使用`MyComponent => <my-component></my-component>`

### 重要指令
- v-text 渲染数据(不会解析html标签)
- v-html 渲染数据(会渲染html标签)
```
<template>
  <div class="hello">
    //两者等价
    <h1>{{ title }}</h1>
    <h1 v-text="title"></h1>
  </div>
</template>
<script>
export default {
  name: 'hello',
  data () {
    return {
      title: 'this is title <div>div</div>'
    }
  }
}
</script>
```
- v-if 控制显示
- v-on 绑定事件
```
<li v-for="item in items" v-on:click="clickEvent(item)">{{item.message}}</li>
<script>
export default {
  name: 'hello',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      items: [{message: 'react'}]
    }
  },
  methods: {
      clickEvent: function (item) {
          alert(item.message)
      }
  }
}
</script>

```
- v-for 循环渲染
- v-bind 设置属性
```
<li v-for="item in items" v-bind:class="{under: item.flag}">
            {{item.message}}
        </li>
<script>
export default {
  name: 'hello',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      items: [{message: 'vue', flag: false}, {message: 'vue', flag: true}]
    }
  }
}
</script>
<style scoped>
.under {
    text-decoration: line-through;
}
</style>
```
- v-model 双向数据绑定
    + `<input v-model="newItem" v-on:keyup.enter="addNew">` newItem 就是绑定的变量属性, 可以操作此值进行双向绑定
```
<input v-model="newItem" v-on:keyup.enter="addNew">
<script>
export default {
  name: 'hello',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      items: [],
      newItem: ''
    }
  },
  methods: {
      clickEvent: function (item) {
          item.flag = !item.flag
      },
      addNew: function() {
          console.log(this.newItem)
          this.newItem = ''
      }
  }
}
</script>
```
### 监听其他组件事件
- `this.$emit(其他组件监听事件名称, 传递参数)` 全局监听事件
- `$dispatch()` 事件沿着父链冒泡
- `$broadcast()` 广播事件, 事件会向下传递给所有子组件
- 父级监听方式 `<sub-comp on:父级监听事件名称="回调方法"></sub-comp>`
