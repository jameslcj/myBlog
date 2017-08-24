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

### 绘制文本
- fillText(要绘制的文本字符串, x坐标, y坐标, 最大像素宽度)
- strokeText(要绘制的文本字符串, x坐标, y坐标, 最大像素宽度)
- font
- textAlign
- textBaseline 文本基线

```
ctx.font = 'bold 14px Arial';
ctx.textAlign = 'center';
ctx.textBaseline = 'middle';
ctx.fillText("xx", 100, 20);
```

### 变化
- rotate(angle) 围绕原点逆时针旋转图像angle弧度
- scale(scaleX, scaleY) 缩放图像
- translate(x, y) 将坐标原点移动到(x, y); 执行变化后, 坐标(0, 0)会变成之前由(x, y)表示的点
- transform(m1_1, m1_2, m2_1, m2_2, dx, dy) 直接修改变化矩阵, 方式乘以如下矩阵
m1_1 m1_2 dx
m2_1 m2_2 dy
0    0    1
- setTransform(m1_1, m1_2, m2_1, m2_2, dx, dy) 将变化矩阵重置为默认状态, 然后再调用transform()

> fillStyle strokeStyle等属性, 如果不被修改, 在上下文中一直有效; 如果修改后, 想恢复刚才的上下文, 我们可以在修改前, 只有`save`方法把刚才的上下文推到`save`堆栈里, 需要恢复的时候再调用`restore`弹栈恢复

- save() 
- restore()

```
document.body.innerHTML = `<canvas id="canvas" widht="500" height="500" ></canvas>`
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext('2d');
ctx.fillStyle = '#ff0000'
ctx.save();
ctx.fillStyle = '#00ff00';
ctx.translate(100, 100);
ctx.save();
ctx.fillStyle = '#0000ff';
ctx.fillRect(0, 0, 100, 200);
ctx.restore();
ctx.fillRect(10, 10, 100, 200);
ctx.restore();
```

### 绘制图像
- drwaImage(图片资源, 起点x, 起点y, [绘制宽度, 绘制高度, ...]) 总共可传9个参数, 可以控制截取源目标图片大小和区域, 在指定位置, 展现指定大小

```
var image = document.images[0];
ctx.drwaImage(image, 10, 10)
```

### 阴影
- shadowColor
- shadowOffsetX
- shadowOffsetY
- shadowBlur

```
document.body.innerHTML = `<canvas id="canvas" widht="500" height="500" ></canvas>`
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext('2d');
ctx.shadowColor = '#ccc';
ctx.shadowOffsetX = 5;
ctx.shadowOffsetY = 5;
ctx.shadowBlur = 4;
ctx.fillStyle = '#ff0000';
ctx.fillRect(10, 10, 50, 50)
```

### 渐变
- createLinearGradient(起点x, 起点y, 终点x, 终点y)
- addColorStop([0-1], 颜色)

```
document.body.innerHTML = `<canvas id="canvas" widht="500" height="500" ></canvas>`
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext('2d');
var gradient = ctx.createLinearGradient(30, 30, 70, 70);
gradient.addColorStop(0, 'white')
gradient.addColorStop(1, 'black')
ctx.fillStyle = gradient;
ctx.fillRect(30, 30, 50, 50)
```

- createRadialGradient(x1, y1, r1, x2, y2, r2)

```
document.body.innerHTML = `<canvas id="canvas" widht="500" height="500" ></canvas>`
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext('2d');
var gradient = ctx.createRadialGradient(30, 30, 10, 30, 30, 30);
gradient.addColorStop(0, 'white')
gradient.addColorStop(1, 'black')
ctx.fillStyle = gradient;
ctx.fillRect(0, 0, 60, 60)
```

### 模式
- createPartern(img元素/video元素/其他canvas, repeat/no-repeat/repeat-x/repeat-y)

### 使用图像数据
- getImageData(x, y, width, height)
- putImageData(imageData, x, y)

### 合成
- globalAlpha 值0-1, 指定绘制的透明度

