---
title: The canvas Element
date: 2018-05-01 11:22:47
tags: Core HTML5 Canvas
---
### css设置canvas大小和canvas元素节点上设置大小区别
> css只会设置元素大小, 无法改变画布表面大小
> 元素节点设置不仅改变元素大小, 也改变画布表面大小

### 速度计算
```
pixels / frame = (X ms / frame) * (Y pixels / second);
// ===> 推到出
pixels / frame = X*Y / 1000;
```