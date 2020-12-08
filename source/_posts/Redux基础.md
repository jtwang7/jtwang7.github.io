---
title: Redux 基础及实战
date: 2020-12-1 18:54
tags: [Redux, React]
categories: [React]
excerpt: Redux是一个用来管理管理数据状态和UI状态的JavaScript应用工具。在当前页面复杂度越来越高的互联时代，React组件的状态管理以及状态提升给管理带来了极大的难度，而Redux就是降低管理难度的。但注意的一点是，Redux除了支持React之外，还能支持Angular、jQuery甚至纯JavaScript。本专题将结合博主“技术胖”的Redux免费教程内容，对自身学习做一个总体的总结，整个专题的编写过程将按照教程内容进行编排，主要针对实战代码的重点做主要总结。
---

# Redux
视频教程：[技术胖 Redux 免费教程](https://www.bilibili.com/video/BV1w441137ss?p=1)
参考链接：[技术胖 Redux 教程笔记汇总](https://jspang.com/detailed?id=48#toc230)
**本文搭配原文教程食用，风味更佳~!**

---

## Redux 简要介绍
* **什么是Redux？**
官方解释：Redux is a predictable state container for JavaScript apps.  === Redux是**js**应用的一种可预测的**状态容器**。
通俗理解：Redux是一个用来管理管理数据状态和UI状态的JavaScript应用工具。
* **Redux作用？**
简化组件的状态传递。如下图是不使用Redux和使用Redux时，父子组件之间的通信方式。没有使用Redux的情况，如果两个组件(非父子关系)之间需要通信的话，可能需要多个中间组件为他们进行消息传递，这样既浪费了资源，代码也会比较复杂。Redux中提出了单一数据源 **Store** 用来**存储状态数据**，所有的组建都可以**通过Action修改Store**，也可以**从Store中获取最新状态**，从而实现统一状态管理，完美解决组建之间的通信问题。
![](/img/posts_img/20201201191412481_5749.png)

## Redux 工作流程(总体框架)
我们将结合 Redux 官方给出的结构图去总体了解 Redux 的工作流程，对结构图的理解将直接反映使用者对 Redux 的掌握程度，十分重要！！！
后续实战代码编写中，我们需要按照该结构流程去实现，所以该章节需要重点关注，初学不理解没关系，不过要不断回顾和反复阅读，同时将理解结合到代码中去。
现在给出官方的 Redux 结构，如下：
![](/img/posts_img/20201201192118706_7818.png)
直接看专业的 Redux 工作流程不是特别好理解，我们可以通过借书的例子来理解：
![](/img/posts_img/20201201192506882_23071.png)
React 的各个子组件更新或者改变状态的过程，等同于读者向图书馆借书的过程。读者(Component)需要从图书馆(Store)借书(state)，就要通过图书管理员去取书，取书的过程就是action，但是图书馆(Store)只负责存放图书(state)，无法告诉图书管理员具体的图书在哪，所以又需要图书管理软件(Reducer)进行精确检索，给它传递书名(state)和取书指令(action)，它将返回该书的具体位置(newState)，即一个新的状态，最终读者将从图书馆借到这本书。
当然上述只是通俗的理解，一些步骤可能存在纰漏，我们后续将通过实战代码进行详细解读。

## 代码实战：TodoLisT 实现
### 初始化项目
方法一：
```
Win+R
cmd
npm install -g create-react app //安装脚手架工具，若以前安装过可跳过该步

进入相应目录
mkdir xxx //创建xxx文件夹
cd ReduxDemo      //进入文件夹
create-react-app yyy  //用脚手架创建React项目，名称自定义，此处为yyy
cd yyy   //等项目创建完成后，进入项目目录
npm start  //预览项目
```

方法二：
```
进入vscode终端：Ctrl+`
进入目标目录
npx create-react-app yyy
cd yyy
npm start
```

项目搭建完成后，删除一下没用的文件，让代码结构保持最小化。删除`src`里边的所有文件，只留一个`index.js`,并且`index.js`文件里一些无关内容也都清空，只保留两项引用及`ReactDOM.render()`中的内容。

### 快速生成组件代码结构
编写`index.js`基础文件，代码如下，主要包含了 React, ReactDOM，同时引入 TodoList 组件。
```
import React from 'react';
import ReactDOM from 'react-dom';

// 引入组件，组件首字母大写
import TodoList from './TodoList' 

ReactDOM.render(
  <React.StrictMode>
    <TodoList /> 
  </React.StrictMode>,
  document.getElementById('root')
);
```

可以利用 VSCode 中的 Simple React Snippets 插件快速生成 React Component 的代码结构，先输入`imrc`,再输入`ccc`。
```
// imrc
import React, { Component } from 'react';

// ccc
class TodoList extends Component {
    render() { 
        return ( 
            <div>Hello World</div>
         );
    }
}
export default TodoList;
```

### AntDesign 优化 UI 界面
1. 安装 antd
```
npm install antd --save
或
yarn add antd
```
2. 在使用Ant Design时，先引入CSS样式，有样式文件才可以让UI组件显示正常。具体可从Ant Design官方文档查看快速上手步骤。
```
import "antd/dist/antd.css"
```
3. antd组件使用参照[AntDesign 组件库](https://ant.design/components/overview-cn/)文档，同时在使用组件前，需要从相应的文件引入：
```
import {Input, Button, List} from "antd" //引入了 Input ,Button, List 组件
```

> Input 等组件的 style 等属性设置及一些细节参考最后贴出的实战代码，此处不做细讲

**TodoList 代码** (此部分是第二遍实现，与首遍代码相比存在出入，功能相同，若要参考具体细节，请移步最后的首版代码)
```
import React, { Component } from 'react';
import "antd/dist/antd.css";
import { Input, Button, List } from "antd";

const data = [
    "第一天",
    "第二天",
    "第三天",
]

class TodoList extends Component {
    constructor(props) {
        super(props);
        this.state = {}
    }
    render() {
        return (
            <div>
                <div style={{margin:"10px"}}>
                    <Input placeholder="Write Something" style={{width:"250px",marginRight:"10px"}} />
                    <Button type="primary" >添加</Button>
                </div>
                <div style={{width:"325px",margin:"10px"}}>
                    <List 
                        size = "small"
                        bordered
                        dataSource = {data}
                        renderItem = {(item) => { return (<List.Item>{item}</List.Item>) }}
                    />
                </div>
            </div>
        );
    }
}

export default TodoList;
```

### Redux 工作流程编写
#### 创建 Store 仓库
1. 项目根目录安装 Redux `npm install redux --save`
2. `src`目录下新建`store`子文件夹，在`store`下创建`index.js`，`index.js`就是整个项目的`store`文件，打开文件，编写下面的代码。**注意几点：** 1. 整个项目只能存在一个 `store` (后续会讲到) 2. 要引入 `createStore()` 方法 3.要将 `store` 暴露出去
```
import { createStore } from 'redux'  // 引入createStore方法
const store = createStore()          // 创建数据存储仓库
export default store                 //暴露出去
```

#### “招募”管理者 Reducer
1. 在`store`文件夹下，新建一个文件`reducer.js`
2. 在`reducer.js`中编写如下代码，**注意几点：** 1. Reducer 中要定义 `defaultState` 存储初始(默认)状态，变量名自定义，但常用 `defaultState` 2.抛出一个方法函数，函数返回**新的状态(newState)**，常用箭头函数。 3.`state=defaultState, action`注意书写顺序，state 写在 action 前，否则出错，**原因暂时不知**。
```
const defaultState = {}
export default (state=defaultState, action) => {
    return state
}
```
3. 将 `reducer` 引入到 `Store` 中，创建`Store`时，以参数的形式传递给`Store`。可以理解为在创建仓库Store的时候就连带着招募一个管理员Reducer。
```
// - const store = createStore()
// + import reducer from './reducer'
// + const store = createStore(reducer)
import {createStore} from "redux"
import reducer from './reducer'
const store = createStore(reducer)
export default store;
```

#### 向仓库“存放数据”
我们创建的仓库是空的，我们可以向仓库中添加一些默认的数据，将其存放在 `defaultState` 中。
```
const defaultState = {
    inputValue: "Write Something",
    list: [
        "第一天",
        "第二天",
        "第三天",
    ]
}
```

#### 组件从仓库获取数据
我们仓库中已经存有数据了，我们需要在组件内通过`store.getState()`方法将其获取出来。
接下来，我们要通过store获取的方法，替换掉原有组件的一些值。**注意：** 要从`"./store/index"` 文件引入 `store`
```
import React, { Component } from 'react';
import "antd/dist/antd.css";
import { Input, Button, List } from "antd";
// + import store from './store/index';
import store from './store/index';

// - const data = [...]

class TodoList extends Component {
    constructor(props) {
        super(props);
        // - this.state = {}
        this.state = store.getState()
    }
    render() {
        return (
            <div>
                <div style={{margin:"10px"}}>
                    {- /* <Input placeholder="Write Something" style={{width:"250px",marginRight:"10px"}} /> */}
                    <Input placeholder={this.state.inputValue} style={{width:"250px",marginRight:"10px"}} />
                    <Button type="primary" >添加</Button>
                </div>
                <div style={{width:"325px",margin:"10px"}}>
                    <List 
                        size = "small"
                        bordered
                        // - dataSource = {data}
                        dataSource = {this.state.list}
                        renderItem = {(item) => { return (<List.Item>{item}</List.Item>) }}
                    />
                </div>
            </div>
        );
    }
}

export default TodoList;
```

#### 添加 `<Input />`事件
到目前为止，我们只构建了Redux最基本的结构，还没有真正体会到Redux在管理state时的工作过程，同时我们只在开头定义过`action`，并没有实际用到它，现在我们给`<Input />`添加`onChange`事件，去看看Redux是怎么通过`action`改变仓库状态的。
```
TodoList.js
import React, { Component } from 'react';
import "antd/dist/antd.css";
import { Input, Button, List } from "antd";
import store from './store/index';

class TodoList extends Component {
    constructor(props) {
        super(props);
        //绑定函数，指向this
        this.inputChangeValue = this.inputChangeValue.bind(this)
        this.state = store.getState()
    }

    //定义函数
    inputChangeValue(e) {
        //定义action对象
        const action = {
            type: "input_change_value",
            value: e.target.value,
        }
        //将 action 对象 dispatch 到仓库上
        store.dispatch(action)
    }

    render() {
        return (
            <div>
                <div style={{margin:"10px"}}>
                    <Input placeholder={this.state.inputValue} style={{width:"250px",marginRight:"10px"}} 
                        //添加事件
                        onChange={this.inputChangeValue}
                    />
                    <Button type="primary" >添加</Button>
                </div>
                <div style={{width:"325px",margin:"10px"}}>
                    <List 
                        size = "small"
                        bordered
                        dataSource = {this.state.list}
                        renderItem = {(item) => { return (<List.Item>{item}</List.Item>) }}
                    />
                </div>
            </div>
        );
    }
}

export default TodoList;

-------------------------
reducer.js
const defaultState = {
    inputValue: "Write Something",
    list: [
        "第一天",
        "第二天",
        "第三天",
    ]
}

export default (state = defaultState, action) => {
    //添加对应的action
    if (action.type === "input_change_value") {
        let newState = JSON.parse(JSON.stringify(state))
        newState.inputValue = action.value
        return newState
    }

    return state
}
```
**注意以下几点：**
1. 函数要用`.bind(this)`绑定，箭头函数除外。函数绑定最好写在`constructor(props) {}`内，原因在代码分离的时候会提到。(也可以用`this.xxx.bind(this,param1,param2,...)`，但后续不好进行企业级代码分离)
2. 绑定事件的函数内部创建 action 对象，**注意 `action` 是对象!!!**
3. **`action` 必须要有 `type` 属性**，其属性值为自定义名称，用于标明当前声明 action 对象的名称。其余属性名称自定义。
4. 拥有 action 对象后，需要传递给 store 仓库，**首先要将 action `dispatch` 到 store 仓库上，注意此处是`store.dispatch(action)`，而不是`action.dispatch(store)`，dispatch是派遣的意思，可以理解为“仓库派遣一个动作”，而不是“动作派遣仓库”**。
5. 由于 store 的自动推送策略（store只是一个仓库，它并没有管理能力，它会把接收到的action自动转发给Reducer），我们将在 `reducer.js` 中对 `action` 进行处理
6. **!!!重点：** reducer 中只能接收 state ，但不能直接改变 state。因此，我们需要换种方法：定义一个新的state变量，作为临时变量，将 state 通过 `JSON.parse(JSON.stringify(state))` 进行深度拷贝，通过重新赋值改变临时变量，再将新的状态抛出

#### 用newState更新组件
经过上述操作后，`Reducer` 通过 `action` 修改了原始 state 并抛出了新的状态 newState, 但是 newState 还没有被我们的组件给利用，现在就将这最后一环补上，实现整个 Redux 的工作流程。
```
import React, { Component } from 'react';
import "antd/dist/antd.css";
import { Input, Button, List } from "antd";
import store from './store/index';

class TodoList extends Component {
    constructor(props) {
        super(props);
        this.inputChangeValue = this.inputChangeValue.bind(this)
        this.state = store.getState()
        //绑定storeChange函数指向this；仓库订阅storeChange函数
        this.storeChange = this.storeChange.bind(this)
        store.subscribe(this.storeChange)
    }

    inputChangeValue(e) {
        const action = {
            type: "input_change_value",
            value: e.target.value,
        }
        store.dispatch(action)
    }
    
    //从仓库中获取新的状态，重新渲染组件
    storeChange() {
        this.setState(store.getState())
    }

    render() {
        return (
            <div>
                <div style={{margin:"10px"}}>
                    <Input placeholder={this.state.inputValue} style={{width:"250px",marginRight:"10px"}} 
                        onChange={this.inputChangeValue}
                    />
                    <Button type="primary" >添加</Button>
                </div>
                <div style={{width:"325px",margin:"10px"}}>
                    <List 
                        size = "small"
                        bordered
                        dataSource = {this.state.list}
                        renderItem = {(item) => { return (<List.Item>{item}</List.Item>) }}
                    />
                </div>
            </div>
        );
    }
}

export default TodoList;
```
**注意以下几点：**
1. `this.setState()`接收的对象是`store.getState()`，即仓库中的状态而非 this.state。
2. 仓库需要订阅`storeChange()`方法，不订阅程序仍能正常运行，但是在某些地方会出错，为了避免这种情况，请将订阅写在 constructor 中。


---
以上就是 Redux 基础的一些操作流程，后续的 Button onClick 事件，List onClick 事件不再详细讲解，感兴趣的同学可在下方整体的代码中找到详细的注释。以上就是我对Redux学习的一些体会，希望对各位有所启示。
### TodoList Redux 实现及详细注释
#### "./src/index.js"
```
import React from 'react';
import ReactDOM from 'react-dom';

// 引入组件，组件首字母大写
import TodoList from './TodoList' 

ReactDOM.render(
  <React.StrictMode>
    <TodoList /> 
  </React.StrictMode>,
  document.getElementById('root')
);
```

#### "./src/TodoList.js"
```
// imrc
import React, { Component } from 'react';
// npm install antd --save ; import css file
import "antd/dist/antd.css"
// 引入CSS样式后，还需引入用到的组件
import { Input, Button, List } from "antd"
// 引入 store，绑定 component 和 store
import store from "./store/index"

// 将默认 state 存入 reducer -- defaultState 中
// const data = [
//     "第一条数据",
//     "第二条数据",
//     "第三条数据",
// ]

// ccc
class TodoList extends Component {
    constructor(props) {
        super(props);
        // store.getState() 方法从 store 仓库获取 state，固定方法
        this.state = store.getState()

        // storeChange 函数绑定 this
        this.storeChange = this.storeChange.bind(this)
        // store 订阅 redux 状态，状态更新则重新渲染
        store.subscribe(this.storeChange)
    }

    // 定义 onChange 函数 changeInputValue
    // 将最新输入的状态通过 action 传递给 store
    // action 是一个对象，需要声明。
    // action: type 属性是必需的，type 属性值自定义，表示该 action 的名字; 其余属性自定义，例如 value
    // e.target.value 获取事件 event 对象的值
    // store.dispatch(action) 是必需的，将 action 绑定到 store 上，参考 redux 官方文档，这是默认的结构
    // store 只是仓库，没有管理能力，根据 store 的自动推送策略，store 会将接收的 action 自动转发给 reducer
    changeInputValue(e) {
        const action = {
            type: "change_input_value",
            value: e.target.value,
        }
        store.dispatch(action)
    }

    valueSubmit() {
        const action = {
            type: "value_submit",
        }
        store.dispatch(action)
    }

    deleteItem(idx) {
        const action = {
            type: "delete_item",
            index: idx,
        }
        store.dispatch(action)
    }

    // reducer action 只更新了 store 状态，需要定义一个 storeChange 函数来更新组件
    // 通过 store.getState() 获取仓库状态，后用 setState 方法重新渲染
    storeChange() {
        this.setState(store.getState())
    }


    render() {
        return (
            <div>
                <div style={{ margin: "10px" }}>
                    {/* 
                    1.React - style 接收css样式对象 
                    2.width 改变宽度；height 改变高度；margin 改变外边距；marginRight 设置右边距
                    3.
                    - placeholder="Please input something ..."
                    + placeholder={this.state.inputValue}
                    4.添加 onChange 事件，监听 input 文本框内容变化,不要忘记 .bind
                    + onChange={this.changeInputValue.bind(this)
                    */}
                    <Input
                        placeholder={this.state.inputValue}
                        style={{ width: "250px", marginRight: "10px" }}
                        onChange={this.changeInputValue.bind(this)}
                    />
                    {/* 
                    1.开头要引入 antd 的 Button 组件
                    2.type 属性见 antd 的 Button 组件文档说明
                    3.
                    +onClick={this.valueSubmit.bind(this)}
                    */}
                    <Button
                        type="primary"
                        onClick={this.valueSubmit.bind(this)}
                    >输入</Button>
                </div>
                <div style={{ margin: "10px", width: "325px" }}>
                    {/* 
                    1.参照 antd 的 List 组件文档
                    2.dataSource 接收一个数组，数组包含List Item
                    3.renderItem 渲染 item，固定写法
                    4.(item)=>(...)接收dataSource数组中的元素
                    5.<List />理解为外框架，则<List.Item></List.Item>理解为框架内的一行，
                    <List.Item></List.Item>中可以添加其他JSX元素
                    6.标签内属性值间用空格或者换行分割
                    */}
                    <List
                        size="small"
                        bordered
                        // 用 this.state.list 替换 const data
                        // dataSource = {data}
                        dataSource={this.state.list}
                        // -(item) => (<List.Item>{item}</List.Item>)
                        // +(item,idx) => (<List.Item onClick={this.deleteItem.bind(this,idx)}>{item}</List.Item>)
                        // 此处箭头函数形式()=>() ，也可写为()=>{return ()}，用{}时要注意必须要用 return 将结果抛出，详情查看 ES6 箭头函数
                        renderItem={(item,idx) => (<List.Item onClick={this.deleteItem.bind(this,idx)}>{item}</List.Item>)}
                    />
                </div>
            </div>
        );
    }
}

export default TodoList;
```

#### "./src/store/index.js"
```
// npm install --save redux
// 创建 redux store 仓库：创建 store 文件夹；创建 index.js 文件

// 引入 createStore 方法，顾名思义--创建仓库
import {createStore} from 'redux'
// 将 reducer 引入 store 中，实现 reducer 到 store 的绑定(见 redux 官方结构图)
import reducer from './reducer'

// 调用 createStore 方法创建 store 仓库
// reducer 以参数形式传递给 createStore()，可以理解为在搭建仓库的时候就招募了一个管理员
const store = createStore(reducer)

// 将仓库暴露出去
export default store
```

#### "./src/store/reducer.js"
```
//仓库需要一个管理员 reducer 进行管理

// 定义默认state
const defaultState = {
    inputValue: "Please input something ...",
    list: [
            "第一条数据",
            "第二条数据",
            "第三条数据",
        ],
}

// 暴露 reducer 的管理行为：接收 state，根据 action 动作返回新的 state
// state=defaultState 写在 action 之前，否则传出的数据不正确，why?
// state 参数：store 仓库内的原始状态
// action 参数：action 新传递的状态
// store 只是仓库，没有管理能力，根据 store 的自动推送策略，store 会将接收的 action 自动转发给 reducer
export default (state=defaultState,action) => {
    // 在 reducer 中，我们获得了原始状态 state 和新传递的状态 action，下一步就是改变 store 内的 state
    //console.log(state,action)

    /*
    用 if 语句识别当前的 action 对象，
    因为每个事件都定义一个 action ，则需要通过 action.type 区分，
    这也就解释了为什么 type 属性在 action 中是必需的。
    */
    if(action.type === "change_input_value") {
        /*
        !!!重点
        reducer 中只能接收 state ，但不能直接改变 state
        因此，我们需要换种方法：
        定义一个新的state变量，作为临时变量，将 state 通过 JSON.parse(JSON.stringify(state)) 进行深度拷贝
        通过重新赋值改变临时变量，再将新的状态抛出
        */
        let newState = JSON.parse(JSON.stringify(state))
        newState.inputValue = action.value
        return newState
    }

    if(action.type === "value_submit") {
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.push(newState.inputValue)
        return newState
    }

    if(action.type === "delete_item") {
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.splice(action.index,1)
        return newState
    }

    return state
}
```


---
## React 三个易错点
1. store必须是**唯一**的，多个store是坚决不允许，只能有一个store空间
2. 只有store能改变自己的内容，Reducer不能改变
3. Reducer必须是**纯函数**

### Store 仓库唯一
```
import {createStore} from "redux"
import reducer from './reducer'
const store = createStore(reducer)
export default store;
```
我们在 `/store/index.js` 文件中，用createStore()方法，声明了一个store。之后整个应用都在使用这个 store，并且只能创建和使用这一个 store ，否则会报错。

### 改变 state 的是 Store ，而不是 Reducer
我们在写 redux 时将业务逻辑代码都写在了 Reducer 中，但这并不意味着组建的 state 状态是在 Reducer 中改变的。事实上 Reducer 中只能接收 state ，而不允许改变 state。
```
export default (state = defaultState, action) => {
    if (action.type === INPUT_CHANGE_VALUE) {
        let newState = JSON.parse(JSON.stringify(state))
        newState.inputValue = action.value
        return newState
    }
    return state
}
```
如上述代码示例，在 Reducer 中我们声明了一个新的临时状态变量`newState`来深度拷贝当前状态，最终只是将新的状态作了一个返回，返回到了store中，并没有作任何改变。Reudcer 只是返回了更改的数据，但是并没有更改 store 中的数据，store 拿到了 Reducer 的数据，自己对自己进行了更新。

> 关于 Store 和 Reducer 中的 state 关联，引用一个网友的评论：
> redux内的reducer里面的state初始时是一个默认赋值，当store里有state数据时，每次传入当前的currentstate给reducer内的state，所以reducer每次拷贝的是当前的state状态，并不是defaultvalue，reducer 经过 action 后返回更改的 newState 状态到 Store 中，而 Store 会将这些状态改动更新到仓库的状态中。

### Reducer 必须是纯函数
**纯函数：** 如果函数的调用参数相同，则永远返回相同的结果。**返回结果**不依赖于程序执行期间函数外部任何状态或数据的变化，必须**只依赖于其输入参数**。
简单理解，若1. 某一函数的返回结果是由传入的值(形参)决定的，而不是其它的东西决定的，2. 函数不产生副作用(不影响外部变量)，那么该函数就是纯函数。

* 纯函数
```
const aaa = (x, y) => {
    let m = 1;
    return x + y + m //函数返回结果由函数参数决定
}
```

* 非纯函数
```
const xxx = (x, y) => {
    let m = new Date() //或 ajax,axios 异步请求等
    return x + y + m //函数结果受外部影响，结果不能由参数决定
}
```

Reducer 中返回状态只由 state 和 action 决定。
```
export default (state = defaultState, action) => {
    if (action.type === INPUT_CHANGE_VALUE) {
        let newState = JSON.parse(JSON.stringify(state))
        newState.inputValue = action.value
        return newState
    }
    return state
}
```