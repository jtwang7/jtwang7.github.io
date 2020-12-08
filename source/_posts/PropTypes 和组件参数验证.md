---
title: React小书学习笔记 —— PropTypes 和组件参数验证
date: 2020-11-17 20:34
tags: React
categories: React

---

# 前言
学习前端一小步，迈向成功一大步！本专栏主要记录学习前端React框架的一些个人心得，分享一些实战教学，如有不足，欢迎交流讨论。React框架的入门教学强推[胡子大哈](https://www.zhihu.com/people/hu-zi-da-ha)的[React小书](http://huziketang.mangojuice.top/books/react/lesson18)，简单易懂还有代码实战。还等什么？让我们开始本篇的前端学习之旅，欢迎各位入坑前端！

<!-- more -->

# PropTypes 和组件参数验证
参考教程：React小书--第24节(PropTypes 和组件参数验证）
教程作者：[胡子大哈](https://www.zhihu.com/people/hu-zi-da-ha)
参考链接：[React小书](http://huziketang.mangojuice.top/books/react/lesson18)
**本文搭配原文教程食用，风味更佳~!**

---

## 为什么需要 React 第三方库: prop-types
JavaScript 的灵活性体现在弱类型、高阶函数等语言特性上。而语言的弱类型一般来说确实让我们写代码很爽，但是也很容易出 bug。
JS 语言在声明变量的时候统一用的是 **let（先前是 var）**，这就意味着**变量是没有固定类型且可以随意赋值的**，假如我起初声明了一个变量`let a={}`，显然 a 是一个对象，但是在工程项目的某处对该共享的对象变量做了改变，如`a=4`，那么 `a` 变量就变成了一个 Number 类型(此处可以看到，共享变量可以通过赋值改变其类型，即应证了上述变量没有固定类型且随意赋值这句话)，此时再使用 `a.xxx` 就会报错，但 debug 的时候不会明确告诉你是这里出错了。

## PropTypes
React.js 提供了一种机制，实现**给组件的配置参数加上类型验证**。
**步骤：**

1. 安装第三方库 **prop-types**:  `npm install --save prop-types`
2. 头文件引入 **PropTypes**: `import PropTypes from 'prop-types'`
3. 在组件中添加类属性 **propTypes**, 属性值接收一个对象：`statics propTypes = {xx: yyy}`
4. 对象键值对写为：`xxx:PropTypes.yyy` ，其中 yyy 为变量类型(object, number,...)
5. 通过 isRequired 关键字强制组件某个参数传值，如`xxx:PropTypes.object.isRequired`

### 实例
```
import React, { Component } from 'react'
import PropTypes from 'prop-types'

class Comment extends Component {
  static propTypes = {
    comment: PropTypes.object
    num: PropTypes.number.isRequired
  }

  render () {
    const { comment } = this.props
    return (
      <div className='comment'>
        <div className='comment-user'>
          <span>{comment.username} </span>：
        </div>
        <p>{comment.content}</p>
      </div>
    )
  }
}
```

### 优势
组件参数验证在构建大型的组件库的时候相当有用。

1. 可以帮助我们迅速定位这种类型错误，让我们组件开发更加规范。
2. 起到了一个说明文档的作用，在使用组件的时候，只要看到组件的 propTypes 就清晰地知道这个组件到底能够接受什么参数，什么参数是可选的，什么参数是必选的。

