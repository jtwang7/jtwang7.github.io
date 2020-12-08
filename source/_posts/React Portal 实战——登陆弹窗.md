---
title: React Portal 实战——登陆弹窗
date: 2020-12-08 13:14
tags: [React, React-Portal]
categories: [React]
excerpt: React Portal ——“传送门”，它给React提供了一个强大的功能，原始的子组件只能在父组件的代码内部调用位置进行渲染，这就会影响到整个父组件的排版，特别是一些类似于弹窗的效果，会挤占父组件的渲染空间，没有 Portal，我们就只能预留空间给弹窗组件，防止布局的混乱。而 Portal 的提出，让子组件能够跳出父组件，在其他任意挂载节点进行渲染，提高了代码的灵活性。本章结合 Portal 的一些学习，动手实现了登陆弹窗的功能组件。
---

# React Portal 实战——登陆弹窗

## 实现效果：
点击初始界面登录按钮，跳转到登录路径，用 Portal 创建一个登录弹窗以及灰色蒙版背景。
要求：

1. 同时保证原始界面内容保留并显示在蒙版后。
2. 点击退出按钮后，重定向到“个人档案”页面。

效果图：
![](/img/posts_img/20201208094720091_21532.png)

## 登陆界面代码
直接解读代码内容
**重点：**

1. 使用 Portal 需要在组件开头导入 `ReactDOM.createPortal()` 方法，即 `import { createPortal } from 'react-dom'`
2. 掌握 Portal 的组织结构：创建DOM节点，添加JSX元素，在组件销毁前移除DOM节点
3. React Router 重定向

```
import React, { Component } from 'react';
import { createPortal } from 'react-dom'
import "../index.css"
import registerPng from '../images/register.png'
//AntDesign调用
import "antd/dist/antd.css"
import { Input, Button } from 'antd'
import { EyeInvisibleOutlined, EyeTwoTone } from '@ant-design/icons';

class Register extends Component {
    constructor(props) {
        super(props);
        this.state = {
            isRender: true,
        }
        this.redirect = this.redirect.bind(this)

        const dom = window.document;
        this.node = dom.createElement("div")
        this.node.setAttribute("id", "account") //this.node.id = "account"
        dom.body.appendChild(this.node)
    }

    redirect() {
        // 多余步骤，跳转时其实已经执行了componentWillUnmount；
        // 此处只做关于组件销毁的常用方法记录；
        this.setState({
            isRender: true,
        })
        this.props.history.push("/person/齐天大圣")
    }

    componentWillUnmount() {
        window.document.body.removeChild(this.node)
    }

    render() {
        return createPortal(
            this.state.isRender ? (
                <div>
                    <div className="beijing">
                        <div className="dengLu">
                            <img
                                src={registerPng}
                                style={{
                                    borderRadius: "15px 15px 0 0",
                                    margin: 0,
                                    padding: 0,
                                }}
                            />
                            <div className="inputContainer">
                                <h3 style={{ marginLeft: "40px" }}>账户</h3>
                                <Input
                                    placeholder="Please enter account..."
                                    className="inputStyle"
                                    style={{
                                        width: "70%",
                                        height: "30px",
                                        marginRight: "40px"
                                    }}
                                />
                            </div>
                            <div className="inputContainer">
                                <h3 style={{ marginLeft: "40px" }}>密码</h3>
                                <Input.Password
                                    placeholder="input password"
                                    iconRender={visible => (visible ? <EyeTwoTone /> : <EyeInvisibleOutlined />)}
                                    style={{
                                        width: "70%",
                                        height: "30px",
                                        marginRight: "40px"
                                    }}
                                />
                            </div>
                            <div className="confirmAction">
                                <Button>确认</Button>
                                <Button onClick={this.redirect}>取消</Button>
                            </div>
                        </div>
                    </div>
                </div>
            ) : null
            , this.node
        )
    }
}

export default Register;
```

### 代码详解
1. React v16 Portal 创建的调用方法
```
import { createPortal } from 'react-dom'
```
2. Portal 主体
constructor()构造函数中创建DOM节点
(给DOM节点添加id属性)
将DOM节点添加到`<body>`中
用`createPortal`方法创建 Portal，传入可渲染子元素以及挂载的DOM节点，并在`render`中返回
在`componentWillUnmount`中，即组件销毁(路由跳转，display:none等都算将组建销毁)前，移除该DOM节点
```
class Register extends Component {
    constructor(props) {
        ...
        //创建dom节点
        const dom = window.document;
        this.node = dom.createElement("div")
        this.node.setAttribute("id", "account") //this.node.id = "account"
        dom.body.appendChild(this.node)
    }

    componentWillUnmount() {
        window.document.body.removeChild(this.node)
    }

    render() {
        return createPortal(
            (
                <div>
                    ...
                </div>
            )
            , this.node
        )
    }
}
export default Register;
```
3. React Router 路由重定向
路由重定向可以实现路由路径的直接跳转，例如进入某一组件，在碰到重定向后自动跳转到另一组件。
路由重定向可以分为：**标签式重定向** 和 **编程式重定向**
详细参考 Router 路由文章，此处用了编程式的重定向方法`this.props.history.push("url")`。
```
redirect() {
    // 多余步骤，跳转时其实已经执行了componentWillUnmount；
    // 此处只做关于组件销毁的常用方法记录；
    this.setState({
        isRender: true,
    })
    this.props.history.push("/person/齐天大圣")
}
```