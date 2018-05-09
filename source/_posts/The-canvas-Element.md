---
title: The canvas Element
date: 2018-05-01 11:22:47
tags: Core HTML5 Canvas
---
### css设置canvas大小和canvas元素节点上设置大小区别
> css只会设置元素大小, 无法改变画布表面大小
> 元素节点设置不仅改变元素大小, 也改变画布表面大小

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        body {
            background: aqua;
        }
        #canvas {
            background: #fff;
            /* width: 600px;
            height: 600px; */
            margin-left: 200px;
        }
    </style>
</head>
<body>
    <canvas id="canvas" width="600" height="600"></canvas>
    <img id="img"/>
</body>
<script>
    var canvas = document.getElementById('canvas'),
    context = canvas.getContext('2d'),
    FONT_HEIGHT = 15,
    MARGIN = 35,
    HAND_TRUNCATION = canvas.width/25,
    HOUR_HAND_TRUNCATION = canvas.width/10,
    NUMERAL_SPACING = 20,
    RADIUS = canvas.width/2 - MARGIN,
    HAND_RADIUS = RADIUS + NUMERAL_SPACING;
    
    function drawCicle() {
        context.beginPath();
        context.arc(canvas.width / 2, canvas.height / 2, RADIUS, 0, 2*Math.PI, true);
        context.stroke();
    }
    function drawCenter() {
        context.beginPath();
        context.arc(canvas.width / 2, canvas.height / 2, 5, 0, 2 * Math.PI, true);
        context.fill();
    }
    function drawHand(loc, isHour, lineWidth) {
        var angle = 2 * Math.PI * (loc / 60) - Math.PI / 2;
        handRadius = isHour ? RADIUS - HAND_TRUNCATION-HOUR_HAND_TRUNCATION : RADIUS - HAND_TRUNCATION;

        context.moveTo(canvas.width / 2, canvas.height / 2);
        context.lineTo(canvas.width / 2 + handRadius * Math.cos(angle), canvas.height / 2 + handRadius * Math.sin(angle));
        context.lineWidth = lineWidth;
        context.stroke();
    }
    function drawHands() {
        var date = new Date();
        var hour = date.getHours();

        hour = hour > 12 ? hour - 12 : hour;

        drawHand(hour * 5 + date.getMinutes() / 60 * 5, true, 5);
        drawHand(date.getMinutes(), false, 2);
        drawHand(date.getSeconds(), false, 0.3);
    }
    function drawNumbers() {
        var numberList = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12];
        var angle = 0;
        var numeralWidth = 0;

        numberList.forEach(function(number) {
            angle = 2 * Math.PI * (number - 3) / 12;
            numeralWidth = context.measureText(number).width;
            context.fillText(number, canvas.width / 2 + HAND_RADIUS * Math.cos(angle) - numeralWidth / 2, canvas.height / 2 + HAND_RADIUS * Math.sin(angle) - FONT_HEIGHT / 3);
        });
    }
    function drawClock() {
        context.clearRect(0, 0, canvas.width, canvas.height);
        drawCicle();
        drawCenter();
        drawHands();
        drawNumbers();
    }
    context.font = FONT_HEIGHT + 'px Arial';
    // drawClock();
    // setInterval(drawClock, 1000);
    var image = new Image();
    image.src = "https://mdn.mozillademos.org/files/225/Canvas_drawimage.jpg";
    canvas.onmousedown = function (event) {
        var oriX = event.clientX, oriY = event.clientY;
        var bbox = canvas.getBoundingClientRect();
        window.onmousemove = function (event) {
            var x = event.clientX, y = event.clientY;
            context.clearRect(0, 0, canvas.width, canvas.height);
            context.strokeRect(oriX - bbox.left, oriY - bbox.top, x - oriX, y - oriY);
            window.onmouseup = function (event) {
                var x = event.clientX, y = event.clientY;
                window.onmousemove = null;
                window.onmousedown = null;
                context.drawImage(image, oriX - bbox.left, oriY - bbox.top, x - oriX, y - oriY)
                setTimeout(function() {
                    var src = canvas.toDataURL("image/png");
                    console.log(src);
                    var img = document.getElementById("img");
                    img.setAttribute('crossOrigin', 'anonymous');
                    img.src = src;
                }, 1000)
            }
        }
    }
</script>
</html>
```

### 使用向量计算碰撞
#### 单元向量 unit vector
```js
var vectorMagnitude = Math.sqrt(Math.pow(vector.x, 2) +
                                  Math.pow(vector.y, 2)),
      unitVector = new Vector();
      unitVector.x = vector.x / vectorMagnitude;
      unitVector.y = vector.y / vectorMagnitude;
```

#### 向量相加 相减
```js
var vectorSum = new Vector();
  vectorSum.x = vectorOne.x + vectorTwo.x;
  vectorSum.y = vectorOne.y + vectorTwo.y;

var vectorSubtraction = new Vector();
  vectorSubtraction.x = vectorOne.x - vectorTwo.x;
  vectorSubtraction.y = vectorOne.y - vectorTwo.y;
```

#### The Dot Product of Two Vectors 向量点积 计算2个物品是否会碰撞
```js
var dotProduct = vectorOne.x * vectorTwo.x + vectorOne.y * vectorTwo.y; //正数同方向, 负数反方向
```

### 计算速度
```
pixels / frame = (X ms / frame) * (Y pixels / second);
// ===> 推到出
pixels / frame = X*Y / 1000;
```