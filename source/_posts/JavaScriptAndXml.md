---
title: JavaScriptAndXml
date: 2017-09-05 11:10:19
tags: javaScript高级程序设计笔记
---
## 浏览器对xml dom的支持
### DOMParser

```
var parser = new  DOMParser()
var xmldom = parser.parseFromString("<root><child/></root>", "text/xml")

xmldom.documentElement.tagName//root
```

