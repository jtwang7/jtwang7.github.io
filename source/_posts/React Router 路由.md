---
title: React Router 路由
date: 2020-12-07 21:28
tags: [React,react-router-dom]
categories: React
excerpt: 上一次简单介绍了 react-router-dom 的简单实用方法，后续又进一步观看了技术胖的 react-router 教程讲解，并结合当前项目实现了多页面路由跳转的功能，本章主要介绍和记录整个代码编写过程、思路以及编写过程中踩过的一些坑。
---

# React Router 路由

## React Router 开发环境搭建
1. 安装React脚手架工具（若之前安装过了可以省略）
```
npm install -g create-react-app
```
2. 创建项目
```
D:  //进入D盘
mkdir ReactRouterDemo   //创建ReactRouterDemo文件夹
cd ReactRouterDemo      //进入文件夹
create-react-app demo01  //用脚手架创建React项目
cd demo01   //等项目创建完成后，进入项目目录
npm  start  //预览项目，可跳过该步
```
3. 安装 React Router
在终端`Ctrl+丶`的项目目录下：
```
npm install --save react-router-dom
```

## React Router 使用
## 创建多个组件作为路由跳转的目标组件
**组件目录树: 新建`components`文件夹存放组件**
```
demo_router_01
└─ src
  ├─ components
     ├─ Analysis.js
     ├─ Home.js
     └─ Person.js
```
**`Home.js`**
```
import React, { Component } from 'react';
import homePng from "../images/Home.png"
import '../index.css'

class Home extends Component {
    constructor(props) {
        super(props);
        this.state = {}
    }
    render() {
        return (
            <div>
                <div className="homePage">
                    <img className="img" src={homePng} alt="" />
                </div>
            </div>
        );
    }
}

export default Home;
```
**`Analysis.js`**
```
import React, { Component } from 'react';
import analysisPng from "../images/Analysis.png"
import '../index.css'

class Analysis extends Component {
    constructor(props) {
        super(props);
        this.state = {  }
    }
    render() { 
        return ( 
            <div>
                <img className="img" src={analysisPng} alt="" />
            </div>
         );
    }
}
 
export default Analysis;
```
**`Person.js`**
```
import React, { Component } from 'react';
import personPng from "../images/Person.png"
import '../index.css'

class Person extends Component {
    constructor(props) {
        super(props);
        this.state = {  }
    }

    render() { 
        return ( 
            <div>
                <img className="img" src={personPng} alt="" />
            </div>
         );
    }
}

export default Person;
```

