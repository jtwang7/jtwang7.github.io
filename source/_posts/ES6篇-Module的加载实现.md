---
title: ES6篇 - Module 的加载实现
date: 2021-04-20 11:20
tags: 
- JavaScript
- ES6
- 面试
categories: JavaScript
excerpt: 文章是以"阮一峰ES6入门"为参考而著的一篇学习总结, 请酌情参考;
---

请结合 [Module 的加载实现](https://es6.ruanyifeng.com/#docs/module-loader) 一起阅读本文;

# 浏览器加载
## 浏览器加载 JS 脚本
浏览器通过 `<script type="application/javascript"></script>` 标签加载脚本, 由于浏览器脚本的默认语言是 JavaScript, 因此 `type="application/javascript"` 可以省略;
```
<!-- 页面内嵌的脚本 -->
<script type="application/javascript">
  // module code
</script>

<!-- 外部脚本 -->
<script type="application/javascript" src="path/to/myModule.js"></script>

<!-- 省略 type -->
<script src="path/to/myModule.js"></script>
```

默认情况下, 浏览器同步加载脚本;
缺点: 容易造成浏览器阻塞, 在脚本执行完成前浏览器整体处于无法响应状态;

浏览器异步加载脚本的两种方案:
`<script>` 标签打开 **defer** 或 **async** 属性, 脚本就会异步加载;
```
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
```

defer 和 async 区别:
1. 执行时刻不同: defer 要等到整个页面在内存中正常渲染结束(DOM 结构完全生成，以及其他脚本执行完成), 才会执行; async一旦下载完, 渲染引擎就会中断渲染, 执行这个脚本以后, 再继续渲染; 一句话, defer是“渲染完再执行”, async是“下载完就执行”;
2. 脚本加载顺序不同: 如果有多个 defer 脚本, 会按照它们在页面出现的顺序加载; 多个 async 脚本是不能保证加载顺序的;

## 浏览器加载 ES6 模块
通过 `<script type="module"></script>` 标签加载模块;
浏览器对于带有 `type="module"` 的 `<script>`, 都是异步加载; 即**等到整个页面渲染完, 再执行模块脚本**, 等同于打开了 `<script>` 标签的 defer 属性; 如果网页有多个 `<script type="module">`, 模块将按照在页面出现的顺序依次执行;
> 浏览器异步加载 ES6 模块时等价于默认开启 defer 属性, 开发者也可显式开启 async 属性加载模块, 其会覆盖 defer 属性, 在模块加载完成时就中断渲染立即执行, 执行完成后再恢复渲染, 且模块加载时不再保证执行顺序;

# Node.js 模块加载
JavaScript 现在有两种模块: 一种是 ES6 模块, 简称 ESM; 另一种是 CommonJS 模块, 简称 CJS;
CommonJS 模块是 Node.js 专用的, 与 ES6 模块不兼容; 它们采用不同的加载方案, 从 Node.js v13.2 版本开始, Node.js 已经默认打开了 ES6 模块支持;

Node.js 碰到 `.mjs` 文件总是以 ES6 模块加载，`.cjs` 文件总是以 CommonJS 模块加载，`.js` 文件的加载取决于 `package.json` 里面 type 字段的设置;
> type 字段配置 "module" 则已 ES6 模块加载 `.js` 文件; 若不显式配置或将 type 字段配置 "commonjs", 则已 CommonJS 模块加载 `.js` 文件;
> ES6 模块与 CommonJS 模块尽量不要混用(可以相互实现加载, 但尽量避免), 不能**直接**用 import 命令加载 `.cjs` 文件, 会报错, 只有 require 命令才可以加载 `.cjs` 文件; 反过来, `.mjs` 文件里面也不能**直接**使用 require 命令, 必须使用 import;

## package.json 模块加载入口配置
`package.json` 文件有两个字段可以指定模块的入口文件: main 和 exports;
### main
```
{
  "type": "module",
  "main": "./src/index.js"
}
```
```
import { something } from "es";
// 实际加载: ./node_modules/es/src/index.js
```
配置 main 字段后, 加载模块时, Node.js 就会选择相应的方式(此处为 ES6 模块)到 `./node_modules` 目录下面, 寻找指定模块, 然后根据该模块 `package.json` 的 `main` 字段去执行入口文件;

### exports
main 字段一般用于简单的模块加载配置;
exports 字段的优先级高于 main 字段, 相较于 main 配置项更加细致复杂;
**1)** `package.json` 文件的 exports 字段可以指定**脚本或子目录的别名**;
脚本别名:
```
{
  "type": "module",
  "exports": {
    "./submodule": "./src/submodule.js"
  }
}

import submodule from "es/submodule";
// 实际加载 ./node_modules/es/src/submodule.js;
```

