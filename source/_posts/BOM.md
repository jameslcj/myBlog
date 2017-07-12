---
title: BOM
date: 2017-07-11 10:30:24
tags: javaScript高级程序设计笔记
---
## window
### 全局作用域
> 所有全局作用域下声明的变量和函数都会变成`window`的属性和方法

```
var test = 'test';
function getTest() {
  console.log(this.test);
}
console.log(window.test);//"test"
getTest();//"test"
window.getTest();//"test"
```

> 全局变量与直接在`window`上定义的变量的还是有区别的;
直接定义在`window`上的变量可以被`delete`, 但全局变量不能;
根本原因是因为通过`var`声明的变量有一个名为`[Configurable]`的特性, 这个特性的值被设置为`false`, 所以无法删除;

```
var test1 = 'hello';
window.test2 = 'world';
delete window.test1; //IE < 9 直接报错, 其他返回false;
delete window.test2; //IE < 9 直接报错, 其他返回true;
console.log(window.test1);//"hello"
console.log(window.test2);//undefined
```

> 如果直接访问一个没有声明的变量会报错, 但是我们可以通过`window`的属性方式来访问, 避免报错

```
console.log(someVar);//ERROR
console.log(window.someVar);//undefined
```

### 窗口位置
> `screenLeft`与 `screenTop`, `screenX`与`screenY` 都是获取窗口相对于屏幕上边和左边的位置;
也可以使用`moveTo()`, `moveBy()`移动窗口位置, ie7以上默认是禁止的

```
var leftPos = (typeof window.screenLeft == "number") ? window.screenLeft : window.screenX;
var topPos = (typeof window.screenTop == "number") ? window.screenTop : window.screenY;
```

### 窗口大小
> 获取浏览器窗口大小, IE9+可以通过 `innerWidth`, `innerHeight`, `outerWidth`, `outerHeight`获取;
在chrome中, `innerWidth`, `innerHeight`与`outerWidth`, `outerHeight`值相等, 即视口(viewport)大小, 而非浏览器窗口大小;
所有浏览器都可以通过`document.documentElement.clientWidth`, `document.documentElement.clientHeight`获取视口大小;
这些属性都是必须在标准模式下才能有效; 如果在混杂模式下, 就必须通过`document.body.clientWidth`和`document.body.clientHeight`获取视口大小;
对于移动设备(除移动IE), `window.innerWidth`和`windowinnnerHeight`保存着可见视口, `document.documentElement`度量的是布局视口, 即渲染后页面的实际大小;
移动IE浏览器把布局视口信息保存在`document.body.clientWidth`和`document.body.clientHeight`中;
可以使用`resizeTo(x, y)`和`resizeBY(x, y)`跳转浏览器窗口大小, 但是ie7+模式是禁用的;

```
var pageWidth = window.innerWidth;
var pageHeight = window.innerHeight;
if (typeof pageWidth != 'number') {
  if (document.compatMode == 'CSS1Compat') {
    pageWidth = document.documentElement.clientWidth;
    pageHeight = document.documentElement.clientHeight;
  } else {
    pageWidth = document.body.clientWidth;
    pageHeight = document.body.clientHeight;
  }
}
```

### 导航和打开窗口
> `window.open()`有4个参数: 要加载的url地址, 窗口目标, 一个特性字符串以及一个表示新页面是否取代浏览器历史记录中当前加载页面的布尔值;
第二个参数可以指定为其他窗口名, 就会在指定的窗口里打开, 也可以使用特殊的窗口名称: `_self`, `_parent`, `_top`, `_blank`;
如果第二个参数不是一个已经存在的窗口或者框架, 并且传入第三个参数, 就会打开一个带有默认设置(工具栏, 地址栏等)的新浏览器窗口;

- 第三个参数可用参数信息:
![第三个可用参数](https://img.alicdn.com/tfs/TB1VwkhSXXXXXXdXVXXXXXXXXXX-1056-472.png)

> 在ie8和chrome中, 为了能与新打开的页面保持通信, 我们获取了它的引用, 但是为了保持通信, 就不能在独立进程中运行.
所以如果需要独立运行, 就需要将这个引用设置为`null`, 一旦切断, 就没有办法恢复;

```
var newWindow = window.open("http://www.aliyun.com", "newWindow", "height=400, width=400, resizable=yes");
//原始窗口可调用newWindow.close()来关闭新打开窗口
//切断通信, 使新页面独立进程运行
newWindow = null;
```

> 为了安全性, 很多浏览器屏蔽了新建打开浏览器窗口, 我们可以通过判断返回值, 来判断是否被屏蔽了; 如果是浏览器扩展或者其他程序阻止的弹出窗口,
那么window.open()通常会抛出一个错误; 可以用以下代码检测:

```
var blocked = false;
try {
  var newWindow = window.open("http://www.aliyun.com", "_blank");
  if (newWindow == null) {
    blocked = true;
  }
} catch (e) {
  blocked = true;
}

if (blocked) {
  alert("The popup was blocked!");
}
```

## location
> `window.location`和`document.location`引用的是同一个对象

### location属性
> https://www.aliyun.com:80/aliware/?q=something#page=1

| 属性名 | 例子 | 说明 |
| ------| ------ | ------ |
| hash | "#page=1" | #号后面的内容 |
| host | "www.aliyun.com:80" | 返回服务器名称和端口号(如果有) |
| hostname | "www.aliyun.com" | 返回服务器名称 |
| href | "http://www.aliyun.com:80/aliware/?q=something#page=1" | 返回当前加载页完整URL地址和location.toString()相等 |
| pathname | "/aliware/" | 返回URL中的目录和文件名 |
| protocol | "https:" | 返回URL中的目录和文件名 |
| search | "?q=something" | 返回URL的查询字符串. 这个字符串以问号开头 |

### 改变地址的方式

```
window.location = "https://www.aliyun.com";
location.href = "https://www.aliyun.com";
//上面2种方式, 最终还是调用了下面的方法
location.assign("https://www.aliyun.com");
```

### replace
> 上面的方式都会生成浏览历史, 可以通过后退回到之前页面; 使用location.replace('https://www.aliyun.com'), 就无法回退到之前的页面

```
location.replace('https://www.aliyun.com');
```
### reload重新加载页面
> reload可以重新加载页面, 它接受一个boolen参数, 如果传递为true, 强制从服务器重新加载, 否则有可能从缓存中加载

```
location.reload();//重新加载(有可能从缓存中加载)
location.reload(true);//重新加载(从服务器重新加载)
```
