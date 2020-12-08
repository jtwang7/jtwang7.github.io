---
title: React小书学习笔记 —— style 属性
date: 2020-11-17 20:19
tags: React
categories: React

---

# 前言
学习前端一小步，迈向成功一大步！本专栏主要记录学习前端React框架的一些个人心得，分享一些实战教学，如有不足，欢迎交流讨论。React框架的入门教学强推[胡子大哈](https://www.zhihu.com/people/hu-zi-da-ha)的[React小书](http://huziketang.mangojuice.top/books/react/lesson18)，简单易懂还有代码实战。还等什么？让我们开始本篇的前端学习之旅，欢迎各位入坑前端！

<!-- more -->

# style 属性
参考教程：React小书--第23节(dangerouslySetHTML 和 style 属性）
教程作者：[胡子大哈](https://www.zhihu.com/people/hu-zi-da-ha)
参考链接：[React小书](http://huziketang.mangojuice.top/books/react/lesson18)
**本文搭配原文教程食用，风味更佳~!**

---
dangerouslySetHTML 主要是防止跨站脚本攻击(XSS)，这个属性不必要的情况就不要使用。所以此处不再赘述，有需要可以在小书中学习。

* HTML 中的 style 属性
`<h1 style='font-size: 12px; color: red;'>React.js 小书</h1>`

* React.js 中的 style 属性
`<h1 style={{fontSize: '12px', color: 'red'}}>React.js 小书</h1>`

对比可看到，React.js 中需要把 CSS 属性变成一个对象，再传给标签元素。此外，`font-size`等 HTML 中 `-` 的表示需要替换为驼峰命名法 `fontSize`。

**总结：**

1. style 属性接收一个对象，对象为原 CSS 属性的样式
2. 样式名采用驼峰命名法

---
为什么要采用对象的方法传递样式参数？
答：用对象作为 style 方**便动态设置元素的样式**。我们可以用 `props` 或者 `state` 中的数据生成样式对象再传给元素，然后用 `setState` 就可以修改样式，非常灵活，例如 `<h1 style={{fontSize: '12px', color: this.state.color}}>React.js 小书</h1>`，此处我只需修改 `setState({color:blue})` 即可更改样式颜色为蓝色。

