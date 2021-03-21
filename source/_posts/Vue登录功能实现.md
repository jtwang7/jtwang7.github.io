---
title: Vue项目登录功能实现
date: 2021-03-21 12:02
tags: 
- Vue
- 前后端联调
- Egg
- MySQL
categories: Vue
excerpt: 项目登陆页面制作，技术栈：vue + vue-router + iviewUI + egg.js。通过路由跳转实现页面整体切换(主页面是组件，登陆页面也是组件，路由跳转更换这两个组件的展示即可切换整体页面)。此外还涉及样式配置，egg.js后端配置，前端业务逻辑，post请求等。
---

# Vue登录功能实现

## 技术栈
vue + vue-router + iviewUI + egg.js

## 登录页面制作
![](/img/posts_img/20210321104031010_1796.png)

**`Login.vue`**
```
<template>
  <div id="login-container">
    <div class="login-body">
      <h1 class="title">账户登录</h1>
      <LoginBar>
        <template v-slot:icon>
          <svg class="icon" aria-hidden="true">
            <use xlink:href="#icon-user"></use>
          </svg>
        </template>
        <template v-slot:input>
          <Input
            v-model="account"
            placeholder="Account ..."
            style="width: 200px; padding-bottom: 15px"
            class="input-style"
          />
        </template>
      </LoginBar>
      <LoginBar>
        <template v-slot:icon>
          <svg class="icon" aria-hidden="true">
            <use xlink:href="#icon-password"></use>
          </svg>
        </template>
        <template v-slot:input>
          <Input
            v-model="password"
            type="password"
            password
            placeholder="Password ..."
            style="width: 200px; padding-bottom: 10px"
            class="input-style"
          />
        </template>
      </LoginBar>
      <LoginBtn :account="account" :password="password"/>
    </div>
  </div>
</template>

<script>
import "@/assets/login/iconfont.js";
import LoginBar from "./LoginBar";
import LoginBtn from './LoginBtn'

export default {
  name: "Login",
  components: {
    LoginBar,
    LoginBtn,
  },
  data() {
    return {
      account: "",
      password: "",
    };
  },
};
</script>

<style scoped>
/* style scoped 包裹样式只能在该组件内使用 */
#login-container {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  z-index: -1;
  /* style内引入图片不能用@ */
  background-image: url("../../../assets/login-background.png");
  background-size: 100%;
}

.login-body {
  margin-top: 20vh;
  margin-left: 20vw;
  margin-right: 20vw;
  height: 200px;

  display: flex;
  flex-direction: column;
  justify-content: space-around;
  align-items: center;
}

.login-body .title {
  color: white;
  font-weight: bolder;
  font-size: 2em;
  margin-bottom: 20px;
}

/* iconfont 下载的图标通过 font-size 调大小 */
.login-body .icon {
  font-size: 30px;
  margin-right: 10px;
}

.login-body .input-style {
    margin-right: 2vw;
}
</style>

<style>
/* 全局CSS样式 */
.icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
</style>
```

**`LoginBar.vue`**
```
<template>
  <div id="login-bar">
      <slot name="icon"></slot>
      <slot name="input"></slot>
  </div>
</template>

<script>
export default {
  name:'LoginBar'
}
</script>

<style scopoed>
.login-bar {
  display: flex;
  justify-content: space-around;
  align-items: center;
}
</style>
```

### 技术点总结：
css 样式篇：

1. `<style scoped>` 管理当前组件内CSS样式，`<style>` 管理全局CSS样式。
2. `<style>` 内外部引入 CSS 样式通过：`@import '../../../xxx.css'`，注意此处必须为 `'../../'` 路径，不能为 `@/xx/xx/`，踩坑的地方是程序编译时会加载成`./@/xxx/xxx/`路径。
3. iconfont 图标库下载的图标，大小可以通过`font-size`调整。
4. iconfont Symbol 类图标使用 (可参考下载文件中的`demo_index.html`)：在 `<script>` 中引入 `'iconfont.js'` 文件，根据样例代码在模板中引用即可。

组件篇：

