---
title: React Portal
date: 2020-12-08 11:14
tags: [React, React-Portal]
categories: [React]
excerpt: React Portal ——“传送门”，它给React提供了一个强大的功能，原始的子组件只能在父组件的代码内部调用位置进行渲染，这就会影响到整个父组件的排版，特别是一些类似于弹窗的效果，会挤占父组件的渲染空间，没有 Portal，我们就只能预留空间给弹窗组件，防止布局的混乱。而 Portal 的提出，让子组件能够跳出父组件，在其他任意挂载节点进行渲染，提高了代码的灵活性。本章内容主要转载于知乎专栏的文章，用于记录和保留，具体的一些文章内容可以通过文章链接去原文查看，后续将原创一些实战的学习过程。
---

# React Portal
参考链接：[传送门：React Portal](https://zhuanlan.zhihu.com/p/29880992)
官方网站：[Portals-React](https://reactjs.org/docs/portals.html#___gatsby)

---

## Portal -- 传送门
为什么React需要传送门？
React Portal之所以叫Portal，因为做的就是和“传送门”一样的事情：**render到一个组件里面去，实际改变的是网页上另一处的DOM结构。**
在React中，一切皆为组件，用组件可以表示一切界面中发生的逻辑，不过，有些特例处理起来还比较麻烦，比如，某个组件在渲染时，在某种条件下需要显示一个对话框(Dialog)，这该怎么做呢？
最直观的做法，就是直接在JSX中把Dialog画出来，像下面代码的样子。
```
<div class="foo">
   <div> ... </div>
   { needDialog ? <Dialog /> : null }
</div>
```
问题是，我们写一个Dialog组件，就这么渲染的话，Dialog最终渲染产生的HTML就存在于上面JSX产生的HTML一起了，类似下面这样。
```
<div class="foo">
   <div> ... </div>
   <div class="dialog">Dialog Content</div>
</div>
```
可是问题来了，对话框应该是一个独立的组件，通常应该显示在屏幕的最中间，现在Dialog被包在其他组件中，要用CSS的position属性控制Dialog位置，就要求从Dialog往上一直到body没有其他postion是relative的元素干扰，这有点难为作为通用组件的Dialog，毕竟，谁管得住所有组件不用position呢。还有一点，Dialog的样式，因为包在其他元素中，各种样式纠缠，CSS样式太容易搞成一坨浆糊了。
因此，React 就引入了 Portal 传送门的概念。

## React v16 之前的传送门实现方法
为什么要讲旧版本的实现方法呢？因为旧版本更能体现传送门实现的一个思想，而新版本更多的是一个封装和便于使用，理解了旧版本就可以更好地使用新版本 Portal 了。
在v16之前，实现“传送门”，要用到两个秘而不宣的React API
```
unstable_renderSubtreeIntoContainer
unmountComponentAtNode
```

* 第一个unstable_renderSubtreeIntoContainer。这个API的作用就是建立“传送门”，可以把JSX代表的组件结构塞到传送门里面去，让他们在传送门的另一端渲染出来。
* 第二个unmountComponentAtNode用来清理第一个API的副作用，通常在unmount的时候调用，不调用的话会造成资源泄露的。

一个通用的Dialog组件的实现差不多是这样，注意看renderPortal中的注释。
```
import React, {Component} from 'react';
import {unstable_renderSubtreeIntoContainer, unmountComponentAtNode} from 'react-dom';

class Dialog extends Component {
  render() {
    return null;
  }

  componentDidMount() {
    const doc = window.document;
    this.node = doc.createElement('div');
    doc.body.appendChild(this.node);

    this.renderPortal(this.props);
  }

  componentDidUpdate() {
    this.renderPortal(this.props);
  }

  componentWillUnmount() {
    unmountComponentAtNode(this.node);
    window.document.body.removeChild(this.node);
  }

  renderPortal(props) {
    unstable_renderSubtreeIntoContainer(
      this, //代表当前组件
      <div class="dialog">
        {props.children} //也可以是其他自定义JSX结构
      </div>, // 塞进传送门的JSX
      this.node // 传送门另一端的DOM node
    );
  }
}
```

1. 首先，**`render`函数不要返回有意义的`JSX`(即返回`null`)**，也就说说这个组件通过正常生命周期什么都不画，要是画了，那画出来的HTML/DOM就直接出现在使用Dialog的位置了，这不是我们想要的。
2. 在**`componentDidMount`**里面，**利用原生API来在body上创建一个div**，这个div的样式绝对不会被其他元素的样式干扰。
3. 然后，无论`componentDidMount`还是`componentDidUpdate`，都**调用一个`renderPortal`来往“传送门”里塞东西**。
4. 在`renderPortal`中，利用`unstable_renderSubtreeIntoContainer`函数往直前创建的`div`里塞`JSX`，这里我们用的`JSX`是这样。
```
<div class="dialog">
   {props.children}
</div>
//--------------------
//调用 Dialog 组件时，可以加上任意的子组件。
<Dialog>
  What ever shit
  <div>Hello</div>
  <p>World</p>
</Dialog>
```

总结，这个Dialog组件做得事情是这样：

* 它什么都不给自己画，render返回一个null就够了；
* 它做得事情是通过调用renderPortal把要画的东西画在DOM树上另一个角落。

## React v16 Portal
正因为 Portal 的强大能力，React v16 开始正式支持 Portal。
在v16中，使用Portal创建Dialog组件简单多了，不需要牵扯到componentDidMount、componentDidUpdate，也不用调用API清理Portal，关键代码在render中，像下面这样就行。
```
import React from 'react';
import {createPortal} from 'react-dom';

class Dialog extends React.Component {
  constructor() {
    super(...arguments);

    const doc = window.document;
    this.node = doc.createElement('div');
    doc.body.appendChild(this.node);
  }

  componentWillUnmount() {
    window.document.body.removeChild(this.node);
  }

  render() {
    return createPortal(
      <div class="dialog">
        {this.props.children}
      </div>, //塞进传送门的JSX
      this.node //传送门的另一端DOM node
    );
  }
}
```
整体思想是类似的：
1. 在 `constructor` 中，获取 DOM，用原生 API 创建节点。
2. 将该节点加载到 DOM 文档树的 body 部分。
3. 调用 `createPortal(child,container)` 方法创建新的 JSX 元素(即构造新组件)。**`createPortal`方法接收两个参数，`child`是任何可渲染的React子元素，例如元素，字符串或片段。`container`是将被传送到的目标节点(DOM元素),它会将`child`插入`container`中，并且将`child`传送到`container`元素内的最底部。**
4. 在组件销毁时调用`componentWillUnmout`将 body 中的节点`<div></div>`移除。
5. 在写模态框时，用了portal，就不会完全挡死，只需调节z-Index，可以覆盖页面上的任意元素(存疑，video，canvas等这类元素未试过)。


## 事件冒泡
v16之前的React Portal实现方法，有一个小小的缺陷，就是Portal是单向的，内容通过Portal传到另一个出口，在那个出口DOM上发生的事件是不会冒泡传送回进入那一端的。具体详情可以看[官方文档](https://reactjs.org/docs/portals.html#___gatsby)，有详细的说明。

代码如下：
```
<div onClick={onDialogClick}>   
   <Dialog>
     What ever shit
   </Dialog>
</div>
```
在Dialog画出的内容上点击，onDialogClick是不会被触发的。
在v16中，通过Portal渲染出去的DOM，事件是会冒泡从传送门的入口端冒出来的，上面的onDialogClick也就会被调用到了。


## 实战
[React Portal 实现 Modal](https://www.cnblogs.com/demodashi/p/8512647.html)
