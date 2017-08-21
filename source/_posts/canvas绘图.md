---
title: canvas绘图
date: 2017-08-21 09:41:45
tags: javaScript高级程序设计笔记
---

## 基本用法

```
<canvas id="drawing" width="200" height="200"></canvas>

var drawing = document.getElementById("drawing");
if (drawing.getContext) {
	var ctx = drawing.getContext("2d");
}
```

> toDataUrl(MIME类型格式) 可以将canvas导出为图片

```
var drawing = document.getElementById("drawing");
if (drawing.getContext) {
	var imgUrl = drawing.toDataUrl("image/png");
	var image = document.createElment("img");
	image.src = imgUrl;
	document.body.appendChild(image);
}
```

## 2D上下文
### 填充和描边
- strokeStyle 描边颜色
- fillStyle 填充颜色

```
var drawing = document.getElementById("drawing");
if (drawing.getContext) {
	var ctx = drawing.getContext("2d");
	ctx.strokeStyle = 'red';
	ctx.fillStyle = '#ccc';
}
```

### 绘制矩形
> fillRect(), strokeRect(), clearRect() 都支持4个参数, 矩形的x坐标, 矩形y坐标, 矩形的宽度, 矩形的高度

```
var drawing = document.getElementById("drawing");
if (drawing.getContext) {
	var ctx = drawing.getContext("2d");
	//先指定描边颜色
	ctx.strokeStyle = 'red';
	//再执行描边区域
	ctx.strokeRect(10, 10, 50, 50);

	//先指定填充颜色
	ctx.fillStyle = '#ccc';
	//在执行填充区域
	ctx.fillRect(100, 100, 50, 50);

	//清楚画布
	ctx.clearRect(10, 10, 150, 150);
}
```

### 绘制路径
> 必须先调用`beginPath()`方法, 表示要开始绘制新路径

- arc(x, y, radius, 开始弧度, 结束弧度, 是否逆时针) 以(x, y)为中心绘制弧线
- arcTo(x1, y2, x2, y2, radius) 从上一个点开始绘制一条弧线, 到(x2, y2)为止, 并且以给定的半径radius穿过(x1, y1)
- bezierCurveTo(c1x, c1y, c2x, c2y, x, y); 从上一个点开始绘制一条比萨尔曲线到(x, y), 并且趋向于(c1x, c1y)和(c2x, c2y)
- lineTo(x, y) 从上一点到(x, y)绘制一条直线
- moveTo(x, y) 将绘制游标移动到(x, y) 不画线
- quadraticCurveTo(cx, cy, x, y) 从上一点开始绘制一条二次曲线, 到(x, y)为止, 并以(cx, cy) 为控制点
- rect(x, y, width, height) 绘制一个矩形路径, 与fillRect和strokeRect有区别, 这两者是独立的图形
- isPointInPath(x, y) 在路径被关闭前可以用此方法来判断点是否在路径上

> 绘制好路径后, 如果想绘制一条链接到路径起点的线条, 可以调用 `closePath()`; 如果想填充路径, 可以调用`fill()`方法, 可以调用`stroke()`方法描边, 最后可以调用`clip()`, 这个方法可以在路径上创建一个剪切区域



