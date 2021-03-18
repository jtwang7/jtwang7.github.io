---
title: Vue.js - vue-router
date: 2021-03-18 13:53
tags: 
- Vue
- vue-router
categories: Vue
excerpt: Vue框架学习笔记：vue-router路由使用。作为Vue框架内的重要库之一，vue-router很好地封装了Vue框架的前端路由跳转功能，前端路由的诞生，极大程度上支撑了单页面富应用(SPA)的应用，提高了浏览器页面渲染效率，在单页面富应用(simple page application)普遍使用的时代，掌握vue-router这个Vue内部封装的前端路由是非常必要的。
---

# Vue-Router
vue-router 学习比较系统全面，因此直接拿 vue 项目练习和讲解，此项目主要包含了 vue-router 的使用以及一些辅助的页面组件。
项目主要的结构如下(主要业务逻辑代码在src路径下)：
```
learnvuerouter
├─ .gitignore
├─ babel.config.js
├─ package-lock.json
├─ package.json
├─ public
│  ├─ favicon.ico
│  └─ index.html
├─ README.md
├─ src
│  ├─ App.vue
│  ├─ assets
│  │  └─ logo.png
│  ├─ components
│  │  ├─ HelloWorld.vue
│  │  ├─ Info.vue
│  │  ├─ News.vue
│  │  ├─ Profile.vue
│  │  └─ User.vue
│  ├─ main.js
│  ├─ router
│  │  └─ index.js
│  └─ views
│     ├─ About.vue
│     └─ Home.vue
└─ 路由学习.txt
```

