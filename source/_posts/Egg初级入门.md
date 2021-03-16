---
title: 初识 Egg.js
date: 2021-03-13 17:55
tags: 
- Egg
- MySQL
categories: Egg
excerpt: 项目需要前后端交互数据，仅需要用到接口制作，数据库连接等基本操作，故对Egg做了一点了解，根据其他教程写了这篇文章，记录了Egg操作数据库最基本的流程，可简单实现当前项目需求。
---

# Egg初级入门
参考文章：

1. [Egg.js 基本使用](https://juejin.cn/post/6844903718106693646)
2. [Egg.js 源码分析-项目启动](https://juejin.cn/post/6844903716777099278)
3. [Egg.js 项目结构](https://juejin.cn/post/6844904081689952269)

## Egg.js 安装
### 方法一
根据[Egg.js官方文档](https://eggjs.org/zh-cn/intro/quickstart.html)快速创建项目：
```
$ mkdir 自定义项目名 && cd 自定义项目名
$ npm init egg --type=simple
$ npm i
```
项目创建完成后可进行初步测试，检测是否安装成功：
```
$ npm run dev
$ open http://localhost:7001
```

### 方法二
通过`egg-init`脚手架初始化项目
```
$ npm i egg-init -g
$ egg-init 自定义项目名 --type=simple
$ cd 自定义项目名
$ npm i
$ npm run dev
```

## Egg.js 项目目录结构
```
egg-project
├── package.json
├── app.js (可选)
├── agent.js (可选)
├── app
│   ├── router.js // 配置路由
│   ├── controller // 控制器
│        └── home.js
│   ├── service (可选) // 服务
│        └── user.js
│   ├── middleware (可选) // 中间件
│        └── response_time.js
│   ├── schedule (可选) // 用于定时任务
│        └── my_task.js
│   ├── public (可选) // 静态资源文件
│        └── reset.css
│   ├── view (可选) // 模板视图
│        └── home.tpl
│   └── extend (可选) // 集成功能插件
│       ├── helper.js (可选)
│       ├── request.js (可选)
│       ├── response.js (可选)
│       ├── context.js (可选)
│       ├── application.js (可选)
│       └── agent.js (可选)
├── config // 必要的核心配置
│   ├── plugin.js // 插件配置
│   ├── config.default.js // 默认基础配置
│   ├── config.prod.js // 生产环境
│   ├── config.test.js (可选) // 测试配置
│   ├── config.local.js (可选) // 本地配置
│   └── config.unittest.js (可选) // 待定 
└── test // 测试需要
    ├── middleware
        └── response_time.test.js
    └── controller
        └── home.test.js
```

`app/router.js` 用于配置 URL 路由规则
`app/controller/**` 用于解析用户的输入，处理后返回相应的结果(管理服务器与用户交互行为)
`app/service/**` 用于编写业务逻辑层(操作数据库)
`app/middleware/**` 用于编写中间件
`app/public/**` 用于放置静态资源
`app/extend/**` 用于框架的扩展
`config/config.{env}.js` 用于编写配置文件(配置环境)
`config/plugin.js` 用于配置需要加载的插件
`test/**` 用于单元测试
`app.js` 和 `agent.js` 用于自定义启动时的初始化工作

## Egg 路由配置
Egg Router: 描述请求 URL 和具体承担执行动作的 Controller 的对应关系。(一个route对应Controller中的执行方法)
Egg 框架约定了 `app/router.js` 文件用于**统一所有路由规则**。通过统一的配置，可以避免路由规则逻辑散落在多个地方，从而出现未知的冲突，集中在一起我们可以更方便的来查看全局的路由规则。

### Router 定义和使用
#### app/router.js 
定义 URL 路由规则
```
// app/router.js
module.exports = app => {
    const { router, controller } = app;
    router.get('/user/:id', controller.user.info);
}
```
`controller.user.info`是Router回调函数所指向的app的controller下面的对象。该路径可以解析为：`controller` 是`app`的一个属性对象， eggjs 会在启动的时候调用`this.loadController();`方法，去加载整个应用`app/controller`文件下的所有的js 文件， 会将文件名作为属性名称，挂载在`app.controller` 对象上，分析 `/controller` 目录下js文件，其都将继承类作为借口export(暴露)出来了，因此，可以通过**类.方法名**的方式使用类内方法了，连在一起就写为了形如 `controller.user.findAll` 的方式来引用Controller 下面的方法了。

#### app/controller/**
在 controller 目录下实现对应的 Controller 方法
```
// app/controller/user.js
class UserController extends Controller {
    async info() {
        const {ctx} = this;
        ctx.body = {
            name: `hello ${ctx.params.id}`,
        }
    }
}

module.exports = UserController;
```

定义完路由规则和对应方法后，我们就算完成了一个最简单的Router定义。效果：用户向`/user/xxx`执行 GET 请求时，user.js 内的相应方法就会执行。

#### 组织和管理路由映射
项目中会涉及很多路由映射，若全写在 `app/router.js` 中会难以管理。我们可以根据需求做如下拆分：
```
// app/router.js
module.exports = app => {
    // require()引入函数，接收参数。因为/router/xx.js内不自带Application对象，因此通过参数传递
    require('./router/news')(app);
    require('./router/admin')(app);
}

// app/router/news.js
module.exports = app => {
    app.router.get('/news/list', app.controller.news.list);
    app.router.get('/news/detail', app.controller.news.detail);
}

// app/router/admin.js
module.exports = app => {
    app.router.get('/admin/user', app.controller.admin.user);
    app.router.get('/admin/log', app.controller.admin.log);
}
```

### Service 操作数据库
若你实现好了上述代码，则已经开发好了Router 和Controller , 但是在我们的controller 中，都是静态的内容,例如上述代码中只给`ctx.body`放入了一个对象。
在项目中我们需要跟数据库交互，实现前后端分离，我们一般将跟数据库交互的内容，都放在Service 层。Service 的目录一般为`app/service`，在该路径下我们可以创建多个 js 文件管理不同的数据库操作。
#### 定义 Service
```
// app/service/user.js

// 引入egg内置的Service对象
const Service = require('egg').Service;

// 继承Service类
class UserService extends Service {
  async find(uid) {
    const user = await this.ctx.db.query('select * from user where uid = ?', uid);
    return user;
  }
}

module.exports = UserService;
```

#### 在 Controller 中使用
```
// app/controller/user.js
class UserController extends Controller {
    async info() {
        const {ctx} = this;
        ctx.body = this.service.user.find();
    }
}

module.exports = UserController;
```
在Controller中调用Service类内的方法和路由调用Controller类内方法类似：`this.service.home.index();`
注：此处也可以用 `this.ctx.service` 代替 `this.service` ，两者是等价的

### egg-mysql 插件访问 MySQL 数据库
Egg.js 提供了 egg-mysql 插件来访问 MySQL 数据库。
**插件安装：**
`$ npm i --save egg-mysql`
**插件配置：**
```
// config/plugin.js
exports.mysql = {
  enable: true,
  package: 'egg-mysql',
};
```
在当前版本中，`plugin.js`文件统一以对象导出，因此可以写为：
```
module.exports = {
  mysql: {
    enable: true,
    package: 'egg-mysql',
  }
};
```
**环境配置(数据库连接信息)：**
单数据源(只访问一个MySQL数据库实例)
```
// config/config.default.js
exports.mysql = {
  // 单数据库信息配置
  client: {
    // host
    host: 'mysql.com',
    // 端口号
    port: '3306',
    // 用户名
    user: 'test_user',
    // 密码
    password: 'test_password',
    // 数据库名
    database: 'test',
  },
  // 是否加载到 app 上，默认开启
  app: true,
  // 是否加载到 agent 上，默认关闭
  agent: false,
};
```
当前 config.default.js 配置文件中，统一用 `module.exports` 导出对象，因此也可以写在 config 中：
```
module.exports = appInfo => {
  const config = exports = {
    mysql: {
      client: {
        // host
        host: 'localhost',
        // 端口号
        port: '3306',
        // 用户名
        user: 'root',
        // 密码
        password: 'root',
        // 数据库名
        database: 'trajsystem',
      },
      // 是否加载到 app 上，默认开启
      app: true,
      // 是否加载到 agent 上，默认关闭
      agent: false,
    }
  };

  // use for cookie sign key, should change to your own and keep security
  config.keys = appInfo.name + '_1615607271496_3440';

  // add your middleware config here
  config.middleware = [];

  // add your user config here
  const userConfig = {
    // myAppName: 'egg',
  };

  return {
    ...config,
    ...userConfig,
  };
};
```
注意此处的写法，`=`从右向左赋值，因此配置对象首先赋值给`exports(exports={mysql:{...}, ...})`，之后 config 在获得 exports 对象。在最后返回值的时候，用了ES6的展开语法`...`将 config 展开后抛出。按照我的理解，config 展开后应该抛出形如`exports.mysql`的形式。因此上述写法和之前将环境配置单独导出类似。

**单实例使用方式：** `await app.mysql.query(sql, values);`

多数据源(应用需要访问多个MySQL数据源)
```
exports.mysql = {
  clients: {
    // clientId, 获取client实例，需要通过 app.mysql.get('clientId') 获取
    db1: {
      // host
      host: 'mysql.com',
      // 端口号
      port: '3306',
      // 用户名
      user: 'test_user',
      // 密码
      password: 'test_password',
      // 数据库名
      database: 'test',
    },
    db2: {
      // host
      host: 'mysql2.com',
      // 端口号
      port: '3307',
      // 用户名
      user: 'test_user',
      // 密码
      password: 'test_password',
      // 数据库名
      database: 'test',
    },
    // ...
  },
  // 所有数据库配置的默认值
  default: {

  },

  // 是否加载到 app 上，默认开启
  app: true,
  // 是否加载到 agent 上，默认关闭
  agent: false,
};
```
**多实例使用方式：**
```
cosnt client1 = app.mysql.get('db1');
await client1.query(sql, values);

const client2 = app.mysql.get('db2');
await client2.query(sql, values);
```

## 总结
至此，我们完成了由Egg.js制作接口，连接数据库，并实现接口的暴露以及前后端通过接口实现数据库交互的一系列简单的操作。
现总结如下：

1. 定义 Router , 在 `app/router.js` 中构建路由映射
2. 定义 Controller，在 `app/controller` 目录下，编写路由跳转后需要执行的方法。
3. 安装 egg-mysql 插件，在 `config/plugin.js` 文件中配置插件，在环境中配置数据库的连接信息等
4. 定义 Service，在 `app/service` 目录下维护一些与数据库有关的操作，例如 CRUD 语句等。
5. 将 Service 中的继承类暴露，并在 Controller 中调用其内部方法。

整体可概括为： Router -- Controller -- Service(MySQL)