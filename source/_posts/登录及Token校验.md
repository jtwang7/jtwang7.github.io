---
title: 登录及Token校验
date: 2021-03-29 16:27
tags: 
- Vue
- Egg
- MySQL
- axios
- 前后端联调
categories: Vue
excerpt: 登陆界面设计，模拟登录流程，后端采用 jsonwebtoken 生成和验证 token。前端实现请求发送，路由跳转等。
---

# 登录及Token校验
## 登录系统工作流程梳理
1. 进入页面，默认重定向到应用页面，但在路由跳转前，先判断浏览器中是否存储 token，若没有 token 且跳转的页面不是登陆页面，则中断本次跳转并强制跳转到登陆页面。
2. 输入用户名与密码，发送至后端校验，若校验成功，后端生成 token 并返回，前端将其存储至 localStorage 和 vuex
3. 同理，在应用页面路由跳转时，每次跳转始终向后端发送校验请求，一旦失败，表明 token 已过期，自动跳转至登陆页面，重新登陆。

## 后端逻辑 (egg)
```
'use strict';

// 导入 jwt
const jwt = require('jsonwebtoken');
const Service = require('egg').Service;

class LoginService extends Service {
  app =  this.app;
  // jwt 的密钥，字符串内容可自定义
  secret = 'MADEBYWANGJT';

  // 当浏览器中没有 token 时触发用户名密码登录函数
  async passwordVerify({ account, password }) {
    // if not exist: got []
    // if exist: got [ {account:string, password:string, token:null, id:number} ]
    const [user] = await this.app.mysql.select('login', {
      where: { account }
    })

    // 用户名不存在
    if (user === undefined) return {
      state: 401,
      message: 'ACCOUNT NOT EXISTED'
    }
    if (user.password === password) {
      if (user.token === null) {
        // jwt.sign() 参数配置
        // payload 负载：可自定义，若 jwt.verify() 校验成功，会显示在 decoded 内
        const payload = {
          name: account,
          admin: true,
        }
        // jwt.sign(payload,secret,options)
        // 作用：生成 token
        // options 可参照 npm jwt 的内容
        const tk = jwt.sign(
          payload,
          this.secret,
          {
            // 加密方式
            algorithm: 'HS256',
            // 有效时间
            expiresIn: '2 hours',
          }
        )
        await this.app.mysql.update('login', {
          id: user.id,
          token: tk,
        });
        return {
          state: 201,
          token: tk,
        };
      }
      return {
        state: 201,
        token: user.token,
      }
    }
    return {
      state: 401,
      message: 'PASSWORD ERROR',
    }
  }

  // 若已知 token, 则进行 token 校验
  async tokenVerify({ account, token }) {
    try {
      let decoded = jwt.verify(token, this.secret);
      console.log('校验成功');
      return {
        state: 200,
        decoded
      }
    } catch (err) {
      // token 过期: 清除数据库token
      const [user] = await this.app.mysql.select('login', {
        where: { account }
      })
      await this.app.mysql.update('login', {
        id: user.id,
        token: null,
      })
      return {
        state: 402,
        message: 'TOKEN EXPIRED',
        err,
      }
    }
  }
}

module.exports = LoginService;
```
**重点：**
`jwt` 的使用：

1. 安装：`npm install --save jsonwebtoken`
2. token 生成：`jwt.sign(payload,secret,options)`
3. token 校验：`jwt.verify(token, secret, [(err,decoded)=>{...}])`

`jwt.sign()`参数解析：

1. payload：负载。(Object) ，由用户自定义。作用：校验成功时所传递的参数
2. secret: 密钥。(String)，由用户自定义。作用：生成 token 的一个环节
3. options: 配置。(Object)，jwt 提供。作用：token 配置项、
4. 返回值：token: String

`jwt.verify()`参数解析：

1. token：token 值。(String) ，jwt 生成的 token。
2. secret: 密钥。(String)，该 token 所对应的密钥。
3. function(err,decoded) {...}：回调函数，可对结果做进一步处理，若返回err，则...；若返回decoded，则...；可选参数，若不指定，则校验成功返回 decoded，失败返回 err，可由 try ... catch ... 捕获。
4. 返回值 decoded: Object  ||  err: Object


## 前端逻辑
```
<template>
  <div class="circle" @click="sign">
    <!-- .stop 修饰符阻止事件冒泡 -->
    <a class="log-in" @click.stop="sign">登录</a>
  </div>
</template>

<script>
import { loginPost } from "@/network/index";

export default {
  name: "LoginBtn",
  methods: {
    async sign() {
      // store 内获取账户和密码
      const {account, password} = this.$store.state.user;

      // 校验是否输入
      if (account === "" || password === "") {
        this.$Message.info("请输入账户或密码 ...");
      } else {
        // 发起请求
        const res = await loginPost(this.$store.state.user);

        if (res.state === 201) {
          // 201状态码：localStorage没有token且登陆成功，token录入缓存
          this.$store.commit("setToken", res.token);
          localStorage.setItem(account, res.token);
          this.$router.push("/main/home");

        } else if (res.state === 200) {
          // 200状态码：本地token与服务端校验成功
          this.$router.push("/main/home");

        } else if (res.state === 401) {
          // 错误，打印错误原因(用户名/密码错误)
          this.$Message.info(res.message);
          this.$router.push("/login");

        } else if (res.state === 402) {
          this.$store.commit("deleteToken", res.state);
        }
      }
    },
  },
};
</script>

<style scoped>
...
</style>
```
**全局路由守卫**
每次跳转前都判断 token 是否有效。
**注意：** 这里判断条件为 `!token && to.name !== 'login'` , 若只判断 token 是否存在，执行 `'next(/login)` 强制跳转时，又会触发 `router.beforeEach` ，导致死循环
```
// 链接跳转前，先校验token
router.beforeEach((to, from, next) => {
  const { account, token } = store.state.user;
  // 若没有 token 且跳往的页面不是登陆页面，则中断当前导航并跳转到登陆页面
  if (!token && to.name !== 'login') {
    alert('token过期,请重新登录~')
    next('/login')
  } else {
    // 若存在 token，则每次跳转都校验 token 是否仍然有效
    loginPost({ account, token }).then(res => {
      console.log(res);
      store.commit('deleteToken', res.state);
    })
    next();
  }
})
```
