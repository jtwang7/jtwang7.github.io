---
title: Redux工程优化
date: 2020-12-02 20:08
tags: [Redux, React]
categories: [React]
excerpt: Redux是一个用来管理管理数据状态和UI状态的JavaScript应用工具。在当前页面复杂度越来越高的互联时代，React组件的状态管理以及状态提升给管理带来了极大的难度，而Redux就是降低管理难度的。但注意的一点是，Redux除了支持React之外，还能支持Angular、jQuery甚至纯JavaScript。本专题将结合博主“技术胖”的Redux免费教程内容，对自身学习做一个总体的总结，整个专题的编写过程将按照教程内容进行编排，主要针对实战代码的重点做主要总结。
---

# Redux工程优化
视频教程：[技术胖 Redux 免费教程](https://www.bilibili.com/video/BV1w441137ss?p=1)
参考链接：[技术胖 Redux 教程笔记汇总](https://jspang.com/detailed?id=48#toc230)
**本文搭配原文教程食用，风味更佳~!**

---

## Action Type 分离
写Redux Action的时候，我们写了很多Action的派发，产生了很多Action Types。
```
inputChangeValue(e) {
    const action = {
        type: "input_change_value",
        value: e.target.value,
    }
    store.dispatch(action)
}

addItem() {
    const action = {
        type: "add_item",
    }
    store.dispatch(action)
}

deleteItem(index) {
    const action = {
        type: "delete_item",
        index,
    }
    store.dispatch(action)
}
```


在项目管理中，不分离 action.type 会导致两个问题：

1. Types不统一管理，不利于大型项目的复用，设置会长生冗余代码。
2. 因为Action里的Type，一定要和Reducer里的type一一对应在，所以这部分代码或字母写错后，浏览器里并没有明确的报错，这给调试带来了极大的困难。

### actionTypes.js
因此，我们需要将组件派发的 `action.type` 单独分离成一个 `actionTypes.js` 文件，`actionTypes.js` 文件放在 `src/store/` 目录下。
```
/*
将 action.type 分离成一个单独的文件 (actionTypes.js)
1.分离后能精确定位错误原因 (原书写方法不会报错，难debug)，避免名称写错难定位的问题
2.增强代码复用性：可以在多个组件引用 action.type 的变量
*/
// 要 export 抛出文件接口(此文件为常量)；常量要大写
// 其他组件使用时不要忘记 import 
export const INPUT_CHANGE_VALUE = "input_change_value"
export const ADD_ITEM = "add_item"
export const DELETE_ITEM = "delete_item"
```

同时，我们需要修改组件`TodoList.js`以及 Reducer `reducer.js`中的 `action.type` 引用。**注意要在开头将 `actionTypes.js` 引入。**

**`TodoList.js`**
```
import React, { Component } from 'react';
import "antd/dist/antd.css";
import { Input, Button, List } from "antd";
import store from './store/index';
// 在使用其他文件内容时，不要忘记 import 接口
import { INPUT_CHANGE_VALUE, ADD_ITEM, DELETE_ITEM } from './store/actionTypes'


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
        const action = {
            // - type: "input_change_value"  + type: INPUT_CHANGE_VALUE
            type: INPUT_CHANGE_VALUE,
            value: e.target.value,
        }
        store.dispatch(action)
    }

    addItem() {
        const action = {
            type: ADD_ITEM,
        }
        store.dispatch(action)
    }

    deleteItem(index) {
        const action = {
            type: DELETE_ITEM,
            index,
        }
        store.dispatch(action)
    }

    storeChange() {
        this.setState(store.getState())
    }

    render() {
        return (
            <div>
                <div style={{ margin: "10px" }}>
                    <Input
                        placeholder={this.state.inputValue}
                        style={{ width: "250px", marginRight: "10px" }}
                        onChange={this.inputChangeValue}
                    />
                    <Button
                        type="primary"
                        onClick={this.addItem}
                    >添加</Button>
                </div>
                <div style={{ width: "325px", margin: "10px" }}>
                    <List
                        size="small"
                        bordered
                        dataSource={this.state.list}
                        renderItem={(item, index) => { return (<List.Item onClick={() => { this.deleteItem(index) }}>{item}</List.Item>) }}
                    />
                </div>
            </div>
        );
    }
}

export default TodoList;
```

**`reducer.js`**
```
// reducer.js 中也不要忘记修改 action.type
import { INPUT_CHANGE_VALUE, ADD_ITEM, DELETE_ITEM } from './actionTypes'

const defaultState = {
    inputValue: "Write Something",
    list: [
        "第一天",
        "第二天",
        "第三天",
    ]
}

export default (state = defaultState, action) => {
    if (action.type === INPUT_CHANGE_VALUE) {
        let newState = JSON.parse(JSON.stringify(state))
        newState.inputValue = action.value
        return newState
    }

    if (action.type === ADD_ITEM) {
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.push(state.inputValue)
        return newState
    }

    if (action.type === DELETE_ITEM) {
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.splice(action.index,1)
        return newState
    }

    return state
}
```


## 管理 Redux Action
工程项目中各组件里有很多Action，并且分散才程序的各个地方，如果庞大的工程，这势必会造成严重的混乱。因此，在工程中通常将所有的Redux Action放到一个文件里进行管理。
我们通常通过以下步骤将 `action` 对象单独分离出来：

1. 在 `src/store/` 目录下新建 `actionCreators.js` 文件。(还记得吗？Redux 官方工作流程中管理 Action 的就是 actionCreators)
2. 将 `action对象` 写入 `actionCreators.js` 中，并以方法的形式(普通函数或箭头函数)抛出。
3. 引入方法，替换原组件内的 `action` 对象。

> 为什么不直接抛出 action 对象？
> 因为事件函数可能会传递参数给 action 对象，因此需要用函数来接受并传递参数。

**`actionCreators.js`**
```
// 不要忘记引入，否则报错
import { INPUT_CHANGE_VALUE, ADD_ITEM, DELETE_ITEM } from './actionTypes'

// 抛出方法，返回 action 对象
export const inputChangeValueAction = (value) => ({
    type: INPUT_CHANGE_VALUE,
    value,
})

export const addItemAction = () => ({
    type: ADD_ITEM,
})

export const deleteItemAction = (index) => ({
    type: DELETE_ITEM,
    index,
})
```

**`TodoList.js`**
```
import React, { Component } from 'react';
import "antd/dist/antd.css";
import { Input, Button, List } from "antd";
import store from './store/index';
// - import { INPUT_CHANGE_VALUE, ADD_ITEM, DELETE_ITEM } from './actionTypes'
// 引入 actionCreators.js，js 后缀可省略
import { inputChangeValueAction, addItemAction, deleteItemAction } from "./store/actionCreators"


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
        // - {type: INPUT_CHANGE_VALUE, value: e.target.value}
        // 用方法返回 action 对象来代替直接在组件内定义 action 对象，实现 action 代码分离
        // 若事件函数存在参数传递，则同样需要在 actionCreators 方法中定义形参
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
                <div style={{ margin: "10px" }}>
                    <Input
                        placeholder={this.state.inputValue}
                        style={{ width: "250px", marginRight: "10px" }}
                        onChange={this.inputChangeValue}
                    />
                    <Button
                        type="primary"
                        onClick={this.addItem}
                    >添加</Button>
                </div>
                <div style={{ width: "325px", margin: "10px" }}>
                    <List
                        size="small"
                        bordered
                        dataSource={this.state.list}
                        renderItem={(item, index) => { return (<List.Item onClick={() => { this.deleteItem(index) }}>{item}</List.Item>) }}
                    />
                </div>
            </div>
        );
    }
}

export default TodoList;
```