## vue-router
### Part1: vue-router 安装及引用
Vue框架在安装的时候会提示开发者是否选择vue-router库，若选择则在项目生成过程中自动安装vue-router库，并提供相应的示例代码。若没有在项目初始化时安装vue-router，可以通过npm或yarn命令安装。
安装：`npm install --save vue-router`
引用：
`ES6 - import VueRouter from 'vue-router'`
`CommonJs - const VueRouter = require('vue-router')`
**注意：vue-router在开发和发布使用时均会使用，为了在`package.json`中添加相应的依赖，安装时请务必加上`--save`**
[Vue-Router官方中文文档](https://router.vuejs.org/zh/)

### Part2: vue-router 在项目目录结构中的存放位置
在Vue项目内，开发人员一般将业务逻辑写在`/src`路径下，其中`/components`主要用于存放项目组件，`/views`存放整体页面视图，`/assets`存放静态资源，`/router`则主要存放路由配置。
注：项目结构内文件名并不是固定的，但是在行业内已经形成不成文规范，建议遵循此规范合理规划项目目录结构。

vue-router 文件夹路径：`/src/router`
vue-router 入口文件：`/src/router/index.js`
注：入口文件一般写为`index.js`，这是因为 webpack 在编译文件路径时，默认可以省略 `/index.js` 以及 `/node_modules`两个路径(可在webpack配置内更改)。因此引用vue-router时，一般简写为`/src/router`即可。

### Part3: vue-router的正式讲解
#### step1: 引用&插件注册&挂载
vue-router作为一个库，一个插件，在使用前必须被Vue项目引用和注册。
**注：所有Vue插件在使用前都需要经过`Vue.use()`注册安装**
具体步骤如下：
`/src/router/index.js`路由入口文件下

1. 引入 vue-router
2. `Vue.use()`注册安装插件
3. 实例化路由 (常用参数有 routes, mode 等，属性名称均固定，不能随意设置)
4. 导出路由

`/src/index.js`项目入口文件下

1. 引入创建的路由实例router
2. 在Vue实例下挂载router路由实例

**完整代码：**
```
// /src/router/index.js(路由入口文件)
import Vue from 'vue'

// step1: 引入vue-router
import VueRouter from 'vue-router'

// Vue.use() 当使用插件时都需要使用 Vue.use()
// step2：Vue.use(插件)
Vue.use(VueRouter)

// step3: 实例化router路由
const router = new VueRouter({
    routes,
    mode: 'history'
})

// step4: 导出路由实例
export default router
```
```
// /src/main.js(项目入口文件)
import Vue from 'vue'
import App from './App.vue'

// step1: 导入router实例，/index.js 可以省略
import router from './router'

// 产品提示，在发布阶段改为true
Vue.config.productionTip = false

new Vue({
// step2: 将router实例传入 Vue 实例进行挂载(此处用了ES6字面量增强写法)
  router,
  render: h => h(App)
}).$mount('#app')
// $mount() <=> el: 'xxx'
```

#### step2: 路由数组配置
我们已经有了一个router实例(有且只有一个)，基础配置如下：
```
const router = new VueRouter({
  base: process.env.BASE_URL,
  mode: 'history',
  routes,
})
```
可以看到，配置参数中有一项 `routes`，我们需要在里面配置路由和组件间的映射关系。
由于路由和组件的映射有多组，因此 `routes` 属性接收一个 `Array` 数组类型，本文后续将其称为路由数组。

路由数组往往包含了复杂的映射关系，我们一般将其单独声明一个变量存储，然后通过ES6字面量增强的写法配置到 router 实例中。
```
// 不要忘记引入组件
import Home from '@/components/Home.vue'

const routes = [
  // 路由重定向
  {
    path: '/',
    redirect: '/home',
  },
  {
    // path 相对路径
    path: '/home',
    // component 路径对应跳转的组件
    component: Home,
    name: 'Home',
  },
]
```
路由数组接收**对象**表示映射关系。
上述代码为基本的路由数组配置，包括了路由重定向以及路由和组件的映射：

##### 路由和组件映射关系配置
常用属性(属性名Vue规定，不能随意更改)：

1. path: 路由相对路径。`'/'`表示根路径，一般为`http://localhost:8080/`
2. component: 路由跳转的组件
3. name: 路由标识名

```
import Home from '@/components/Home.vue'

//示例：路径跳转到'/home'时，展示Home组件(要将组件导入，否则路由配置不知道Home是什么)
{
    path: '/home',
    component: Home,
    name: 'Home',
}
```

##### 路由重定向
路由重定向常用属性：

1. path: 路由相对路径
2. redirect: 路由重定向的目标路径

```
// 示例：当路径为'/'时，自动重定向(跳转)到'/home'路径
{
    path: '/',
    redirect: '/home',
},
```

##### 路由进阶配置——懒加载
**路由懒加载**
定义：实现了不同路由组件的分割，当路由被访问时才加载对应组件。
实现：当发生路由跳转时，调用箭头函数，触发import，导入文件，而非在index.js开始就import组件。

普通路由加载方式：在最开始就 import 组件
```
import Home from '@/components/Home.vue'

{
    path: '/home',
    component: Home,
    name: 'Home',
}
```
**路由懒加载方式**：当发生路由跳转时导入文件
```
{
    path: '/about',
    name: 'About',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: ()=>import('@/components/About.vue'),
},
```
**路由懒加载管理**
```
建议将路由懒加载单独提取赋值，方便管理
const About = () => import('../views/About.vue')
const User = () => import('../components/User.vue')
const News = () => import('../components/News.vue')
const Info = () => import('../components/Info.vue')
const Profile = () => import('../components/Profile.vue')

...

{
    path: '/about',
    name: 'About',
    component: About,
},
```

##### 动态路由配置
在路由跳转过程中，我们往往需要传递某些信息给路由跳转后的组件。
动态路由配置可以**让 url 携带一些配置参数进行传递**，主要为 `params` 和 `query` 两种方式。
作用：路由切换时，传递参数。

**`params`动态路由配置：**
```
const User = () => import('../components/User.vue')

...

{
    // params: path 配置时需要事先通过'/:xxx'预留params位置
    path: '/user/:id',
    component: User,
}
```
**`params`参数接收：**
params 传递的参数可通过挂载在 Vue 实例下的全局 route 的 params 属性获取：
`this.$route.parmas.id`

**`query`动态路由配置：**
query 配置路由时，不需要 `/:xxx` 预留位置，而是在 url 中添加 `xxx=aaa&yyy=bbb` 传递参数。
query 传参格式参考：
协议头://主机名:端口号?query
`http://localhost:8080?name=wjt&age=18`
```
const Profile = () => import('../components/Profile.vue')

...

{
    // query 动态传参：path 配置普通形式
    path: '/profile',
    component: Profile,
}
```
**`query`参数接收：**
query 传递的参数可通过挂载在 Vue 实例下的全局 route 的 query 属性获取：
`this.$route.query`
获取的是对象类型，形如`{name:'wjt',age:18}`

##### 嵌套路由配置
当路由跳转组件内还有路由和组件的映射关系时，需要在当前路由内配置嵌套路由。
```
{
    path: '/about',
    name: 'About',
    component: About,
    // 嵌套路由
    children: [
      {
        // 此处不需要加 '/'，加'/'表示根路径，即 localhost 之后的路径。
        // 此处承接父级 '/about'，嵌套路由会自动拼接为 '/about/news'
        path: 'news',
        component: News,
      },
      {
        path: 'info',
        component: Info,
      },
    ],
},
```
嵌套路由配置同普通路由配置相同，唯一要注意的是**嵌套路由在相应路由内通过`children`属性配置**，由于子路由也是路由，因此**`children` 属性接收的也是数组类型，数组内为子路由对象**。

#### 导航守卫
导航守卫主要用于**监听路由的创建/跳转等，并执行回调函数**。类似于Vue的生命周期。
导航守卫根据作用域不同可以分为：

1. 全局导航守卫
2. 路由独享守卫
3. 组件守卫

本文主要介绍全局导航守卫，让读者了解导航守卫大致的作用以及使用方式，具体细节以及其余守卫请参考Vue-Router官方文档。

全局导航守卫：作用于全局，内部又根据执行回调的节点不同分为前置守卫，后置守卫等等

**前置守卫** (通过router实例的beforeEach()方法执行)
`xxx.beforeEach()`: 在路由跳转前自动执行自定义操作。(监听全局的路由跳转)
`.beforeEach()` 是 router 实例的方法，**接收函数作为参数**。其**回调函数中又包含三个参数**：

1. to: 跳转的目的路由, 源码内 to 是 route 类型，取数据方法同 $route.xxx
2. from: 跳转前的路由, 源码内 from 是 route 类型，取数据方法同 $route.xxx
3. next: **next()，必须要调用，否则不能执行下一步**。

```
{
    // params: path 配置时需要事先通过'/:xxx'预留params位置
    path: '/user/:id',
    // component: User,
    component: User,
    meta: {
      title: '用户',
    },
},

...

// 此处前置守卫作用是：发生路由跳转前，获取源路由(to指向的路由)的meta属性中的title值。
// 此时 to 实际上就是源路由的 route 实例(注意是route实例，不是router实例)
router.beforeEach((to, from, next) => {
  document.title = to.meta.title;
  // next()内部还能传入参数，具体功能参考 Vue-router 官网
  next()
})
```

**后置钩子(守卫)**
`.afterEach()`: 在路由跳转后自动执行自定义操作。(监听全局的路由跳转)
`.afterEach()` 是 router 实例的方法，**接收函数作为参数**。其**回调函数中又包含两个参数**：

1. to: 跳转的目的路由, 源码内 to 是 route 类型，取数据方法同 $route.xxx
2. from: 跳转前的路由, 源码内 from 是 route 类型，取数据方法同 $route.xxx

!!!注意 .afterEach() 没有next: 因为路由已经跳转完成，不需要进行其他操作，因此没有 next() 函数。
```
router.afterEach((to, from) => {
  console.log('--------------')
})
```