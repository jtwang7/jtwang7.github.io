---
title: react-router-dom 路由
date: 2020-11-29 16:15
tags: [React,react-router-dom]
categories: React
excerpt: React 多页面跳转时，需要用到 react-router-dom 第三方库提供的路由跳转功能。其具体的使用方法及步骤总结如下。
---

# react-router-dom运用
参考链接：[React：react-router-dom 详解](https://www.cnblogs.com/lovels/p/11574700.html)

---
## 第一步：安装 react-router-dom 第三方库
```
npm install --save react-router-dom
```
在 React 框架 `/src` 文件夹下新建 `router.js`，存放路由设置代码。

## 第二步：导入
```
import React from 'react'
import {BrowserRouter, Route, Switch} from 'react-router-dom'
```
在 `router.js` 文件开头导入 **BrowserRouter, Route, Switch**。

## 第三步：设置并导出路由
```
import Home from './pages/home'
import Car from './pages/car'
import User from './pages/user'
...

export default ()=>(
    <BrowserRouter>
        <Switch>
            <Route path={'/home'} component={Home}></Route>
            <Route path={'/car'} component={Car}></Route>
            <Route path={'/user'} component={User}></Route>
            ...
        </Switch>
    </BrowserRouter>
)
```
> `<Route path={} component={}></Route>` 可以理解为将相应组件绑定到了对应的路由上。


# 注意事项
1. **书写顺序：** `<BrowserRouter> -> <Switch> -> <Route>`，其中 `<Route>` 可以存在多条，每条`<Route>`指向一条跳转路径。
2. **`<Route>`参数：** path 参数对应跳转页面的相对路径，component 参数对应相应页面的组件。注意将组件从相应位置导入，否则路由不知道 component 设置的变量是什么意思。
3. **关于箭头函数()=>()：** 在ES6中规定，如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用 return 语句返回。此处的 JSX 元素需要用`()`包裹，从而可以视为一个条语句，省略 `{}` 和 `return` 。使用`()`包裹jsx，而不要用`{}`包裹，准备杨可以避免js中换行自动插入`;`的问题。


---
# Link 标签实现路由跳转
## 第一步：从 react-router-dom 引入 Link
```
import {Link} from react-router-dom
```

## 第二步：使用 Link 标签
```
<Link to='/more'></ Link>
```
实现效果：点击后跳转到 `'./more'` 界面。