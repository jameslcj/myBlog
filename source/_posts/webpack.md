---
title: webpack
date: 2017-03-02 21:25:37
tags: tools
---
## webpack 主要功能
- 将不同类型的文件打包成一个或多个文件
- 不同类型的文件, 根据loader配置加载, 没有配置对应的loader就被导致导入报错
### entry
- 打包文件地址
- 可以是数组, 引入多个地址.
- 也可以是对象, 引入多个页面
- `webpack-dev-server`默认是不支持html模板刷新的, 可以添加`'webpack-dev-server/client?http://localhost:8080/'`,  实现双向监听, 就可以监听到模板刷新了

### ouput
- 导出地址
- 格式与entry类似
#### 支持参数
- `path`为目录
- `filename` 文件名
    + 当文件名指定时, 将会全部打包到一个文件里
    + 当以`js/[name]-[hash]-[chunhash].js`时, 会根据输入文件生成对应的hash文件

### 插件
- `html-webpack-plugin` 
    + 如果生成的bundle文件带有hash值, 每次都不一样, 那么html页面里就需要每次修改很麻烦, 所以就需要这个插件
    + `template`参数指定模板
    + `inject` js插入地方; false就不自动插入生成好的js
    + `publicPath` 地址前缀, 比如网址
    + 可以使用 `<%= htmlWebpackPlugin.option.参数名 %>`在模板里替换变量
```
plugins: [
    new HtmlwebpackPlugin({
        template: 'index.html',
        inject: 'head'
    })
  ]
```

### 实时更新
- 地址: `http://localhost:8080/webpack-dev-server/bundle`

### 问题
- css图片引入报错
    + 解决办法: 修改正则 将`$ 替换成 (?.*$|$)` 
```
{ 
          test: /\.(eot|png|gif|jpg|woff|woff2|svg|ttf)([\?]?.*)$/, 
          loader: "file-loader" 
      },
```

### 配置
webpack
```
var path = require('path');
var HtmlwebpackPlugin = require('html-webpack-plugin');
//定义了一些文件夹的路径
var ROOT_PATH = path.resolve(__dirname);
var APP_PATH = path.resolve(ROOT_PATH, 'app');
var BUILD_PATH = path.resolve(ROOT_PATH, 'build');

module.exports = {
  //项目的文件夹 可以直接用文件夹名称 默认会找index.js 也可以确定是哪个文件名字
  entry: [
      'webpack-dev-server/client?http://localhost:8080/',
      APP_PATH
  ],
  //输出的文件名 合并以后的js会命名为bundle.js
  output: {
    path: BUILD_PATH,
    filename: 'bundle.js'
  },
  devServer: {
    historyApiFallback: true,
    hot: true,
    inline: true,
    // progress: true,
  },
  module: {
    loaders: [
      {
          test: /\.css$/,
          loader: 'style-loader!css-loader'//添加对样式表的处理
      },
      { 
          test: /\.(eot|png|gif|jpg|woff|woff2|svg|ttf)([\?]?.*)$/, 
          loader: "file-loader" 
      },
    ]
  },
  //添加我们的插件 会自动生成一个html文件
  plugins: [
    new HtmlwebpackPlugin({
        template: path.resolve(APP_PATH, 'index.html'),
        inject: 'head'
    })
  ]
};

```
package.json
```
{
  "name": "test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "webpack-dev-server --hot --inline"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "css-loader": "^0.26.2",
    "file-loader": "^0.10.1",
    "html-webpack-plugin": "^2.28.0",
    "html-withimg-loader": "^0.1.16",
    "style-loader": "^0.13.2",
    "url-loader": "^0.5.8",
    "webpack-dev-server": "^2.4.1"
  }
}

```
