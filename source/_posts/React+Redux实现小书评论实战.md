---
title: React+Redux实现小书评论实战(一~三)
date: 2020-12-04 14:53
tags: [Redux, React]
categories: [React]
excerpt: React小书中已经实现了评论功能，但是对于多级的状态管理，用Redux更加方便，为了学习和体会Redux的状态管理优势，在经过一段时间的Redux学习后，尝试动手实现评论功能，并通过实战对项目进行优化，包括代码分离，界面和业务逻辑独立等优化。
---

# React+Redux实现小书评论实战(一~三)
个人github: [个人github](https://github.com/jtwang7)
github地址：[项目](https://github.com/jtwang7/reduxCommentApp.git)

---
## 项目结构树
```
React小书评论功能(一~三)
├─ README.md
└─ src
   ├─ CommentApp.js
   ├─ components
   │  ├─ comment.css
   │  ├─ Comment.js
   │  ├─ CommentInput.js
   │  └─ CommentList.js
   ├─ componentsUI
   │  └─ CommentInputUI.js
   ├─ index.js
   └─ store
      ├─ actionCreators.js
      ├─ actionTypes.js
      ├─ index.js
      └─ reducer.js
```

用 Redux 状态管理实现了 React 小书的评论功能(一~三)，同时基于企业级项目开发要求进行了整体优化，包括 actionTypes 和 actionCreator ，以及 UI 界面与业务逻辑分离。项目结构如上述所示，项目整体样式未经过调整，主要采用了阿里开源的 Ant Design 作为主要样式。

## 项目文件
### src/CommentApp.js
```
import React, { Component } from 'react';
import CommentInput from "./components/CommentInput"
import CommentList from './components/CommentList';
// 整体框架
class CommentApp extends Component {
    constructor(props) {
        super(props);
        this.state = {}
    }
    render() {
        return (
            <div>
                // 二级组件
                <CommentInput />
                <CommentList />
            </div>
        );
    }
}
export default CommentApp;
```

### CommentInput
#### src/components/CommentInput.js
CommentInput 业务逻辑
```
import React, { Component } from 'react';
import store from "../store/index"
import { inputChangeAction, textAreaChangeAction, commentSubmitAction } from "../store/actionCreators"
import { CommentInputUI } from "../componentsUI/CommentInputUI"

class CommentInput extends Component {
    constructor(props) {
        super(props);
        this.state = store.getState()
        // store订阅
        this.storeChange = this.storeChange.bind(this)
        store.subscribe(this.storeChange)
        // 事件方法绑定
        this.inputChange = this.inputChange.bind(this)
        this.textAreaChange = this.textAreaChange.bind(this)
        this.commentSubmit = this.commentSubmit.bind(this)
    }
    // store仓库内容改变后，重新渲染组件，与subscribe一同使用
    storeChange() {
        this.setState(store.getState())
    }
    // 方法内声明action,并注入到store仓库
    inputChange(e) {
        const action = inputChangeAction(e.target.value)
        store.dispatch(action)
    }
    textAreaChange(e) {
        const action = textAreaChangeAction(e.target.value)
        store.dispatch(action)
    }
    commentSubmit() {
        const action = commentSubmitAction()
        store.dispatch(action)
    }
    render() {
        return (
            <div>
                <CommentInputUI
                    inputChange={this.inputChange}
                    textAreaChange={this.textAreaChange}
                    commentSubmit={this.commentSubmit}
                />
            </div>
        )
    }
}
export default CommentInput;
```

#### src/componentsUI/CommentInputUI.js
CommentInput UI界面 + 无状态组件
```
import "antd/dist/antd.css"
import { Input, Button } from 'antd'
import "../components/comment.css"

const { TextArea } = Input;
export const CommentInputUI = (props) => {
    return (
        <div>
            <div className="inputRow">
                <span>用户名:</span>
                <Input
                    style={{ width: "300px", marginLeft:"20px" }}
                    placeholder="Please enter your name"
                    onChange={props.inputChange}
                />
            </div>
            <div className="inputRow secondRow">
                <span>评论内容:</span>
                <TextArea
                    rows={5}
                    style={{ width: "285px", marginLeft:"20px" }}
                    // ref={(textArea) => (this.textArea = textArea)}
                    onChange={props.textAreaChange}
                />
            </div>
            <div>
                <Button
                    type="primary"
                    onClick={props.commentSubmit}
                >发布</Button>
            </div>
        </div>
    );
}
```

### CommentList
#### src/components/CommentList.js
```
import React, { Component } from 'react';
import store from "../store/index"
import Comment from "./Comment"

class CommentList extends Component {
    constructor(props) {
        super(props);
        this.state = store.getState()
        // CommentList要渲染到页面，所以要订阅storeChange方法
        this.storeChange = this.storeChange.bind(this)
        store.subscribe(this.storeChange)
    }
    storeChange() {
        this.setState(store.getState())
    }
    render() {
        return (
            <div>
                <div>
                    {this.state.list.map(
                        (item, index) => (<Comment content={item} index={index} key={index} />)
                    )}
                </div>
            </div>
        );
    }
}
export default CommentList;
```

#### src/components/Comment.js
```
import React, { Component } from 'react';
import "antd/dist/antd.css"
import { Button } from 'antd'
import store from '../store';
import { deleteCommentAction } from "../store/actionCreators"

class Comment extends Component {
    constructor(props) {
        super(props);
        this.deleteComment = this.deleteComment.bind(this);
    }
    deleteComment(index) {
        const action = deleteCommentAction(index)
        store.dispatch(action)
    }
    render() {
        return (
            <div>
                <div>
                    {this.props.content}
                    <Button
                        type="link"
                        onClick={() => (this.deleteComment(this.props.index))}
                    >删除</Button>
                </div>
            </div>
        );
    }
}
export default Comment;
```

### Store
#### src/store/actionTypes.js
action.type 常量单独分离
```
export const INPUT_CHANGE = "input_change"
export const TEXT_AREA_CHANGE = "text_area_change"
export const COMMENT_SUBMIT = "comment_submit"
export const DELETE_COMMENT = "delete_comment"

```

#### src/store/actionCreators.js
action 对象单独分离管理，用函数方式调用
```
import { INPUT_CHANGE, TEXT_AREA_CHANGE, COMMENT_SUBMIT, DELETE_COMMENT } from "./actionTypes"
export const inputChangeAction = (value) => ({
    type: INPUT_CHANGE,
    value,
})
export const textAreaChangeAction = (value) => ({
    type: TEXT_AREA_CHANGE,
    value,
})
export const commentSubmitAction = () => ({
    type: COMMENT_SUBMIT,
})
export const deleteCommentAction = (index) => ({
    type: DELETE_COMMENT,
    index,
})
```

#### src/store/index.js
store仓库创建
```
import {createStore} from 'redux'
import reducer from './reducer'

const store = createStore(reducer)
export default store
```

#### src/store/reducer.js
reducer 管理，定义 action 操作
```
import { act } from "react-dom/test-utils"

const defaultState = {
    inputValue: "",
    textAreaValue: "",
    list: [],
}

export default (state = defaultState, action) => {
    if (action.type === "input_change") {
        let newState = JSON.parse(JSON.stringify(state))
        newState.inputValue = action.value
        return newState
    }
    if (action.type === "text_area_change") {
        let newState = JSON.parse(JSON.stringify(state))
        newState.textAreaValue = action.value
        return newState
    }
    if (action.type === "comment_submit") {
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.push(`${newState.inputValue}: ${newState.textAreaValue}`)
        return newState
    }
    if (action.type === "delete_comment") {
        let newState = JSON.parse(JSON.stringify(state))
        newState.list.splice(action.index,1)
        return newState
    }
    return state
}
```
