title: Virtual DOM
date: 2017-02-18 20:29:54
tags: [ReactNative,算法]
---
### 为什么要使用Vitrul DOM
- 因为原生dom自带很多属性, 直接操作dom是非常笨重的, 性能效率非常低

### Vitrul DOM 原理
- 因为原生dom过于笨重, 初始化了很多属性, 我们可以将原生dom用js对象来表示, 这个对象里只保存这个dom独有的属性, 省略了很多其他默认的属性
```JavaScript

<ul id='list'>
  <li class='item'>Item 1</li>
  <li class='item'>Item 2</li>
  <li class='item'>Item 3</li>
</ul>
//上面的js表示如下
var element = {
  tagName: 'ul', // 节点标签名
  props: { // DOM的属性，用一个对象存储键值对
    id: 'list'
  },
  children: [ // 该节点的子节点
    {tagName: 'li', props: {class: 'item'}, children: ["Item 1"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 2"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 3"]},
  ]
}

```
- Virturl Dom 会把旧dom和新dom, 用js对象生成两棵树进行比较之间的差异
- 正如你所预料的，比较两棵DOM树的差异是 Virtual DOM 算法最核心的部分，这也是所谓的 Virtual DOM 的 diff 算法。两个树的完全的 diff 算法是一个时间复杂度为 O(n^3) 的问题。但是在前端当中，你很少会跨越层级地移动DOM元素。所以 Virtual DOM 只会对同一个层级的元素进行对比
- 在比较过程, 记录每个节点的差异, 比如是节点元素类型变换了, 还是增删改了属性值等等
- 最后对有变化的dom进行操作

### 如果dom只是重新排序了? 利用上面的方法, 性能也会很差
- 这个问题抽象出来其实是字符串的最小编辑距离问题（Edition Distance），最常见的解决算法是 Levenshtein Distance，通过动态规划求解，时间复杂度为 O(M * N)。但是我们并不需要真的达到最小的操作，我们只需要优化一些比较常见的移动情况，牺牲一定DOM操作，让算法时间复杂度达到线性的（O(max(M, N))。具体算法细节比较多，这里不累述，有兴趣可以参考[代码](https://github.com/livoras/list-diff/blob/master/lib/diff.js)

### 为什么组件上要加上唯一的key标识
- 因为tagName是可重复的，不能用这个来进行对比。所以需要给子节点加上唯一标识key，列表对比的时候，使用key进行对比，这样才能复用老的 DOM 树上的节点

### 参考学习
- [戴嘉华: 深度剖析：如何实现一个 Virtual DOM 算法](https://github.com/livoras/blog/issues/13)