子目录别名:
```
{
  "type": "module",
  "exports": {
    "./features/": "./src/features/"
  }
}

import submodule from "es/features/index.js";
// 实际加载 ./node_modules/es/src/features/index.js;
```

**2)** exports 主入口别名
exports 字段的别名如果是 `.`, 就代表模块的主入口, 优先级高于 main 字段, 并且可以直接简写成 exports 字段的值;
```
{
  "exports": {
    ".": "./main.js"
  }
}

// 简写
{
  "exports": "./main.js"
}
```

# ES6 模块与 CommonJS 模块的差异
> CommonJS: `module.exports = {...}`, `exports.xxx = yyy`; 可以认为 CommonJS 先创建了一个 exports 空对象, 挂载属性和方法的拷贝, 向外抛出该 exports 对象;
> ES6: `export 接口`, `export default 值`; export 抛出对外接口, export default 将值赋予 default 后抛出 default 接口;

1. CommonJS 模块输出**值的拷贝**, ES6 模块输出**值的引用**;
2. CommonJS 模块是**运行时加载**, ES6 模块是**编译时输出接口**;
3. CommonJS 模块的 `require()` 是**同步加载模块**, ES6 模块的 import 命令是**异步加载**, 有一个独立的模块依赖的解析阶段;

## 差异一
CommonJS 输出值的拷贝, 模块一旦输出该值, 后续模块内的变化就不会影响到这个输出的值;
ES6 模块的运行机制与 CommonJS 不同: JS 引擎**对脚本静态分析**的时候, 遇到模块加载命令 **import**, 就会**生成一个只读引用**。等到脚本**真正执行时**，再根据这个只读引用，到被加载的那个模块里面去**取值**; 每次读取引用时, 都会去相应的输出模块内取值, 因此 ES6 模块是动态引用, 并且不会缓存值, 模块里面的变量绑定其所在的模块;
> 由于 ES6 输出的是引用, 真实值始终定义在输出模块内, 因此不同脚本读取 export 的对外接口, JS 引擎总会去输出模块内取值, export 输出的实际上是同一个值;

## 差异二
产生差异二的主要原因: CommonJS 加载的是一个**对象**(即 module.exports 属性), 该对象只有在**脚本运行**完才会生成; 而 ES6 模块不是对象，它的对外接口只是一种**静态定义**，在代码**静态解析阶段**就会生成;


# CommonJS 模块加载原理
CommonJS 的一个模块, 就是一个脚本文件; **require 命令第一次加载脚本, 就会执行整个脚本, 然后在内存生成一个对象**:
```
// Node 内部加载模块后生成的一个对象: 包含 id (模块名); exports (模块输出对象); loaded (boolean | 表示模块脚本是否执行完毕); ...
{
  id: '...',
  exports: { ... },
  loaded: true,
  ...
}
```
以后需要用到这个模块的时候, 就会**到 exports 属性上取值**; 即使再次执行 require 命令, 也不会再次执行该模块, 而是到缓存之中取值;
CommonJS 模块无论加载多少次, 都只会在第一次加载时运行一次, 以后再加载, 就返回第一次运行的结果, 除非手动清除系统缓存;

