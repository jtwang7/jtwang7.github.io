---
title: React的img标签
date: 2020-11-29 17:28
categories: [React]
tags: React
excerpt: 在一个html文件中，img的src属性赋值为相对路径或绝对路径的字符串即可访问到图片。但在jsx文件中，不支持直接在标签内写图片的路径，因此将介绍两种 React 引用本地图片的方法。
---

# React的img标签
参考链接：
[react中img引入本地图片的2种方式](https://www.cnblogs.com/tu-0718/p/12530654.html)
[react项目中关于img标签的src属性的使用](https://www.cnblogs.com/chenbeibei520/p/10930281.html)

---

## 方法一：import
在 js 文件开头用 import 将本地图片从相对路径中引入为变量
```
import homePage from '../static/images/home.png'
```
例如此处将 `"../static/images/"` 路径下的 `home.png` 图片引入为 `homePage` 变量。
然后在 `<img src={}>`中引用该变量名即可。(变量名随意，`{}`必须，因为 jsx 元素要用 `{}` 包裹表达式)
```
<img src={homePage}>
```

## 方法二：require方法
用 require 方法在代码内导入本地图片。
```
const homePage = require('../static/images/home.png')
<img src={homePage}>
```
或者直接
```
<img src={require('../static/images/home.png')}>
```
整体思想同 import 类似，即从相对路径引入本地图片并赋值到一个变量。因此，**require中只能写字符串，不能写变量。（因为 require 接收的是一个路径字符串）**

> 一般推荐使用 import 引入图片。
> 用 require 导入容易存在后续 webpack 打包时读取不到本地图片的情况。
> 但是 require 可以实现动态的加载。


## import 和 require
### 定义
import 和 require 是 react 的两种导入方式。其可以导入图片 / 组件等。
其本质就是从路径地址读取目标，并赋值给一个变量。(js中的export即为导出的接口)

1. import(常用)
`import component from './component'`
2. require
`const component = require('./component')`

### 区别
提出的规范不同
import是ES6语法,reuqire是CommonJs提出的，import会通过babel转换成CommonJS规范。
因此，下面两行代码是等价的
```
import component from './component'
// => 
const component = require('./component')
```
推荐统一规范一种导入方式,防止混乱，当然,不同情况使用的方式也是不一样的。

> 一般来说使用import就够了，但是要注意import需要放在定义组件的外部。这就导致一个问题: 如果我需要通过动态路径动态加载组件，那么我们就要用到 require 的导入方法。
> 当需要实现动态加载图片时，我们往往会在require中引入一个变量，但require中不能直接赋值一个变量，正确做法应该是将`require(path)`拆分成三个部分(即文件路径+名称+后缀)，如下：

```
let homePage = "home";
const data = require('../static/images/' + homePage + '.png');
```