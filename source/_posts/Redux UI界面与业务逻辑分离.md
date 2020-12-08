---
title: Redux UI界面与业务逻辑分离
date: 2020-12-03 11:23
tags: [Redux, React]
categories: [React]
excerpt: Redux是一个用来管理管理数据状态和UI状态的JavaScript应用工具。在当前页面复杂度越来越高的互联时代，React组件的状态管理以及状态提升给管理带来了极大的难度，而Redux就是降低管理难度的。但注意的一点是，Redux除了支持React之外，还能支持Angular、jQuery甚至纯JavaScript。本专题将结合博主“技术胖”的Redux免费教程内容，对自身学习做一个总体的总结，整个专题的编写过程将按照教程内容进行编排，主要针对实战代码的重点做主要总结。
---


# Redux UI界面与业务逻辑分离
视频教程：[技术胖 Redux 免费教程](https://www.bilibili.com/video/BV1w441137ss?p=1)
参考链接：[技术胖 Redux 教程笔记汇总](https://jspang.com/detailed?id=48#toc230)
**本文搭配原文教程食用，风味更佳~!**

---

**UI界面与业务逻辑分离的必要性**
1. 让项目更容易维护。
2. 多人协作，实现超大型项目的开发和快速上线。比如两个人同时写一个模块，一个写UI部分，一个写业务逻辑部分，之后两个人在一起整合。

## 分离步骤
1. `src/` 文件夹下新建 `TodoListUI.js`，用于存储 UI 界面代码
2. 将 JSX 代码迁移到 `TodoListUI.js`
3. UI组件与业务逻辑组件整合
4. 进阶：无状态组件编写UI界面

## 实战代码
### TodoListUI.js
* 在src目录下新建一个文件`TodoListUI.js`, `imrc ccc`快速生成页面的基本结构.
* 去`TodoList.js`里把`JSX`拷贝过来，并在`TodoList.js`中引入`<TodoListUI />`标签。此时`TodoListUI`中并没有组件所需要的`state`(状态信息)，接下来需要改造父组件进行值传递。
* 父组件通过属性传值的形式(props)，把需要的值传递给子组件，子组件接收这些值，进行相应的绑定就可以了。

```
import React, { Component } from 'react';
// 别忘了引入antd组件
import "antd/dist/antd.css";
import { Input, Button, List } from "antd";

class TodoListUI extends Component {
    constructor(props) {
        super(props);
    }
    render() {
        return (
            <div>
                {/* 迁移UI界面JSX代码 */}
                <div style={{ margin: "10px" }}>
                    <Input
                        // - placeholder={this.state.inputValue} 
                        // + placeholder={this.props.inputValue}
                        // 属性值从TodoList组件通过 <TodoListUI /> 的props传入
                        placeholder={this.props.inputValue}
                        style={{ width: "250px", marginRight: "10px" }}
                        onChange={this.props.inputChangeValue}
                    />
                    <Button
                        type="primary"
                        onClick={this.props.addItem}
                    >添加</Button>
                </div>
                <div style={{ width: "325px", margin: "10px" }}>
                    <List
                        size="small"
                        bordered
                        dataSource={this.props.list}
                        /*
                        不能写成(item, index) => (<List.Item onClick={(index) => { this.props.deleteItem(index) }}>{item}</List.Item>)
                        因为在onClick={}代码块中，又重新声明了一个index，此时this.props.deleteItem(index)取的是代码块内的index
                        正确写法如下，将(index)=>{}改为()=>{}，让外层代码块的index传入进来。
                        */
                        // this.props.deleteItem.bind(this,index) 在这里无法使用，因为无法绑定到TodoList组件的this
                        // 所以推荐将 .bind(this) 写入原组件的 constructor
                        renderItem={(item, index) => { return (<List.Item onClick={() => { this.props.deleteItem(index) }}>{item}</List.Item>) }}
                    />
                </div>
            </div>
        );
    }
}

export default TodoListUI;
```

### TodoList.js
```
import React, { Component } from 'react';
import store from './store/index';
import { inputChangeValueAction, addItemAction, deleteItemAction } from "./store/actionCreators"
// 引入 UI 界面代码
import TodoListUI from './TodoListUI'


class TodoList extends Component {
    constructor(props) {
        super(props);
        this.inputChangeValue = this.inputChangeValue.bind(this)
        this.addItem = this.addItem.bind(this)
        this.deleteItem = this.deleteItem.bind(this)
        this.state = store.getState()

        this.storeChange = this.storeChange.bind(this)
        store.subscribe(this.storeChange)
    }

    inputChangeValue(e) {
        const action = inputChangeValueAction(e.target.value)
        store.dispatch(action)
    }

    addItem() {
        const action = addItemAction()
        store.dispatch(action)
    }

    deleteItem(index) {
        const action = deleteItemAction(index)
        store.dispatch(action)
    }

    storeChange() {
        this.setState(store.getState())
    }

    render() {
        return (
            <div>
                {/* 用组件替代 */}
                <TodoListUI
                    inputValue={this.state.inputValue}
                    inputChangeValue={this.inputChangeValue}
                    addItem={this.addItem}
                    list={this.state.list}
                    deleteItem={this.deleteItem}
                />
            </div>
        );
    }
}

export default TodoList;
```

## 无状态组件
无状态组件其实就是一个函数，它不用再继承任何的类（class），也不存在state（状态）。当我们需要的组件**不需要其余业务逻辑**的时候，即只是构造**单纯的UI组件**时，把其改成无状态组件可以提高程序性能。

1. 不在需要引入React中的{ Component }。
2. 用一个TodoListUI函数代替组件继承, 里边只返回JSX的部分。
3. 函数传递一个props参数，之后修改里边的所有props，去掉this (因为函数中已经没有 this 了，原 this 指向的是继承类)。

```
// - import React, { Component } from 'react';
// Component 不需要，但是 React 在 React 项目中必须引入
import React from 'react';
import "antd/dist/antd.css";
import { Input, Button, List } from "antd";

// 无状态组件是个函数，返回 UI 界面的 JSX 
const TodoListUI = (props) => {
    return (
        <div>
            {/* 
            此处将props打印到控制台，发现props为obj对象
            即<TodoListUI />组件在属性传递时，是将所有属性打包成了一个对象进行传递的。
            */}
            {console.log(props)}
            <div style={{ margin: "10px" }}>
                <Input
                    // - this.props.inputValue
                    // + props.inputValue
                    placeholder={props.inputValue}
                    style={{ width: "250px", marginRight: "10px" }}
                    onChange={props.inputChangeValue}
                />
                <Button
                    type="primary"
                    onClick={props.addItem}
                >添加</Button>
            </div>
            <div style={{ width: "325px", margin: "10px" }}>
                <List
                    size="small"
                    bordered
                    dataSource={props.list}
                    renderItem={(item, index) => { return (<List.Item onClick={() => { props.deleteItem(index) }}>{item}</List.Item>) }}
                />
            </div>
        </div>
    );
}

export default TodoListUI;
```