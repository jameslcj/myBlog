---
title: Drawing
date: 2018-05-06 17:00:48
tags: Core HTML5 Canvas
---

## createLinearGradient(oriX, oriY, desX, desY)
```js
var canvas = document.getElementById('canvas'),
    context = canvas.getContext('2d');
function createLinearGradient() {
    var linearGradient = context.createLinearGradient(0, 0, canvas.width, canvas.height);

    linearGradient.addColorStop(0, 'blue');
    linearGradient.addColorStop(0.25, 'yellow');
    linearGradient.addColorStop(0.5, 'green');
    linearGradient.addColorStop(0.75, 'white');
    linearGradient.addColorStop(1, 'black');
    context.fillStyle = linearGradient;
    context.fillRect(0, 0, canvas.width, canvas.height);
}
createLinearGradient()
```

## createRadialGradient(oriX, oriY, oriR, desX, desY, desR)
```js
function createRadialGradient() {
    var linearGradient = context.createRadialGradient(canvas.width / 2, canvas.height, 10, canvas.width / 2, 0, 100);

    linearGradient.addColorStop(0, 'blue');
    linearGradient.addColorStop(0.25, 'yellow');
    linearGradient.addColorStop(0.5, 'green');
    linearGradient.addColorStop(0.75, 'white');
    linearGradient.addColorStop(1, 'black');
    context.fillStyle = linearGradient;
    context.fillRect(0, 0, canvas.width, canvas.height);
}
createRadialGradient()
```

## createPattern(HTMLImageElement | HTMLCanvasElement | HTMLVideoElement image, 'repeat|repeat-x|repeat-y|no-repeat')
```js
    function createCanvasPattern() {
        var SHADOW_COLOR = 'rgba(0,0,0,0.7)';
        var image  = new Image();
        image.src = "https://img.alicdn.com/tfs/TB1sLMKe8fH8KJjy1XbXXbLdXXa-300-300.jpg";
        image.onload = function() {
            var pattern = context.createPattern(image, "no-repeat");
            context.clearRect(0, 0, canvas.width, canvas.height);
            context.fillStyle = pattern;
            context.shadowColor = SHADOW_COLOR;
            context.shadowOffsetX = 10;
            context.shadowOffsetY = 10;
            context.shadowBlur = 10;
            context.fillRect(0, 0, canvas.width, canvas.height);
        }
    }
    createCanvasPattern();
```

## Paths, Stroking, and Filling
- arc(x, y, r, oriPI, desPI, true/false(是否顺时针))
- beginPath() 开始新的路径绘图
- closePath() 闭合路径
- fill() 根据fillStyle填充图形
- rect(double x, double y, double width, double height) 绘制矩形路径
- stroke() 根据strokeStyle绘制路径


## 划线
> 如果1px宽度的线刚好落在像素的边界上, 会导致线的宽度变成2px

```js
context.lineWidth = 1;
context.beginPath();
context.moveTo(50, 10);
context.lineTo(450, 10);
context.stroke();
context.beginPath();
context.moveTo(50.5, 50.5);
context.lineTo(350.5, 50.5);
context.stroke();
```

## 划虚线
```js
function drawDashedLine(context, x1, y1, x2, y2, dashLen = 5) {
    var dx = x2 - x1;
    var dy = y2 - y1;
    var dashNum = 0;

    if (dx === 0) {
        dashNum = Math.abs(dy / dashLen);
    } else if (dy === 0) {
        dashNum = Math.abs(dx / dashLen);
    } else {
        dashNum = Math.floor(Math.sqrt(dx * dx, dy * dy) / dashLen);
    }

    context.beginPath();
    for (var i = 0; i < dashNum; i++) {
        context[i % 2 === 0 ? "lineTo" : "moveTo"](x1 + dx / dashNum * i, y1 + dy / dashNum * i);
    }
    context.stroke();
}

drawDashedLine(context, 20, 20, 220, 20);
drawDashedLine(context, 220, 20, 220, 280);
drawDashedLine(context, 220, 280, 20, 280);
drawDashedLine(context, 20, 280, 20, 20);
```

### CanvasRenderingContext2D

```js
var moveToFunction = CanvasRenderingContext2D.prototype.moveTo;
    CanvasRenderingContext2D.prototype.lastMoveToLocation = {};
    CanvasRenderingContext2D.prototype.moveTo = function (x, y) {
        moveToFunction.apply(context, [x,y]);
        this.lastMoveToLocation.x = x;
        this.lastMoveToLocation.y = y;
    };
    CanvasRenderingContext2D.prototype.dashedLineTo = function (x, y, dashLength) {
        dashLength = dashLength === undefined ? 5 : dashLength;
        var startX = this.lastMoveToLocation.x;
        var startY = this.lastMoveToLocation.y;
        var deltaX = x - startX;
        var deltaY = y - startY;
        var numDashes = Math.floor(Math.sqrt(deltaX * deltaX
                                    + deltaY * deltaY) / dashLength);
        for (var i=0; i < numDashes; ++i) {
            this[ i % 2 === 0 ? 'moveTo' : 'lineTo' ]
                (startX + (deltaX / numDashes) * i,
                    startY + (deltaY / numDashes) * i);
        }
        this.moveTo(x, y);
    };

    context.lineWidth = 3;
    context.strokeStyle = 'blue';
    context.moveTo(20, 20);
    context.dashedLineTo(context.canvas.width-20, 20);
    context.dashedLineTo(context.canvas.width-20,
                        context.canvas.height-20);
    context.dashedLineTo(20, context.canvas.height-20);
    context.dashedLineTo(20, 20);
    context.dashedLineTo(context.canvas.width-20,
    context.canvas.height-20);

    context.stroke();
```

## Line joins
- miter(default)
- bevel
- round

### arcTo(x1, y1, x2, y2, radius)
```js
(function () {
        function roundedRect(cornerX, cornerY,
            width, height, cornerRadius) {
            if (width > 0) 
                context.moveTo(cornerX + cornerRadius, cornerY);
            else 
                context.moveTo(cornerX - cornerRadius, cornerY);
                
            context.arcTo(cornerX + width, cornerY,
                cornerX + width, cornerY + height,
                cornerRadius);
            context.arcTo(cornerX + width, cornerY + height,
                cornerX, cornerY + height,
                cornerRadius);
            context.arcTo(cornerX, cornerY + height,
                cornerX, cornerY,
                cornerRadius);
            if (width > 0) {
                context.arcTo(cornerX, cornerY,
                    cornerX + cornerRadius, cornerY,
                    cornerRadius);
            }
            else {
                context.arcTo(cornerX, cornerY,
                    cornerX - cornerRadius, cornerY,
                    cornerRadius);
            }
        }

        function drawRoundedRect(strokeStyle, fillStyle, cornerX, cornerY,
            width, height, cornerRadius) {
            context.beginPath();
            roundedRect(cornerX, cornerY, width, height, cornerRadius);
            context.strokeStyle = strokeStyle;
            context.fillStyle = fillStyle;
            context.stroke();
            context.fill();
        }

        drawRoundedRect('blue',   'yellow',  50,  40,  100,  100, 10);
        drawRoundedRect('purple', 'green',  275,  40, -100,  100, 20);
        drawRoundedRect('red',    'white',  300, 140,  100, -100, 30);
        drawRoundedRect('white',  'blue',   525, 140, -100, -100, 40);

    })()
```