## 编写路由组件(React万物皆组件)
**新建`router`文件夹存放路由组件**
```
import React from 'react'
import { BrowserRouter as Router, Route, NavLink } from 'react-router-dom'
import Home from '../components/Home'
import Analysis from '../components/Analysis'
import Person from '../components/Person'
import "../index.css"
import "../images/iconfont.css"

const HomePage = () => {
    const routeList = [
        {
            path: "/",
            exact: true,
            title: "首页",
            component: Home,
            icon: "iconfont icon-yemian-copy"
        },
        {
            path: "/analysis/",
            exact: false,
            title: "相关性分析",
            component: Analysis,
            icon: "iconfont icon-fenxi"
        },
        {
            path: "/person/",
            exact: false,
            title: "个人档案",
            component: Person,
            icon: "iconfont icon-icon-text-fn-documentation"
        },
    ]
    return (
        <div>
            <div>
                <div className="pageTop">
                    <h2 className="pageTitle">社矫人员属性及轨迹数据可视化分析平台</h2>
                </div>
                <div className="titleBottom">
                    <Router>
                        <div className="leftMenu">
                            <ul>
                                {
                                    routeList.map((item, index) => (
                                        <div className="subTitle" key={index}>
                                            <i className={item.icon} key={index}></i>
                                            <li className="textItem">
                                                <NavLink
                                                    exact={item.exact} //NavLink-exact: if true, 精确匹配active
                                                    to={item.path}
                                                    key={index}
                                                    className="link"
                                                    activeClassName="activeLink"
                                                >{item.title}</NavLink>
                                            </li>
                                        </div>
                                    ))
                                }
                            </ul>
                        </div>
                        <div>
                            {
                                routeList.map((item, index) => {
                                    return (
                                        <Route
                                            exact={item.exact}
                                            path={item.path}
                                            key={index}
                                            component={item.component}
                                        ></Route>
                                    )
                                })
                            }
                        </div>
                    </Router>
                </div>
            </div>
        </div>
    )
}

export default HomePage
```
**重点：**
1. 上述路由组件中没有状态的变化，因此采用了无状态组件的方式书写。**注意引入`React`，凡是用到`JSX`语法的都要`import React from 'react'`**
```
import React from 'react'
const xxx = () => {
    return (
        <div>...</div>
    )
}
export default xxx
```
2. 开头引入`react-router-dom`，否则无法使用路由标签。**`NavLink`与`Link`相比可以在`active`时改变样式，`BrowserRouter as Router`设定别名**。
React Router 具体标签属性和使用可以参考[官方文档](https://reactrouter.com/web/api/NavLink/exact-bool)
```
import { BrowserRouter as Router, Route, NavLink } from 'react-router-dom'
or 
import { BrowserRouter as Router, Route, Link } from 'react-router-dom
```
3. 开头引入路由跳转组件
```
import Home from '../components/Home'
import Analysis from '../components/Analysis'
import Person from '../components/Person'
```
4. 列表存放路由参数，模拟接收后台请求数据。此处采用遍历路由参数信息列表的方式搭建路由。相较于单独搭建，该方法灵活性更高。
```
const routeList = [
    {
        path: "/",
        exact: true,
        title: "首页",
        component: Home,
        icon: "iconfont icon-yemian-copy"
    },
    {
        path: "/analysis/",
        exact: false,
        title: "相关性分析",
        component: Analysis,
        icon: "iconfont icon-fenxi"
    },
    {
        path: "/person/",
        exact: false,
        title: "个人档案",
        component: Person,
        icon: "iconfont icon-icon-text-fn-documentation"
    },
]
```
5. 遍历路由参数列表，设定`<Route>`
```
routeList.map((item, index) => {
    return (
        <Route
            exact={item.exact}
            path={item.path}
            key={index}
            component={item.component}
        ></Route>
    )
})
```
6. 遍历路由参数列表，引用`<NavLink>`标签
```
routeList.map((item, index) => (
    <div className="subTitle" key={index}>
        <i className={item.icon} key={index}></i>
        <li className="textItem">
            <NavLink
                exact={item.exact} //NavLink-exact: if true, 精确匹配active
                to={item.path}
                key={index}
                className="link"
                activeClassName="activeLink"
            >{item.title}</NavLink>
        </li>
    </div>
))
```

**注意事项：**

* 注意路由的层级关系为 `<BrowserRouter>`>`<Route>`=`<Link>`，即`<Route>`和`<Link>`都必须被包裹在 `<BrowserRouter>`内使用。
```
<BrowserRouter>
    <Link to=""></Link>
    <Route path="" exact component=""></Route>
</BrowserRouter>
```
* `Link`,`NavLink`,`Route`常用参数举例：
to: 跳转路径 "string"
path: 路由路径 "string"
exact: 是否精确匹配 (true/false)
component: 目标组件
activeClassName: 链接激活后的样式 "string"
activeStyle: 链接激活后的样式 (css obj)
NavLink - exact: NavLink 中定义的 exact 指确认 active 激活的精确匹配，若不设置可能导致 `"/""/post/"`两个链接都显示 active 样式
`<Link to="/"></Link>`
`<NavLink to="/post/" activeClassName="activeAction" activeStyle={{color:red,}}></NavLink>`
`<Route path="/post/" exact=true component="Home"></Route>`
* `map()` 遍历时要给所遍历的组件加上`key`参数，且`key`要保证唯一性，此处用`key={index}`，但实际应用中往往采用其他办法。
* `<ul><li></li></ul>` 无序列表书写格式，不要写错(填坑)

### 精确匹配 exact
精确匹配从字面上理解，就是你的路径信息要**完全匹配**成功，才可以实现跳转，匹配一部分是不行的。
例如路由设置了两个跳转路径`/`和`/post/`，若`exact=false`，则路由既可以跳转到`/`对应的组件，也可以跳转到`/post/`对应的组件。
值得一提的是，我们一般在首页`/`的时候采用精确匹配，其余时候不用，当然这也要考虑的实际项目需求，酌情而定。

## 路由动态传参
参考链接：[路由动态传参](https://www.cnblogs.com/yky-iris/p/9161907.html)

### 通配符传参
在进行路由跳转的过程中，我们可以通过 url 向子组件传递一些参数，这也被称为路由的动态传参。
**设置动态传参的步骤如下：**

1. 在`Route`上设置动态传值 **(设置传参的 key 值)**
```
<Route path="/post/:key" component={Home}></Route>
```
2. 在`Link`上传递值 **(value)**
```
<Link to="/post/123"></Link>
```
3. 在子组件上获取传递的参数
```
value === this.props.match.params.key
value === 123
```

**注意事项：**

* 如果不往 url 里传任何东西，是没办法匹配路由成功的。即若设置了动态传参`:`，则必须要给定一个参数。
* params：传递过来的参数，`key`和`value`值。通过 `this.props.params.xxx` 取值，xxx = key

> 优点：简单快捷，并且在刷新页面的时候，参数不会丢失。
> 缺点：只能传字符串，并且如果传的值太多的话，url会变得长而丑陋。
> 如果想传对象的话，可以用`JSON.stringify()`,想将其转为字符串，然后另外的页面接收后，用`JSON.parse()`转回去。

#### 实例
**设置动态路由 & Link传值**
```
import React from 'react'
import { BrowserRouter as Router, Route, NavLink } from 'react-router-dom'
import Home from '../components/Home'
import Analysis from '../components/Analysis'
import Person from '../components/Person'
import "../index.css"
import "../images/iconfont.css"

const HomePage = () => {
    const routeList = [
        ...
        {
            // href 定义了 value
            // path 定义了 key
            href: "/person/齐天大圣",
            path: "/person/:name",
            exact: false,
            title: "个人档案",
            component: Person,
            icon: "iconfont icon-icon-text-fn-documentation"
        },
    ]
    return (
        <div>
            ...
        </div>
    )
}

export default HomePage
```

**子组件接收参数**
用了 `Ant Design` 的标签，标签具体属性可参照官网。
```
import React, { Component } from 'react';
import '../index.css'
import "antd/dist/antd.css"
import {Input} from "antd"

class Person extends Component {
    constructor(props) {
        super(props);
        this.state = {  }
        this.onSearch = this.onSearch.bind(this)
    }

    onSearch () {
        console.log(1)
    }

    render() {
        const {Search} = Input 
        return ( 
            <div>
                <div>
                    <h1 className="basicInfo">基本信息</h1>
                </div>
                <div>
                    <Search 
                        // 接收传递的参数
                        placeholder={this.props.match.params.name}
                        allowClear
                        onSearch={this.onSearch}
                        style={{ width: 250, margin: '0 10px' }}
                    />
                </div>
            </div>
         );
    }
}

export default Person;
```

### query 传参
1. `<Route>`定义方式同普通路由相同
```
<Route path="/post/" component={Home}></Route>
```
2. `<Link>`定义前需要声明一个对象，保存 url 地址和传递的参数 `key: value`
```
const name = {
    pathname: "/post/",
    query: {
        xxx: yyy,
        aaa: bbb,
        ...
    }
    // query: value (string or obj)
}
<Link to={name}></Link>
```
3. 子组件参数获取
```
this.props.location.query
```

**注意事项：**

* `pathname, query`都是固定名称，不能修改，其中`pathname`不是驼峰命名，`query`表示传递的参数，等价于通配符的`key`。
* 获取参数时，取的是 url 链接内的 `query` 属性值，而不是 `name`。即`this.props.location.name` 是错误的

> 优点：优雅，可传对象。
> 缺点：刷新页面，参数丢失。

#### 实例
```
const HomePage = () => {
    const routeList = [
        {
            // href 对应的是<Link>标签内的 to 属性值
            href: {pathname:'/analysis/', query:{id:1024, name:"齐天大圣"}},
            path: "/analysis/",
            exact: false,
            title: "相关性分析",
            component: Analysis,
            icon: "iconfont icon-fenxi"
        },
        ...
    ]
    return (
        <div>
            ...
        </div>
    )
}

export default HomePage
```

**子组件接收参数**
```
import React, { Component } from 'react';
import analysisPng from "../images/Analysis.png"
import '../index.css'

class Analysis extends Component {
    constructor(props) {
        super(props);
        this.state = {}
    }

    componentDidMount() {
        // 刷新后参数不保留，会报错，因此在这添一个判断
        if (this.props.location.query === undefined) {
            this.setState({
                id: 0,
                name: "无",
            })
            console.log(this.props)
        } else {
            this.setState({
                id: this.props.location.query.id,
                name: this.props.location.query.name
            })
            // this.props.location.query 接收传递的参数值，此处为一个对象
            console.log(this.props.location.query)
        }
    }

    render() {
        return (
            <div>
                <div>
                    <img className="img" src={analysisPng} alt="" />
                </div>
                <h3>{this.state.id}</h3>
                <h3>{this.state.name}</h3>
            </div>
        );
    }
}

export default Analysis;
```

### state 传参
与 `query` 传参类似：
1. `<Route>`定义方式同普通路由相同
```
<Route path="/post/" component={Home}></Route>
```
2. `<Link>`定义前需要声明一个对象，保存 url 地址和传递的参数 `key: value`
```
const name = {
    pathname: "/post/",
    state: {
        xxx: yyy,
        aaa: bbb,
        ...
    }
    // state: value (string or obj)
}
<Link to={name}></Link>
```
3. 子组件参数获取
```
this.props.location.state
```

## 重定向
**重定向** 和 **跳转** 有一个重要的**区别**，就是跳转式可以用浏览器的回退按钮返回上一级的，而重定向是不可以的。

### 标签式重定向
一般用在不是很复杂的业务逻辑中，比如我们进入Index组件，然后Index组件,直接重定向到Home组件。

1. 引入`<Redirect>`重定向标签
```
import {Redirect} from 'react-router-dom'
```
2. 在`render`函数里使用
```
render() {
    return(
        <div>
            ...
            <Redirect to="...">
            ...
        </div>
    )
}
```
标签式重定向只能写在 `render` 函数中，因此无法绑定到一些业务逻辑上，例如一些业务逻辑绑定函数函数体是独立于`render`函数之外的，将`<Redirect>`标签定义在这类函数体中是不奏效的。因此，我们可以使用另一种编程式的重定向方法，它直接使用JS的语法实现重定向，一般用在业务逻辑比较发杂的场合或者需要多次判断的场合。

### 编程式重定向
调用方法：直接在函数体中加入以下语句即可
```
this.props.history.push("url地址")
```
例如在 `constructor` 构造函数中加入上述代码，即可在组件构造时就进行重定向。
在登陆界面中，我们可以将编程式重定向应用到点击事件判断中，例如当用户名密码与后端一致时，点击确认，重定向到目标路由。这也是标签式重定向无法做到的，具体过程在实战中继续领悟！