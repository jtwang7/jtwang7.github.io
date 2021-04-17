---
title: 前端工程化 - webpack 概念
date: 2021-04-17 10:02
tags: 
- webpack
- ES6
- 面试
categories: webpack
excerpt: 参考 webpack 官方文档, 加上个人思考的一篇学习笔记, 请酌情参考;
---

# webpack 官方文档
[webpack概念](https://www.webpackjs.com/concepts/)

# 概念
本质: webpack 是一个**高度可配置**的现代 **JavaScript** 应用程序**静态模块打包器(module bundler)**。
打包流程: webpack **递归构建一个依赖关系图(dependency graph)**，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。
核心概念: entry, output, loader, plugin;

## entry(入口)
概念: webpack 构建内部依赖图的**起始模块**。webpack 会基于 entry 寻找与入口起点**直接或间接依赖的模块和库**。
> 个人理解: 将 webpack 递归构建依赖关系图看作遍历树, 则 entry 本质上就是 webpack 的递归起点(树的根节点), webpack 寻找各依赖模块和库的过程, 就是遍历搜索节点间关联关系的过程;

### webpack.config.js
`webpack.config.js` 配置: 通过配置 entry 属性，来指定一个入口起点（或多个入口起点）。默认值为 `./src`。

#### 单入口语法
用法：`entry: string|Array<string>`
```
module.exports = {
  entry: {
    main: './src/app.js'
  }
}
```
简写版:
```
module.exports = {
  entry: './src/app.js',
}
```
若向 entry 属性传入「文件路径(file path)数组」, 将创建“**多个同名入口**”, 使得**多个依赖文件**一起注入，并且将它们的依赖导向(graph)到**一个“chunk”**。
> 个人理解: 一个入口对应生成一个 chunk 打包代码块, 若传入数组, 则是将该数组内所有路径的依赖文件注入到一个 chunk 中, 执行时将它们以相同的入口名分别创建, 但注入目标都是同一个 chunk;

```
module.exports = {
  entry: ['./src/app.js', './src/print.js']
}
```

#### 对象语法
用法：`entry: {[entryChunkName: string]: string|Array<string>}`
```
module.exports = {
  entry: {
    app: './src/app.js',
    vendors: './src/vendors.js',
    actions: ['./src/eat.js', './src/move.js']
  }
}
```
```
module.exports = {
  entry: {
    pageOne: './src/pageone.js',
    pageTwo: './src/pagetwo.js',
    pageThree: './src/pagethree.js'
  }
}
```
> 单入口语法其实是对象语法键名为 main 的简写;
> webpack **不同入口创建的依赖关系图彼此完全分离, 互相独立**, 最终会打包成多个 bundle, 每个 bundle 中都对应一个 webpack 引导(bootstrap);

### 总结
entry 单入口语法比较方便, 常用于单页面应用配置;
entry 对象语法则更具扩展性, 它可用于:
1. 应用程序入口和第三方库入口的分离(如上例 app 与 vendors(第三方库) 分离);
2. 多页面应用;


## output(出口)
概念: 整个应用程序结构被编译打包后的指定**输出路径**。你可以通过在配置中指定一个 output 字段，来配置这些处理过程; 

### webpack.config.js
`webpack.config.js` 配置: 通过配置 output 属性，指定 webpack 输出打包的 bundles 文件的位置, 以及如何命名这些文件, 默认值为 `./dist`。

#### 用法
output 属性值接收一个对象，包括以下两点：
* filename: 用于输出文件的文件名
* path: 目标输出目录的**绝对路径**

```
const path = require('path');

module.exports = {
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```
> 将编译打包后的文件以 'bundle.js' 文件名输出到 './dist' 路径;
> path 是 Node.js 核心模块之一, 用于操作文件路径; `path.resolve()` 拼接参数并生成绝对路径, 拼接方式从右到左直到生成绝对路径为止; `__dirname` 表示当前绝对路径;

#### 多入口起点的 output 配置
`webpack.config.js` 中只允许指定一个 output 配置, 因此当遇到多个入口起点时, 需要通过[占位符](https://www.webpackjs.com/configuration/output/#output-filename)实现对应 output 配置;
```
module.exports = {
  entry: {
    pageOne: './src/pageone.js',
    pageTwo: './src/pagetwo.js',
    pageThree: './src/pagethree.js'
  },
  output: {
    filename: '[name].js',
    path: path.resolve(__dirname, 'dist')
  }
}
```
> 占位符在 webpack 中有明文规定, 不是随意取名的, 可以将它理解为一个变量, 用 `[]` 引用; `[name]` 占位符表示入口模块的名称;

## loader
我们已知 webpack 是 JavaScript 应用程序的静态模块打包工具, 因此 **webpack 自身实际上只能理解并处理 JavaScript**;
loader 则为 webpack 提供了**处理非 JavaScript 文件**的能力; 其可以将所有类型的文件转换为 webpack 能够处理的有效模块，然后利用 webpack 的打包能力，对它们进行处理。
本质上，webpack loader 将所有类型的文件，**转换为**应用程序的依赖图（和最终的 bundle）**可直接引用的模块**。
> webpack loader 使代码能够通过 import 导入任何类型的模块(例如 `import color from 'style.css'`), 即上述代码实现是需要通过 webpack loader 支持的, 只有文件被 webpack 打包后, 该行代码才能被正确识别并处理(此时非 JS 文件已经被转换为 bundle 可直接引用的模块了);

### webpack.config.js
loader 在 module 属性对象的 rules 属性值内部定义，包括以下两点：
* test: 被 use 指定 loader 所转换的文件标志符(后缀名), 常用正则表达式作为属性值;
* use: 用于转换 test 指定文件的 loader;

```
npm install --save-dev css-loader
npm install --save-dev ts-loader
```
> 首先通过 npm 安装对应的 loader, 常用 loader 有: style-loader, css-loader, ts-loader, file-loader 等;

```
module.exports = {
  module: {
    rules: [
      {test: /\.txt$/, use: 'css-loader'},
      {test: /\.ts$/, use: 'ts-loader'},
      {test: /\.(png|jpg|gif|svg)$/, use: 'file-loader'},
    ]
  }
}
```
> 一个 loader 对应一个对象类型的配置项, 定义在 module 对象的 rules 属性值数组中;
> 正则表达式中: `\`表示转义, `$`表示末尾匹配;

## plugins
插件用于拓展 webpack 功能; webpack 插件是具有 apply 属性的 JavaScript 对象, 其 apply 属性会被 webpack compiler 调用，并且 compiler 对象可在整个编译生命周期访问。

### webpack.config.js
想要使用一个插件，你需要先通过 npm 安装, 在 `webpack.config.js` 开头 `require()` 它，然后把它添加到 plugins 数组中。
插件可以携带参数/选项，在 webpack 中配置插件，需要向 plugins 属性传入 **new 实例**。
```
const HtmlWebpackPlugin = require('html-webpack-plugin'); // npm 安装
const webpack = require('webpack'); // webpack 内置插件

module.exports = {
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({
      template: './src/index.html'
    })
  ]
}
```
> plugins 属性接收数组, 数组内接收插件实例;
> 插件实例可以初始化传入参数;
