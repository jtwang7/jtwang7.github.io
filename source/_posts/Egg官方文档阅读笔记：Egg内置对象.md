---
title: Egg官方文档阅读笔记：Egg.js内置基础对象
date: 2021-03-13 19:25
tags: Egg
categories: Egg
excerpt: Egg内置对象继承于Koa并进行了一定的拓展，内置对象无论是在Koa还是Egg中都十分重要，由于初学Egg又不具备一定的Koa基础，直接上手代码会云里雾里。于是乎，抽出了一定的时间阅读Egg的官方文档，大致了解了下Egg内的基础对象究竟是什么，关系是什么，并根据自己的思考做了相应的总结。
---


# Egg官方文档阅读笔记：Egg内置对象
## Egg.js 框架内的内置基础对象
1. Application
2. Context
3. Request
4. Response
5. Controller
6. Service
7. Helper
8. Config
9. Logger

1-4的内置对象继承于Koa，5-9为Egg.js框架扩展封装的一些对象
本文介绍常用的内置对象，其余内置对象可参考[Egg.js官方文档(框架内置基础对象)](https://eggjs.org/zh-cn/basics/objects.html)

## Application
Application 继承自 Koa.Application，是**全局应用对象**，其意味着：
**在一个应用中，只会实例化一个**，在它上面**可以挂载全局的方法和对象**，同时可以在插件或者应用中**扩展 Application 对象**。(全局/唯一/可操作)

### 获取方式
Application 对象几乎可以在编写应用时的**任何一个地方获取**
常用的获取方式：

1. export 暴露函数时获取 Application 对象
被框架**Loader加载**的文件(一般常用路径下的文件，例如Controller，Service，Schedule 等路径文件都被Loader加载)在**暴露函数**时，其函数被 Loader 调用，此时就会**传入 app 对象作为参数**：
`module.exports = app => {...}`
2. 通过继承类的实例访问 Application 对象
在继承于 Controller, Service 等基类的实例中，可以通过 `this.app` 访问到 Application 对象。
```
// app/controller/user.js
class UserController extends Controller {
  async fetch() {
    this.ctx.body = this.app.cache.get(this.ctx.query.id);
  }
};
```
3. 在 Context 对象上，可以通过 `ctx.app` 访问 Application 对象
```
// app/controller/user.js
class UserController extends Controller {
  async fetch() {
    this.ctx.body = this.ctx.app.cache.get(this.ctx.query.id);
  }
}
```

## Context
Context 继承自 Koa.Context，是一个**请求级别的对象**，其意味着：
框架将会在**每次发生用户请求时实例化一个 Context 对象**，对象**封装**这次用户请求的信息，同时提供(暴露)许多方法来获取请求参数或者设置响应信息。
此外，框架内的 Service 会全部挂载到 Context 实例上，一些插件也会将一些其他的方法和对象挂载到它上面（egg-sequelize 会将所有的 model 挂载在 Context 上）。

### 获取方式
Context 实例获取方式一般是在 Middleware, Controller 以及 Service 中。此外还有一些 Context 实例的获取(如在未发生请求时，匿名创建Context实例等)不做详述。

1. Controller 中获取 Context 实例
```
// app/controller/user.js
class UserController extends Controller {
  async fetch() {
    this.ctx.body = this.ctx.app.cache.get(this.ctx.query.id);
  }
}
```
2. Service 中获取 Context 实例 (同在 Controller 中获取)
3. Middleware 中获取 Context 实例 (同Koa框架在中间件中获取 Context 对象的方式)
```
// Koa v2
async function middleware(ctx, next) {
  // ctx is instance of Context
  console.log(ctx.query);
}
```

## Request & Response
Request 继承自 Koa.Request ，是一个请求级别的对象。封装了 Node.js 原生的 HTTP Request 对象，同时提供(暴露)了一些方法，用于获取 HTTP 请求常用参数。

Response 继承自 Koa.Response ，是一个请求级别的对象。封装了 Node.js 原生的 HTTP Response 对象，同时提供(暴露)了一些方法，用于设置 HTTP 响应。

### 获取方式
在 Context 实例上获取当前请求的 Request(ctx.request) 和 Response(ctx.response) 实例。(因为 Context 本身也是请求发生时被框架实例化的)
```
// app/controller/user.js
class UserController extends Controller {
  async fetch() {
    const { app, ctx } = this;
    const id = ctx.request.query.id;
    ctx.response.body = app.cache.get(id);
  }
}
```

## Controller
Controller 对象是 Egg.js 框架提供的一个基类，该基类有以下属性(注意基类中有ctx,app等，这也解释了ctx,app等实例可以从基类中获取的原因)：

1. ctx - 当前请求的 Context 实例。
2. app - 应用的 Application 实例。
3. config - 应用的配置。
4. service - 应用所有的 service。
5. logger - 为当前 controller 封装的 logger 对象。

### Controller 基类的继承与引用
**方法一：**
```
// 从 egg 上获取（推荐）
const Controller = require('egg').Controller;
class UserController extends Controller {
  // implement
}
module.exports = UserController;
```
方法二：
```
// 从 app 实例上获取
module.exports = app => {
  return class UserController extends app.Controller {
    // implement
  };
};
```

## Service
Service 对象是 Egg.js 框架提供的一个基类，其基类属性和访问方式和 Controller 基类一致：

1. ctx - 当前请求的 Context 实例。
2. app - 应用的 Application 实例。
3. config - 应用的配置。
4. service - 应用所有的 service。
5. logger - 为当前 controller 封装的 logger 对象。

### Service 基类的继承与引用
**方式一：**
```
// 从 egg 上获取（推荐）
const Service = require('egg').Service;
class UserService extends Service {
  // implement
}
module.exports = UserService;
```
方式二：
```
// 从 app 实例上获取
module.exports = app => {
  return class UserService extends app.Service {
    // implement
  };
};
```

## Helper
Helper 作用: 将一些可复用的函数抽离在 helper.js 里面成为一个独立的函数。
Helper 自身是一个类，有和 Controller 基类一样的属性，它也会在每次请求时进行实例化，因此 **Helper 上的所有函数也能获取到当前请求相关的上下文信息**。

### 获取方式
在 Context 的实例上获取到当前请求的 Helper(ctx.helper) 实例。
```
// app/controller/user.js
class UserController extends Controller {
  async fetch() {
    const { app, ctx } = this;
    const id = ctx.query.id;
    const user = app.cache.get(id);
    ctx.body = ctx.helper.formatUser(user);
  }
}
```

### helper.js 中方法定义
通过框架扩展的形式来自定义 helper 方法，例如上述`ctx.helper.formatUser()`
框架拓展：框架会把 `app/extend/helper.js` 中定义的对象与内置 helper 的 prototype 对象进行合并，在处理请求时会基于扩展后的 prototype 生成 helper 对象。(即将自定义 helper 对象合并到框架内置 helper 对象内，生成新的扩展 helper 对象)
```
// app/extend/helper.js
module.exports = {
  formatUser(user) {
    return only(user, [ 'name', 'phone' ]);
  }
};
```
注意：Helper 是个类，我们抽离的可复用方法是通过`app/extend/helper.js` 中的模块打包为对象导出后，遵循框架扩展的原则与内置helper对象合并得到的。需要通过 helper 对象访问内部自定义方法。


## 总结
内置对象间的引用看似复杂，实际上都大同小异：
从**获取方式**上对它们做了相应的总结

**通过类内this获取：**

1. 对于从Koa继承的对象(app,ctx)，Egg的扩展内置对象均对其做了封装，即在类内都能直接通过this获取。
2. 从Koa继承的(request,response)请求级别对象，本身就被Koa的(ctx)对象调用，因此在Egg的其他内置对象类也可以通过ctx调用request,response。

**各内置对象间的调用：**

1. ctx 可以通过 `ctx.app` 返回 app 对象的引用
2. ctx 上可挂载 service 和一些插件即配置信息

**其他方式：**

1. exports暴露函数时传参：所有被Loader加载的文件暴露函数时，均会默认传入 app 对象作为参数
2. 中间件定义时，默认传入ctx对象作为参数

总体来说，app(全局对象) 和 ctx(每次请求均被实例化) 几乎是全局存在的，在其他内置对象内可通过this调用，app 可被 ctx 返回，app 可在exports函数时作为参数传入，ctx 可作为中间件参数传入。request/response 通过 ctx 调用。Controller 和 Service 均包含五大属性(ctx,app,config,service,logger)，此外Service全部会被挂载在 ctx 对象上，通过 ctx 对象也可以访问 service。