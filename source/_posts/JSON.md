---
title: JSON
date: 2017-09-06 10:46:26
tags: javaScript高级程序设计笔记
---
## 解析与序列化

### 序列化选项
> `JSON.stringify`第二个参数可以是个数组, 过滤掉其他参数, 只留下指定参数; 

```
var obj = {a: 'a', b: 'b', c:'c'}
JSON.stringify(obj, ['a','c']) //"{"a":"a","c":"c"}"
```

> 第二个参数也可以是个方法

```
var book = {
    "title": "Professional JavaScript",
    "authors": ["Nicholas C. Zakas"],
    edition: 3,
    year: 2001
};
var jsonText = JSON.stringify(book, function(key, value) {
    switch(key) {
        case "authors":
            return value.join(",");
        case "year":
            return 5000;
        case "edition":
            return undefined;
        default:
            return value;
    }
});
jsonText; //"{"title":"Professional JavaScript","authors":"Nicholas C. Zakas","year":5000}"
```

> 第三个参数, 如果是数值可以指定缩进几个空格(最大缩进10), 也可以指定是字符串

```
var book = {
    "title": "Professional JavaScript",
    "authors": ["Nicholas C. Zakas"],
    edition: 3,
    year: 2001
};
var jsonText = JSON.stringify(book, null, 4);
/** 
"{
    "title": "Professional JavaScript",
    "authors": [
        "Nicholas C. Zakas"
    ],
    "edition": 3,
    "year": 2001
}"**/

JSON.stringify(book, null, '-');
/**
"{
-"title": "Professional JavaScript",
-"authors": [
--"Nicholas C. Zakas"
-],
-"edition": 3,
-"year": 2001
}"*/
```

- toJSON

```
var book = {
    "title": "Professional JavaScript",
    "authors": ["Nicholas C. Zakas"],
    edition: 3,
    year: 2001,
    toJSON: function() {
        if (this.edition > 3) {
            return 'big';
        } else {
            return 'small';
        }
    }
}

JSON.stringify(book) //"small"
```

## 解析选项
### JSON.parse()