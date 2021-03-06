---
title: canvas基本用法
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

#### webgl上下文
- getContext("webgl", 参数如下)

![webgl参数](https://img.alicdn.com/tfs/TB1xr3vXmFRMKJjy0FhXXX.xpXa-1818-518.png)

```
document.body.innerHTML = `<canvas id="canvas" widht="500" height="500" ></canvas>`
var canvas = document.getElementById("canvas");
var gl = canvas.getContext("webgl", {alpha: false})
```

#### 准备绘图
> 先用实色清除cavas, 为绘图做准备

- clearColor(r, g, b, o) 每个参数的值在0-1之间

```
document.body.innerHTML = `<canvas id="canvas" widht="500" height="500" ></canvas>`
var canvas = document.getElementById("canvas");
var gl = canvas.getContext('webgl');
//设置颜色
gl.clearColor(0, 0, 0, 1)
//使用上面定义的颜色清楚

#### 视口与坐标

> 视口坐标定义是以左下角为坐标原点; 视口内部的坐标系是以中心为原点, 右上是(1, 1), 左下为(-1, -1)

- viewport(x, y, width, height)
gl.clear(gl.COLOR_BUFFER_BIT) 
```

#### 缓冲区
> 顶点信息保存在JavaScript的类型化数组中, 使用之前必须先转换到WebGL的缓冲区; 在页面重载之前, 缓冲区始终保持在内存中, 或是调用`gl.deleteBuffer(buffer)`释放内存

- bufferData 最后一个参数如下
	+ gl.STATIC_DRAW 数据只加载一次, 在多次绘图中使用
	+ gl.STREAM_DRAW 数据只加载一次, 在几次绘图中使用
	+ gl.DYNAMIC_DRAW 数据动态改变, 在多次绘图中使用

```
var buffer = gl.createBuffer();
//将buffer与webgl上下文绑定
gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([0, 0.5, 1]), gl.STATIC_DRAW);
gl.deleteBuffer(buffer);
```

#### 错误

> webgl 不会主动抛出错误, 需要通过`getError()`方法获取

```
var error = gl.getError();
console.log(error)
```

#### 着色器
> 用的是GLSL语言, 一种类C语言
> webgl有两种着色器: 顶点着色器和片段着色器 顶点着色器将3D顶点转换为需要渲染的2D. 片段着色器用于准备计算要绘制的每个像素颜色
为着色器传递数据的方式有两种: Attribute和Uniform

- Attribute 向顶点着色器传入顶点信息

```
// 定义了一个类型为vec2的aVertexPosition变量名
attribute vec2 aVertexPosition;
void main() {
	//转换为3D  vec4 表示接受4个值 aVertexPosition是vec2类型 所有算2个值
	gl_position = vec4(aVertexPosition, 0.0, 1.0);
}
```

- Uniform 可以向任何着色器传入常量值

```
//
uniform vec4 uColor;
void main() {
	//绘图时的颜色
	gl_FragColor = uColor;
}
```

#### 编写着色器程序
> 因为浏览器不识别GLSL语法, 所以需要将其变成字符串进行转换

- gl.createShader(gl.VERTEX_SHADER/gl.FRAGMENT_SHADER)


```
<script type="x-webgl/x-vertex-shader" id="vertexShader">
attribute vec2 aVertexPosition;
void main() {
	//转换为3D  vec4 表示接受4个值 aVertexPosition是vec2类型 所有算2个值
	gl_position = vec4(aVertexPosition, 0.0, 1.0);
}
</script>
var vertexShaderText = document.getElementById("vertexShader").text;
var vertexShader = gl.createShader(gl.VERTEX_SHADER)
gl.shaderSource(vertexShader, vertexShaderText);
gl.compileShader(vertexShader)

<script type="x-webgl/x-fragment-shader" id="fragmentShader">
attribute vec2 aVertexPosition;
void main() {
	//转换为3D  vec4 表示接受4个值 aVertexPosition是vec2类型 所有算2个值
	gl_position = vec4(aVertexPosition, 0.0, 1.0);
}
</script>

var fragmentShaderText = document.getElementById("fragmentShader").text;
var fragmentShader = gl.createShader(gl.FRAGMENT_SHADER)
gl.shaderSource(fragmentShader, fragmentShaderText);
gl.compileShader(fragmentShader)

//创建程序
var program = gl.createProgram()
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);
//将两个对象链接到着色器程序中
gl.linkProgram(program)


//通知webgl使用这个程序
gl.useProgram(program);

```

#### 为着色器传入值
```
//找到内存地址
var uColor = gl.getUniformLocation(program, "uColor");
//给uColor赋值
gl.uniform4fv(uColor, [0, 0, 0, 1]);

//找到attribute在内存中的位置
var aVertexPosition = gl.getAttribLocation(program, "aVertexPosition");
//启用
gl.enableVertexAttribArray(aVertexPosition);
//创建指针, 指向gl.bindBuffer()指定的缓冲区, 并将其保存在aVertexPosition中, 以便顶点着色器使用
gl.vertextAttribPointer(aVertexPosition, itemSize, gl.FLOAT, false, 0 , 0);
```
#### 调试着色器和程序

```
//编译成功返回true 否则返回false
if (! gl.getShaderParameter(vertexShader, gl.COMPILE_STATUS)) {
	//获取错误信息
	alert(gl.getShaderInfoLog(vertexShader));
}
```

#### 绘图
> webgl只能绘制3种形状: 点, 线和三角

- gl.drawArrays() 数组缓冲区
- gl.drawElements 元素数组缓冲区
![第一个参数选项](https://img.alicdn.com/tfs/TB1IRGuawoQMeJjy0FnXXb8gFXa-2022-1360.png)

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<canvas id="canvas" widht="500" height="500" ></canvas>
</body>
<script type="x-webgl/x-vertex-shader" id="vertexShader">
attribute vec2 aVertexPosition;
void main() {
	//转换为3D  vec4 表示接受4个值 aVertexPosition是vec2类型 所有算2个值
	gl_position = vec4(aVertexPosition, 0.0, 1.0);
}
</script>


<script type="x-webgl/x-fragment-shader" id="fragmentShader">
attribute vec2 aVertexPosition;
void main() {
	//转换为3D  vec4 表示接受4个值 aVertexPosition是vec2类型 所有算2个值
	gl_position = vec4(aVertexPosition, 0.0, 1.0);
}
</script>
<script type="text/javascript">
var canvas = document.getElementById("canvas");
var gl = canvas.getContext('webgl');
var vertexShaderText = document.getElementById("vertexShader").text;
var vertexShader = gl.createShader(gl.VERTEX_SHADER)
gl.shaderSource(vertexShader, vertexShaderText);
gl.compileShader(vertexShader)

var fragmentShaderText = document.getElementById("fragmentShader").text;
var fragmentShader = gl.createShader(gl.FRAGMENT_SHADER)
gl.shaderSource(fragmentShader, fragmentShaderText);
gl.compileShader(fragmentShader)

//创建程序
var program = gl.createProgram()
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);
//将两个对象链接到着色器程序中
gl.linkProgram(program)


//通知webgl使用这个程序
gl.useProgram(program);


var vertices = new Float32Array([0, 1, 1, -1, -1, -1]),
buffer = gl.createBuffer(),
vertexSetSize = 2,
vertexSetCount = vertices.length / vertexSetSize,
uColor, aVertexPosition;

//数据放到缓冲区
gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);

//为片段着色器传入颜色值
uColor = gl.getUniformLocation(program, "uColor");
gl.uniform4fv(uColor, [0, 0, 0, 1]);

//为着色器传入顶点信息
aVertexPosition = gl.getAttribLocation(program, "aVertexPosition");
gl.enableVertexAttribArray(aVertexPosition);
gl.vertexAttribPointer(aVertexPosition, vertexSetSize, gl.FLOAT, false, 0, 0);

//绘制三角形
gl.drawArrays(gl.TRIANGLES, 0, vertexSetCount);
</script>
</html>
```
