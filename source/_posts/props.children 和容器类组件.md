---
title: React小书学习笔记 —— props.children 和容器类组件
date: 2020-11-17 14:18
tags: React
categories: React

---

# 前言
学习前端一小步，迈向成功一大步！本专栏主要记录学习前端React框架的一些个人心得，分享一些实战教学，如有不足，欢迎交流讨论。React框架的入门教学强推[胡子大哈](https://www.zhihu.com/people/hu-zi-da-ha)的[React小书](http://huziketang.mangojuice.top/books/react/lesson18)，简单易懂还有代码实战。还等什么？让我们开始本篇的前端学习之旅，欢迎各位入坑前端！

<!-- more -->

# props.children 和容器类组件
参考教程：React小书--第22节(props.children 和容器类组件）
教程作者：[胡子大哈](https://www.zhihu.com/people/hu-zi-da-ha)
参考链接：[React小书](http://huziketang.mangojuice.top/books/react/lesson18)
**本文搭配原文教程食用，风味更佳~!**

---

## 容器类组件
容器类组件是一种功能性组件，其充当了容器的作用，它定义了一种外层结构形式，允许开发者往容器里添加任意的内容。
首先，我们介绍未使用 `props.children` 的容器组件，通过代码了解它存在什么问题以及 `props.children` 的作用。
```
class Card extends Component {
  render () {
    return (
      <div className='card'>
        <div className='card-content'>
          {this.props.content}
        </div>
      </div>
    )
  }
}

ReactDOM.render(
  <Card content={
    <div>
      <h2>React.js 小书</h2>
       <div>开源、免费、专业、简单</div>
      订阅：<input />
    </div>
  } />,
  document.getElementById('root')
)
```
Card 是我们定义的容器类组件，其内部结构为`<div ...><div className='card-content'>...</ div></ div>`，即用块状元素定义了一个容器，卡片内容通过 `this.props` 从外部接收，即实现让开发者自定义添加容器内容的效果。
在`ReactDOM.render()`渲染过程中，我们传入 props 参数，其为一个 JSX 元素(`{<div>...</ div>}`)(是否还记得，之前提及 JSX 元素内部用`{}`包裹可以接收另一个 JSX 元素)，然后 Card 内部会通过 `this.props.content` 将内容渲染到页面上。

*  **存在的问题：** 如果 Card 除了 content 以外还能传入其他属性的话，那么这些 JSX 和其他属性就会混在一起，很不好维护，如下，不仔细看我都不知道还有个`onClick`属性：
```
<Card content={
    <div>
        ...
    </ div>
}, onClick={...}>
```

---
我们希望改变这种写法，让组件接收的 JSX 元素独立作为一部分，但又与当前组件相关联，于是乎 React.js 提供了 `props.children` 写法

## props.children
`props.children`使得组件标签也能像普通的 HTML 标签那样编写内嵌的结构，具体使用方法如下代码所示：
```
class Card extends Component {
  render () {
    return (
      <div className='card'>
        <div className='card-content'>
          {this.props.children}
        </div>
      </div>
    )
  }
}

ReactDOM.render(
  <Card>
    <h2>React.js 小书</h2>
    <div>开源、免费、专业、简单</div>
    订阅：<input />
  </Card>,
  document.getElementById('root')
)
```
先看 `ReactDOM.render()` 实现了什么：
可以看到，相比于原先将容器类组件的内容通过 `props` 参数传入 JSX 元素的方法，该种方法**直接将内容作为一个HTML内嵌结构编写，将组件参数与内嵌结构分开写**，而`<card></ card>`容器组件标签则起到了和`<div></ div>`类似的作用。
所有容器类组件内部嵌套的 JSX 结构都通过组件内的 `props.children` 来获取，即上例中的`this.props.children`。

### 内部实现原理
我们将 `props.children` 打印出来，可以发现，其包含的其实是个数组，**React.js 就是把我们嵌套的 JSX 元素一个个都放到数组当中，然后通过 props.children 传给了 Card。**
![](/img/posts_img/20201117195904513_10716.png)
**由于 JSX 会把插入表达式里面数组中的 JSX 一个个罗列下来显示。** 所以其实就相当于在 Card 中嵌套了 JSX 结构，都会显示在 Card 的类名为 card-content 的 div 元素当中。

### 优点
按照这种HTML内嵌式的编写方法：

1. 结构清晰，将内嵌 JSX 结构与组件标签参数分离。
2. `props.children` 将内嵌 JSX 结构变成数组的机制，使得我们在编写组件时十分灵活，我们可以选择数组中某些 JSX 元素(而非全部选择，例如`this.props.children[1]`)，然后将其安置在不同的位置。使我们能够选择内套元素，并进行高度复用。
3. 大型 React.js 项目在编写容器型组件时非常常用。
