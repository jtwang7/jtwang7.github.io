---
title: Vue.js - axios
date: 2021-03-12 18:45
tags: 
- Vue
- axios
- ES6
categories: Vue
excerpt: Vue框架学习笔记：axios异步请求。网络请求是前后端联调的关键一环，axios库在众多网络请求方式中脱颖而出，在Vue-cli3中，官方也明确将axios作为框架主要的网络请求方式。本文记录axios的基本使用，以及axios将框架目录结构内的存放方式。
---

# Vue-axios
## Part1: axios库安装和引用
Vue框架内不包含axios库，因为axios库本身就作为一个独立的第三方库存在并长期维护。因此，Vue框架内使用axios就有必须引入axios第三方库。
安装：`npm install --save axios`
引用：
ES6 - `import axios from 'axios'`
CommonJs - `const axios = require('axios')`
**注意：axios在开发和发布使用时均会使用，为了在`package.json`中添加相应的依赖，安装时请务必加上`--save`**
[axios-npm文档](https://www.npmjs.com/package/axios)

## Part2: 全局axios
全局axios: 未创建实例的axios对象称为全局axios

**测试网站：(练习时可用以下网址测试)**
`httpbin.org`
`123.207.32.32:8000/home/multidata`
`123.207.32.32:8000/home/data?type=sell&page=3`

### axios({config})
A. axios 接收**对象**作为参数,**在参数内做相应的配置**.
参数配置项可参考axios官方文档，以下列出几个常用的属性：

1. url: url地址
2. method: 请求方式(get | post | ...) //默认值 get
3. params: 接收对象类型, 专门针对get请求的参数拼接(query部分)
4. timeout: 设定超时阈值
5. baseURL: 设定基础url,若设定了则 url 属性传入相对路径即可

B. axios 支持 Promise, 其**本身包裹并执行了网络请求(异步操作)**, 通过 .then() | .catch() 等获取传递的数据进行进一步处理.
```
// 向httpbin.org发送get请求，并对请求成功的传递值做相应的打印
axios({
    url: 'http://httpbin.org/get',
    method: 'get',
}).then(
    (res) => {
        console.log(res)
    }
)

// url带有query传递参数的请求，query通过params传递
axios({
    baseURL: 'http://123.207.32.32:8000',
    url: '/home/data',
    method: 'get',
    params: {
        type: 'sell',
        page: 3,
    }
}).then(
    res => console.log('baseURL',res)
)
```

### axios.get(url, {config})
axios 除了所有参数都在config对象内定义之外，还有其他的写法，例如本例指定以get方式发送网络请求。此时参数包括了 url 和其他剩余配置参数(以对象包裹)。
```
// axios.get(url,{config})
axios.get(
    'http://123.207.32.32:8000/home/multidata',
    {timeout: 5000},
).then(
    res => {
        console.log(res)
    }
)
```

### axios.all()
有些网络请求是需要合并发送的，当请求存在多个且需要等待所有请求结果均返回后才落定状态(resolve | reject)，那么需要用到 axios 的 all() 方法。
作用：发送并发请求,所有请求均完成后执行处理程序
参数：接收数组作为参数,数组内接收多个请求. 与 Promise.all() 类似.
返回值：数组类型,包含多个请求结果对象. 若期望将数组展开,可调用 axios.spread 方法(ES6数组解构方法也可以)
```
axios.all([
    axios({
        url: 'http://123.207.32.32:8000/home/multidata',
    }),
    axios({
        url: 'http://123.207.32.32:8000/home/data',
        method: 'get',
        params: {
            type: 'sell',
            page: 3,
        }
    })
]).then(
    results => console.log(results)
)

axios.all([
    axios({
        url: 'http://123.207.32.32:8000/home/multidata',
    }),
    axios({
        url: 'http://123.207.32.32:8000/home/data',
        method: 'get',
        params: {
            type: 'sell',
            page: 3,
        }
    })
]).then(
    // axios.spread 使用
    axios.spread((res1, res2) => {
        console.log('spread', res1);
        console.log('spread', res2);
    })
)

axios.all([
    axios({
        url: 'http://123.207.32.32:8000/home/multidata',
    }),
    axios({
        url: 'http://123.207.32.32:8000/home/data',
        method: 'get',
        params: {
            type: 'sell',
            page: 3,
        }
    })
]).then(
    // 数组解构
    ([res1, res2]) => {
        console.log('array', res1);
        console.log('array', res2);
    }
)
```

### axios 全局配置
语法：`axios.defaults.xxx = yyy`
xxx 为axios的配置参数
举例：
```
axios.defaults.timeout = 5000
```

## Part3: axios实例
**axios 实例创建:**  `let aaa = axios.create({config})`
局部axios实例创建时配置的 config 为初始化参数配置. 后续还可以通过给实例传递参数进行配置. 
举例： `aaa({timeout:5000})` 在axios实例基础上，额外配置超时阈值参数。
**注意：尽量不要用全局axios, 要根据不同应用场景创建局部axios实例.**
```
export const request = (config) => {
    const instance = axios.create({
        baseURL: 'http://123.207.32.32:8000',
        method: 'get',
        timeout: 5000,
    })
    // 创建的实例返回 Promise 对象
    return instance(config)
}
```

## Part4: axios 拦截器
axios 拦截器又分为**请求拦截器`(success/failure)`和响应拦截器`(success/failure)`**
拦截器内又细分为成功拦截和失败拦截两个状态。

### request请求拦截器
语法：`axios实例.interceptors.request.use()`   (看源码还有`axios实例.interceptors.request.eject()`，但暂时没碰到，后续补充)
作用：请求拦截器拦截的是网络请求的配置信息(config)
参数：`.use((config)=>{},(err)=>{})` 接收两个回调函数作为参数，第一个回调函数传入截获的网络请求配置参数config，第二个回调函数在拦截失败时执行，传入err错误对象。
**在请求拦截器中，通常执行以下业务需求：**

1. 对config信息预处理
2. 发送网络请求时,都希望在界面中显示请求图标
3. 某些网络请求(比如登录(token)),必须携带一些特殊信息.可在请求时拦截,进行判断,若不符合要求则展示相应的错误提示.

**特别注意！！！**：拦截的config对象必须被手动返回，否则config会因为被拦截器拦截，导致无法向后传递。
```
instance.interceptors.request.use(
    // 请求拦截器拦截网络请求的配置信息(config),此处config可以随便命名,但最好命名为config,含义更清晰
    config => {
        console.log(config);
        // config 必须被返回,因为拦截器将其拦截,我们做了相应操作后要返回config.
        return config
    },
    err => {
        console.log(err);
    },
)
```

### response响应拦截器
语法：`axios实例.interceptors.response.use()`   (看源码还有`axios实例.interceptors.response.eject()`，但暂时没碰到，后续补充)
作用：响应拦截器拦截网络响应后传递回来的结果对象(result)
参数：`.use((res)=>{},(err)=>{})` 接收两个回调函数作为参数，第一个回调函数传入截获的响应结果对象res，第二个回调函数在拦截失败时执行，传入err错误对象。
**在响应拦截器中，最常执行的是对响应结果的过滤：**
axios通常会对响应结果做一层自己的包装,包装成一个对象
对象内包含(config,data,headers,...)属性,其中真正的网络请求响应数据在 data 中
因此平时调用时通常需要 res.data.data 获取,在响应拦截中可对此做一个过滤
**特别注意！！！**：拦截的res对象必须被手动返回，否则res会因为被拦截器拦截，导致无法向后传递。
```
instance.interceptors.response.use(
    // 响应拦截器拦截网络响应后传递回来的结果对象(result)
    res => {
        console.log(res);
        res = res.data;
        // 同样需要返回被拦截的 res
        return res
    },
    err => {
        console.log(err);
    }
)
```

### 拦截器练习的完整代码
```
export const requestInter = (config) => {
    const instance = axios.create({
        baseURL: 'http://123.207.32.32:8000',
        method: 'get',
        timeout: 5000,
    })

    // request拦截器: axios实例.interceptors.request.use()
    instance.interceptors.request.use(
        config => {
            console.log(config);
            // config 必须被返回,因为拦截器将其拦截,我们做了相应操作后要返回config.
            return config
        },
        err => {
            console.log(err);
        },
    )

    // response拦截器: axios实例.interceptors.response.use()
    instance.interceptors.response.use(
        // 响应拦截器拦截网络响应后传递回来的结果对象(result)
        res => {
            console.log(res);
            res = res.data;
            // 同样需要返回被拦截的 res
            return res
        },
        err => {
            console.log(err);
        }
    )

    return instance(config)
}
```

## Part5: 网络请求结果封装
实际项目中, 通常对所有网络请求进行封装, 有以下几点好处:

1. 方便管理
2. 便于维护
3. 减少项目对第三方框架的依赖(例如axios)

项目中通常新建 `/network` 存储网络请求的封装, 不同网络请求方式进行不同的封装

**用函数封装网络请求并导出,项目文件以该函数为入口,传入各项目文件自定义的参数即可.**
函数封装的要点:
1. 封装的内容要有复用性, 一些特殊需求用参数代替, 期望调用者自定义传值, 增加代码灵活性
2. 函数内封装一些公共且固定的配置信息
3. **函数封装本质上就是通过函数包裹执行代码,以函数名为入口,以函数返回值为出口.**
4. **函数封装过程类似于模块化自定义实现**(开发者调用函数时,不需要知道函数内部操作,只需要配置相应的参数,接收返回值即可).

举例：
封装了一个基础地址为 http://123.207.32.32:8000 的axios实例，并将该实例向外界暴露
```
import axios from 'axios'

export const request = (config) => {
    const instance = axios.create({
        baseURL: 'http://123.207.32.32:8000',
        method: 'get',
        timeout: 5000,
    })

    instance.interceptors.response.use(
        res => res.data,
        err => console.log(err)
    )

    return instance(config)
}
```
此处又做了进一步的封装，将所有项目用到的自定义网络请求(上面是通用网络请求的封装)封装在一个文件内管理。
```
//引入了通用的网络请求
import {request} from './request'

export const getHomeMultidata = () => {
    // 通用请求的使用
    return request({
        url: '/home/multidata'
    })
}
```
