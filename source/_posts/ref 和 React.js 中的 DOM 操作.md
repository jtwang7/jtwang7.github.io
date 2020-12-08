---
title: React小书学习笔记 —— ref 和 React.js 中的 DOM 操作
date: 2020-11-16 17:16
tags: React
categories: React

---

# 前言
学习前端一小步，迈向成功一大步！本专栏主要记录学习前端React框架的一些个人心得，分享一些实战教学，如有不足，欢迎交流讨论。React框架的入门教学强推[胡子大哈](https://www.zhihu.com/people/hu-zi-da-ha)的[React小书](http://huziketang.mangojuice.top/books/react/lesson18)，简单易懂还有代码实战。还等什么？让我们开始本篇的前端学习之旅，欢迎各位入坑前端！

<!-- more -->

# ref 和 React.js 中的 DOM 操作
参考教程：React小书--第21节(ref 和 React.js 中的 DOM 操作）
教程作者：[胡子大哈](https://www.zhihu.com/people/hu-zi-da-ha)
参考链接：[React小书](http://huziketang.mangojuice.top/books/react/lesson18)
**本文搭配原文教程食用，风味更佳~!**

---

## ref
React.js 提供了一系列的 on* 方法 (例如`onClick()`等) 帮助我们进行事件监听，所以 React.js 当中不需要直接调用 `addEventListener` 的 DOM API，因此避免了大量与DOM元素交互的操作。但是，React.js 仍然提供了调用 DOM API 的一些方法，以方便人们对 DOM 元素进行一些自定义的操作。
React.js 提供了**ref 属性**来帮助我们**获取已挂载元素的 DOM 节点**，你可以给某个 **JSX 元素加上 ref属性**。
> JSX元素：在JavaScript语言内部编写的HTML标签结构，本质上是JS对象。JSX内部(即形如HTML标签结构内部)可以插入任何JS语法(这是成立的，因为JSX就是个JS对象，接受JS语法)，包括变量，表达式计算，函数以及 JSX 元素等，前提是要用`{}`包裹。JSX元素经过编译和构造，在React中转为JavaScript对象(为什么需要先转为JS对象？因为获得JS对象后可以进行更多的操作，例如react-canvas等，不一定要转为DOM元素)，再经过ReactDOM.render()即可转为真正的DOM元素。

### 实例
```
class AutoFocusInput extends Component {
  componentDidMount () {
    this.input.focus()
  }

  render () {
    return (
      <input ref={(input) => this.input = input} />
    )
  }
}

ReactDOM.render(
  <AutoFocusInput />,
  document.getElementById('root')
)
```
ref **属性**
作用：获取**已挂载**元素的 DOM 节点
添加位置：同JSX元素属性设置，即`<input ref={...}>`
属性值：ref 属性**接收一个函数**，当 JSX 元素在页面上挂载完成以后，React.js 就会调用 JSX元素内部 ref 属性定义的函数，并且把这个挂载以后的 DOM 节点传给这个函数(在实例中为`(input)中的input`，input 可以换成任意名称，因为这仅仅代表一个形参)。

在上述实例中，DOM 节点在 JSX元素挂载完成后被传给了`this.input`，这样我们就可以通过 this.input 获取到这个 DOM 元素。然后我们就可以在 componentDidMount 中使用这个 DOM 元素，并且调用 this.input.focus() 的 DOM API。整体就达到了页面加载完成就自动 focus 到输入框的功能。

## 注意事项
1. React.js 支持给任意代表 **HTML 元素标签以及组件标签**加上 ref 从而**获取到它 DOM 元素然后调用 DOM API**。但是记住一个原则：**能不用 ref 就不用**。特别是要避免用 ref 来做 React.js 本来就可以帮助你做到的页面自动更新的操作和事件监听。多余的 DOM 操作其实是代码里面的“噪音”，不利于我们理解和维护。
2. 组件标签中添加 ref 属性，属性值函数获取的形参值为组件标签在 React.js 内部初始化的实例(即通过 constructor 定义的最初始组件渲染后的实例)，这并不是什么常用的做法，而且也并不建议这么做。