# CommonJS 处理循环加载
CommonJS 模块的重要特性是加载时执行, 结合 CommonJS 的加载原理可知, 脚本代码在 require 的时候, 就会**全部**执行, 一旦出现某个模块被"循环加载", 就只输出已经执行的部分, 还未执行的部分不会输出;
`a.js`
```
exports.done = false; // 1
var b = require('./b.js'); // 2
console.log('在 a.js 之中，b.done = %j', b.done); // 8 输出 true
exports.done = true; // 9
console.log('a.js 执行完毕'); // 10
```

`b.js`
```
exports.done = false; // 3
var a = require('./a.js'); // 4
console.log('在 b.js 之中，a.done = %j', a.done); // 5 输出 false
exports.done = true; // 6
console.log('b.js 执行完毕'); // 7
```

`main.js`
```
let a = require('./a.js');
let b = require('./b.js');
```
执行 `main.js`, 首先整体加载 `a.js` 脚本; `a.js` 脚本运行过程中, 碰到 `require('./b.js')` 时, 会去整体加载 `b.js` 脚本, (此时 a 模块中生成的对象{id:..., exports:{...}, loaded: false}中, exports 属性只有 `exports.done = false`), 因此执行 `b.js` 脚本时, 循环加载 a 模块, 会从已生成的对象中取值, 该值就是 a 模块在暂停执行前已执行的部分; 随后将 b 模块执行完毕, 执行权交还给 a 模块(此时 a 模块生成对象的 loaded 为 false), 确保 a 模块执行完毕;
> 循环加载很好体现了 CommonJS 模块的加载机制: 同步执行; 模块生成对象; 完全执行;

# ES6 模块加载原理
JS 引擎**对脚本静态分析**的时候, 遇到模块加载命令 **import**, 就会**生成一个只读引用**。等到脚本**真正执行时**，再根据这个只读引用，到被加载的那个模块里面去**取值**; 每次读取引用时, 都会去相应的输出模块内取值, 因此 ES6 模块是动态引用, 并且不会缓存值, 模块里面的变量绑定其所在的模块;
`a.mjs`
```
import {bar} from './b';
console.log('a.mjs');
console.log(bar);
export let foo = 'foo';
```

`b.mjs`
```
import {foo} from './a';
console.log('b.mjs');
console.log(foo);
export let bar = 'bar';
```
执行 `a.mjs` , 引擎发现它加载了 `b.mjs`, 因此会优先执行 `b.mjs`, 然后再执行 `a.mjs`; 接着, 执行b.mjs的时候, 从 `a.mjs` 输入了 foo 接口, 这时不会去执行 `a.mjs`, 而是认为这个接口已经存在了(即生成了一个只读引用), 继续往下执行; 执行到第三行 `console.log(foo)` 的时候, 才发现这个接口根本没定义(脚本真正执行遇到该只读引用时才会去取值), 因此报错;
> 个人理解: ES6 模块在编译阶段就已经完成, 其在静态分析脚本时, 生成了 bar 和 foo 两个接口的只读引用; 当脚本真正运行到 foo 引用时, 才回去取值, 发现此时接口对应的模块值并没有生成, 因此报错;

稍微修改一下:
`a.mjs`
```
import {bar} from './b';
console.log('a.mjs');
console.log(bar());
function foo() { return 'foo' }
export {foo};
```

`b.mjs`
```
import {foo} from './a';
console.log('b.mjs');
console.log(foo());
function bar() { return 'bar' }
export {bar};
```
将 foo 声明成一个变量, 而非 let 声明的变量, 脚本便可正常运行; 这是因为在执行阶段, 创建变量时, 函数声明会被提升至顶部, 因此在执行`import {bar} from './b'`时，函数`foo`就已经有定义了，所以`b.mjs`加载的时候不会报错;