```
document.body.innerHTML = `<canvas id="canvas" widht="500" height="500" ></canvas>`
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext('2d');

ctx.globalAlpha = 0.5

var gradient = ctx.createRadialGradient(30, 30, 10, 30, 30, 30);
gradient.addColorStop(0, 'white')
gradient.addColorStop(1, 'black')
ctx.fillStyle = gradient;
ctx.fillRect(0, 0, 60, 60)
```

- globalCompositeOperation 表示绘制的图形怎么和先绘制的图形结合 其值如下
![对应的值](https://img.alicdn.com/tfs/TB10aVaXgoQMeJjy0FpXXcTxpXa-1944-1444.png)

```
document.body.innerHTML = `<canvas id="canvas" widht="500" height="500" ></canvas>`
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext('2d');var gradient = ctx.createLinearGradient(30, 30, 70, 70);
gradient.addColorStop(0, 'white')
gradient.addColorStop(1, 'black')
ctx.fillStyle = gradient;
ctx.fillRect(30, 30, 50, 50)

ctx.globalCompositeOperation = 'destination-over'

var gradient = ctx.createRadialGradient(30, 30, 10, 30, 30, 30);
gradient.addColorStop(0, 'white')
gradient.addColorStop(1, 'black')
ctx.fillStyle = gradient;
ctx.fillRect(0, 0, 60, 60)

```

## WebGL
### 类型化数组
```
var buffer = new ArrayBuffer(20);
// 获取大小字节
var bytes = buffer.byteLength;
```

#### 视图
- DataView(buffer, 开始位置, 要选字节数)

```
var buffer = new ArrayBuffer(20);

//基于整个缓冲器创建一个视图
var view = new DataView(buffer)

//创建一个从字节9开始的视图
var view = new DataView(buffer, 9)

//创建一个从字节9开始到18字节的视图
var view = new DataView(buffer, 9, 10)
view.byteOffset; //9
view.byteLength; //10
```

- setter和getter方法 第一个参数是一个字节偏移量, 最后一个参数是布尔值, 表示数据的最低有效位是否保存在高内存地址中
![getter和setter方法](https://img.alicdn.com/tfs/TB1.8IrXgoQMeJjy1XaXXcSsFXa-1802-1362.png)

```
var buffer = new ArrayBuffer(20);
var view = new DataView(buffer)
view.setUnit16(0, 25);
view.setUnit16(2, 50); //只能从2开始 因为16位占2个字节
view.getUnit8(0);//0
view.getUnit8(1);//25
view.getUnit16(0);//25 一个性取2位, 以0位为高位 00000000 00011001
view.getUnit16(1);//6400 因为它以1位为高位 00011001 00000000

view.setUnit16(0, 25, true);
view.getUnit8(0);//25
view.getUnit8(1);//0
```

#### 类型化视图
- 类型化数组 第一个参数为ArrayBuffer对象, 第二个参数为起点字节偏移量, 第三个参数包含的字节数
![类型化数组](https://img.alicdn.com/tfs/TB1UFsBXogQMeJjy0FiXXXhqXXa-796-462.png)

```
var buffer = new ArrayBuffer(20);
var int8s = new Int8Array(buffer)
//从字节4开始
var int16s = new Int16Array(buffer, 4)
//从字节4到8字节
var int16s = new Uint16Array(buffer, 4, 5)
```

- 不用先创建ArrayBuffer

```
//创建10个8位整数(10字节)
var int8s = new Int8Array(10)
//创建10个16位整数(20字节)
var int8s = new Int16Array(10)

//直接创建一个数组 保存4个8位整数
var int8s = new Int8Array([1, 2, 3, 4]) //[1, 2, 3, 4]
//越界
var int8s = new Int8Array([1, 2, 129, 4])// [1, 2, -127, 4]
```

- subarray(开始元素的索引, 可选的结束索引) 基于底层数组缓冲器的子集创建一个新视图

```
var u = new Int16Array(10)
u.subarray(2, 5)
```
