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