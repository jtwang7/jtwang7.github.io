---
title: React小书学习笔记 —— 高阶组件（Higher-Order Components）
date: 2020-11-29 21:23
tags: React
categories: React

---

# 前言
学习前端一小步，迈向成功一大步！本专栏主要记录学习前端React框架的一些个人心得，分享一些实战教学，如有不足，欢迎交流讨论。React框架的入门教学强推[胡子大哈](https://www.zhihu.com/people/hu-zi-da-ha)的[React小书](http://huziketang.mangojuice.top/books/react/lesson18)，简单易懂还有代码实战。还等什么？让我们开始本篇的前端学习之旅，欢迎各位入坑前端！

<!-- more -->

# 高阶组件（Higher-Order Components）
参考教程：React小书--第28节(高阶组件（Higher-Order Components））
教程作者：[胡子大哈](https://www.zhihu.com/people/hu-zi-da-ha)
参考链接：[React小书](http://huziketang.mangojuice.top/books/react/lesson28)
**本文搭配原文教程食用，风味更佳~!**

---

## 什么是高阶组件？
高阶组件是一个函数（而不是组件），它接受一个组件作为参数，返回一个新的组件。例如：
```
import React,{Component} from 'react'

export default (WrappedComponent) => class NewComponent extends Component {
    // 可以做很多自定义逻辑
    render () {
      return <WrappedComponent />
    }
}
```
* 接收 `WrappedComponent` 组件**(组件名首字母大写)** ，返回了 `NewComponent` 新组件，此处因为只有 `class ...` 一个整体语句，所以省略了 `{} 和 return`。
* `WrappedComponent` 代表子组件，是形参名可以更改，但是习惯上用 `WrappedComponent` 来表示。
* 高阶组件是一个函数

## 高阶组件作用
用于代码复用，可以把组件之间可复用的代码、逻辑抽离到高阶组件当中。高阶组件包装组件和被包装组件之间通过 props 传递数据。

## 高阶组件使用示例
```
import React, { Component } from 'react'

export default (WrappedComponent, name) => {
  class NewComponent extends Component {
    constructor () {
      super()
      this.state = { data: null }
    }

    componentWillMount () {
      let data = localStorage.getItem(name)
      this.setState({ data })
    }

    render () {
      return <WrappedComponent data={this.state.data} />
    }
  }
  return NewComponent
}
```
这里我们定义了一个高阶组件，除了传入子组件外，我们还传入了一个参数 name ，在高阶组件内我们实现了从 name 读取数据，并设置 data 参数向子组件传递该值。
子组件调用高阶组件的方法如下所示：

1. import 高阶组件(上述未定义变量名，因此引入的变量自定义名称)
2. 高阶组件是个函数，因此同函数调用类似，在子组件外部套上高阶组件，传入子组件名和参数
3. 导出子组件

```
import wrapWithLoadData from './wrapWithLoadData'

class InputWithUserName extends Component {
  render () {
    return <input value={this.props.data} />
  }
}

InputWithUserName = wrapWithLoadData(InputWithUserName, 'username')
export default InputWithUserName
```

## 编写高阶组件步骤
1. 实现一个普通组件
2. 将普通组件用函数包裹
3. 在函数内用 `return` 将新的组件抛出

编写的方式有两种：
第一种:
```
function funcName(OldComponent) {
    return class NewComponent extends Component {
        render () {
            return (
                <div>
                    <WrappedComponent />
                </div>
            )
        }
    }
}
export default funcName;
```
第二种:
```
export default (WrappedComponent) => class NewComponent extentds Component {
    render () {
        return (
            <div>
                <WrappedComponent />
            </div>
        )
    }
}
```
