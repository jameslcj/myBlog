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