1. `<LoginBar>` 是自定义的预留插槽组件。
2. `<LoginBtn>` 是封装的登录按钮(讲业务逻辑时贴代码)。
3. `<Input>`是[iviewui](https://www.iviewui.com/)的封装组件，按照官方文档使用即可。


## 主应用“登录/注册”按钮制作
![](/img/posts_img/20210321105417697_12639.png)

**`NoLogged.vue`**
```
<template>
  <div id="no-logged" v-if="isLogged">
    <!-- mouseover.native -->
    <!-- 监听原生事件时，需要通过.native修饰符 -->
    <!-- 自定义事件官方文档：https://cn.vuejs.org/v2/guide/components-custom-events.html -->
    <router-link
      :to="sign"
      :class="signHover ? 'text-active-style' : 'text-style'"
      @mouseover.native="signHover = true"
      @mouseout.native="signHover = false"
      >登录</router-link
    >
    /
    <router-link
      :to="reg"
      :class="regHover ? 'text-active-style' : 'text-style'"
      @mouseover.native="regHover = true"
      @mouseout.native="regHover = false"
      >注册</router-link
    >
  </div>
</template>

<script>
export default {
  name: "NoLogged",
  data() {
    return {
      signHover: false,
      regHover: false,
    };
  },
  props: {
    sign: {
      type: String,
      default: "",
    },
    reg: {
      type: String,
      default: "",
    },
    isLogged: {
      type: Boolean,
      default: true,
    },
  },
};
</script>

<style>
#no-logged {
  color: white;
}

#no-logged .text-style {
  color: white;
}

#no-logged .text-active-style {
  color: rgb(250, 150, 133);
}
</style>
```

### 技术点总结：
此处用 `<router-link>` 组件监听和实现路由跳转 (编程式路由也可以实现)。

**业务逻辑：**

1. 点击登录或注册，实现路由跳转(此处暂时只实现了登陆页面的跳转)。
2. 样式修改：当鼠标移动到登录或注册上时，通过监听原生事件动态修改样式。

**重点：**

1. vue 监听原生事件(mouseover, keyup, ...)时，需要通过 `.native` 修饰符。详情参考官方文档：[自定义事件](https://cn.vuejs.org/v2/guide/components-custom-events.html)
2. 动态改变样式(两种方法)：vue的[动态class绑定](https://cn.vuejs.org/v2/guide/class-and-style.html#对象语法)`:class="{'styleName1':isActived, 'styleName2':!isActived}"` 或者通过三元运算符`:class="isActived?'styleName1':'styleName2'"`

## 主应用登陆成功后“用户头像”展示
![](/img/posts_img/20210321110943778_9692.png)

**`Logged.vue`**
```
<template>
  <Dropdown v-if="isLogged" @on-click="click">
    <a href="javascript:void(0)">
      <img
        class="avatar-container"
        src="../../../assets/userAvatar.png"
        alt=""
      />
      <Icon type="ios-arrow-down" style="color:white;"></Icon>
    </a>
    <DropdownMenu slot="list">
      <DropdownItem>个人中心</DropdownItem>
      <DropdownItem>首页</DropdownItem>
      <DropdownItem disabled>权限管理</DropdownItem>
      <DropdownItem divided name="signout">退出登录</DropdownItem>
    </DropdownMenu>
  </Dropdown>
</template>

<script>
export default {
  name: "Logged",
  props: {
    isLogged: {
      type: Boolean,
      default: false,
    },
  },
  methods: {
      click(name) {
          if(name === 'signout') {
              this.$router.replace('/main/gaode/tourist');
          }
      }
  },
};
</script>

<style scoped>
img {
  width: 100%;
}

.avatar-container {
  border-radius: 50%;
  width: 40px;
}
</style>
```

### 技术点总结：
css 样式篇：

1. `img {}` 对所有 img 标签进行样式管理。
2. 图片样式裁剪为圆形的方法：在 img 标签内定义样式 `border-radius: 50%`。

组件篇：

1. 下拉菜单采用 [Dropdown](https://www.iviewui.com/components/dropdown) 。

业务逻辑篇：

1. 监听下拉菜单内的点击事件`on-click` (详见官方文档API)，当点击**退出登录**时，通过编程式路由跳转到游客界面(即**登录/注册**)。

## “登录/注册”与“用户头像”切换
**`Sign.vue`**
```
<template>
  <div id="sign">
    <Logged :isLogged='Logged' />
    <NoLogged :isLogged='!Logged' sign="/login" reg="/login" />
  </div>
</template>

<script>
import Logged from './Logged'
import NoLogged from './NoLogged'

export default {
  name: "Sign",
  components: {
    Logged,
    NoLogged,
  },
  computed: {
    Logged() {
      console.log(this.$route.params.id);
      if (this.$route.params.id === 'tourist') {
        return false;
      } else {
        return true;
      }
    }
  }
};
</script>

<style>
#sign {
  margin-left: 30px;
}
</style>
```

### 技术点总结：
**计算属性 + 路由传参**
通过计算属性判断当前路由：
若路由 parmas 动态传参值为 `tourist`，则“登录/注册”的 `v-if=true`；
若路由 parmas 动态传参值为 `用户id`，则“用户头像”的 `v-if=true`；


## 登录按钮及其业务逻辑
**`LoginBtn.vue`**
```
<template>
  <div class="circle" @click="sign">
    <!-- .stop 修饰符阻止事件冒泡 -->
    <a class="log-in" @click.stop="sign">登录</a>
  </div>
</template>

<script>
import { login } from "@/network/login";

export default {
  name: "LoginBtn",
  props: {
    account: {
      type: String,
      default: "",
    },
    password: {
      type: String,
      default: "",
    },
  },
  methods: {
    async sign() {
      if (!this.account) {
        this.$Message.info("请输入用户名...");
        // 中断该方法继续进行，否则还会进行请求
        return;
      } else if (!this.password) {
        this.$Message.info("请输入密码...");
        return;
      }

      // 需要设置拦截器！！否则请求的是axios包装的值
      const res = await login({
        account: this.account,
        password: this.password,
      });

      if (res === 'success') {
        this.$router.replace(`/main/gaode/${this.account}`);
      } else if (res === 'refused') {
        // iviewUI的方法，不是Vue内部的
        this.$Message.info("密码错误！请重新登录...");
      } else {
        this.$Message.info("用户名不存在...");
      }
    },
  },
};
</script>

<style scoped>
.circle {
  width: 100px;
  height: 100px;
  background: rgb(248, 143, 74);
  -moz-border-radius: 50px;
  -webkit-border-radius: 50px;
  border-radius: 50px;

  display: flex;
  justify-content: center;
  align-items: center;

  margin-top: 20px;
}

.log-in {
  font-weight: bolder;
  font-size: 2em;
  display: block;

  color: rgb(119, 119, 143);
}
</style>
```

### 技术点总结：

1. `.stop` 修饰符阻止事件冒泡：此例中，子组件和父组件都监听了click事件，当子组件监听到click事件后，由于事件冒泡，父组件也会重复响应。通过`.stop`修饰符可以解决该问题。
2. `this.Message.info('xxx')`为 [iviewui 的全局提示事件](https://www.iviewui.com/components/message)，非 vue 自带。
3. 判断是否登陆成功的业务逻辑：若未输入用户名，提示(且中断函数) => 若未输入密码，提示(且中断函数)；若上述完成，向后端发送请求，后端判断后返回数据，若成功则返回`success`，失败返回`refused`，用户名不存在返回`empty`，前端异步等待接收结果后，作进一步判断(`success`:编程式路由跳转，`refused || empty`:全局提示)


## egg.js后端业务逻辑
### router.js
```
'use strict';

/**
 * @param {Egg.Application} app - egg application
 */
module.exports = app => {
  require('./router/map')(app);
  require('./router/account')(app);
};
```
**技术点总结：**
此处对路由做了管理，具体路由分类放至创建的`router`文件夹下，通过CommonJS模块导入方法`require()`引入。各路由文件返回的是方法且接受`app`参数，因此 require 请求后得到的结果当作方法使用。

### /router/account.js
```
module.exports = app => {
    const { router, controller } = app;
    // 用户账号路由
    router.post('/login',controller.getAccount.verify);
};
```
**技术点总结：**
导出Post路由，路径指向`/login`，调用 controller 对象内挂载的 getAccount 实例的 verify (异步)方法。

### /controller/getAccount.js
```
'use strict';

const Controller = require('egg').Controller;

class GetAccountController extends Controller {
    async verify() {
        const { ctx } = this
        ctx.body = await ctx.service.account.verify()
  }
}

module.exports = GetAccountController;
```
**技术点总结：**
从 `egg.Controller` 中导入 Controller 类，继承该基类，类内定义方法：等待 `service.accout.verify()` 完成并将值返回给 `ctx.body` 暴露到响应体内传给前端。记住导出该类。

### /service/account.js
```
'use strict';

const Service = require('egg').Service;

class AccountService extends Service {
    async verify() {
        const { ctx, app } = this;
        // body: {account:'xxx',password:'yyy'}
        const { account, password } = ctx.request.body
        // 数据库若没有用户则返回 []
        // 数据库存在用户，返回形如 [ RowDataPacket { id: 1, user: 'admin', password: '123456' } ]
        const target = await app.mysql.select('account', {
            where: {
                user: account,
            }
        })
        if (target.length !== 0) {
            if (target[0].password === password) {
                return 'success'
            } else {
                return 'refused'
            }
        } else {
            return 'empty'
        }
    }
}

module.exports = AccountService;
```
**技术点总结：**
`/service`主要管理数据库一类的操作，此处业务逻辑：从前端请求(post)的body中读取账户密码，通过账户在数据库内查找用户，若存在该用户，返回形如 `[ RowDataPacket { id: 1, user: 'admin', password: '123456' } ]`的数组，若不存在则返回空数组`[]`，之后再验证密码即可。
注意：egg.js 中默认开启了 csrf 安全防护，请参考[官方文档](https://eggjs.org/zh-cn/basics/router.html)，在`config/config.default.js`中关闭。

## 数据库管理
采用 DBeaver 可视化管理数据库：
新建表 - 新建列 - 添加数据 - 设置约束(主键)
![](/img/posts_img/20210321115010709_24364.png)


## 前端 Post 请求及路由配置
### Post请求
**`/network/index.js`**
```
export const basePost = (config) => {
    const instance = axios.create({
        baseURL: 'http://127.0.0.1:7001',
        method: 'post',
        timeout: 5000,
    })

    instance.interceptors.response.use(
        res => {
            return res.data
        },
        err => {
            console.log(err);
        })

    return instance(config)
}
```
**技术点总结：**
统一封装基础 post 请求配置，并设置响应拦截器，用于过滤 axios 对响应结果的包装。

**`/network/login.js`**
```
import { basePost } from './index'

export const login = (config) => {
    const { account, password } = config;
    return basePost({
        url: '/login',
        data: {
            account,
            password,
        }
    })
}
```
**技术点总结：**
独立封装符合当前业务逻辑的 post 请求，返回一个 Promise 对象(axios实例本身就是 Promise 对象，当获得响应结果后，可以通过该 Promise 执行后续处理操作)。

### 路由配置
```
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

// 应用页面
const MainRoute = () => import('@/views/Main')
// main-地图路由
const GaodeRoute = () => import('@/components/common/map/EchartsGaode');
const MapboxRoute = () => import('@/components/common/map/Mapbox');

// 登陆页面
const LoginRoute = () => import('@/components/common/login/Login');

const routes = [
  {
    path: '/',
    redirect: '/main/gaode/tourist',
  },
  {
    // 应用页面
    path: '/main',
    component: MainRoute,
    children: [
      {
        path: 'gaode/:id',
        component: GaodeRoute,
      },
      {
        path: 'mapbox',
        component: MapboxRoute,
      },
    ],
  },
  {
    // 登陆页面
    path: '/login',
    component: LoginRoute,
  },
]

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

export default router

```
**技术点总结：**
路由代码包括了主应用界面的路由配置和登陆页面的路由配置。主页面组件`<Main />`和登陆界面组件`<Login />`均放在项目入口组件`<App />`下，通过`<router-view>`展示。
注意：`<Main />` 和 `<Login />`是同级关系，这样可以实现整体页面的刷新(及发生路由跳转时，登陆页面整体替换主页面)，而`<Main />`组件内的路由跳转要写在其子级路径，如代码所示。

#### 项目入口组件 App.vue
```
<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'App',
}
</script>


<style>
</style>
```
此处`<router-view>`展示的是`<Main />` 和 `<Login />`，及最顶层的路由关系。