---
title: ES6篇 - Module 语法
date: 2021-04-20 09:40
tags: 
- JavaScript
- ES6
- 面试
categories: JavaScript
excerpt: 文章是以"阮一峰ES6入门"为参考而著的一篇学习总结, 请酌情参考;
---

请结合 [Module 的语法](https://es6.ruanyifeng.com/#docs/module) 一起阅读本文;

# Module 概述
在 ES6 之前, 最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器; 
**ES6 的模块功能**实现, 取代了 CommonJS 和 AMD 规范, 是**浏览器和服务器通用**的模块解决方案;
ES6 模块的设计思想: **静态化**; 即在编译阶段就确定模块间的依赖关系, 以及输入和输出的变量;

## CommonJS 运行时加载
```
// CommonJS模块
let { stat, exists, readfile } = require('fs');

// 等同于
let _fs = require('fs'); // 加载模块所有方法并生成一个对象
let stat = _fs.stat; // 从生成对象中取出方法
let exists = _fs.exists;
let readfile = _fs.readfile;
```
本质: 整体加载模块(加载模块的所有方法), 生成一个对象, 然后从该对象上获取属性和方法; 
缺点: 只能在运行时执行该语句才能得到模块对象, 无法在编译阶段做到"静态优化";

## ES6 模块静态加载
本质: 由一个模块输出模块接口, 另一模块接收该接口
优点: 在编译阶段就完成模块加载, 效率比 CommonJS 高; 基于编译时加载的特点, 可以进行类型检测等静态分析的功能;
缺点: 无法引用 ES6 模块本身


# Module
## export 关键字
模块是独立的文件, 模块内部的所有变量, 外部无法获取; export 关键字为外部读取模块内部的变量提供了方法;
作用: 用于规定模块的**对外接口**;
书写位置: 可出现在**模块顶层的任意位置**, 如果出现在块级作用域内则会报错, 因为在块级作用域内无法实现静态优化(块级作用域需要程序运行时访问);
```
//  写法一: 逐个抛出对外接口(变量/函数/类...)
export let firstName = 'Siri';
export let secondName = 'John';
export function sayHi() {};
export const call = function() {};
export class MyClass {};

// ==========================

// 写法二: 用 {} 包裹一组(变量/函数名/类名)抛出
let firstName = 'Siri';
let secondName = 'John';
export {firstName, secondName};

// ==========================

// 错误示例:
let firstName = 'Siri';
export firstName; // 报错, 抛出的实际是 'Siri' 值;
```
> export 向外抛出的是一个接口, 不能抛出一个具体值;
> ES6 模块抛出值的引用, CommonJS 抛出值的拷贝;

## as 关键字
作用: 重命名
```
let firstName = 'Siri';
let secondName = 'John';
export {
  firstName as fn, 
  secondName as sn,
  secondName as sc,
};
```
> 对外接口重命名可达到重复抛出的效果;

## ES6 export 特点
export 输出接口, 其与模块内变量一一对应, 外部代码调用该接口时, 可以通过该接口读取到模块内实时的值, 即接口动态绑定了模块内对应变量的值;
```
let count = 1;
exports.count = count; // <=> exports.count = 1;
```
CommonJS exports 输出的是值的缓存, 因此不存在动态更新;
> 从上例 CommonJS 输出可以看到, CommonJS 相当于创建了一个空 exports 对象, 并将抛出的值拷贝到该对象上, 然后整体抛出 exports 对象;

## import 关键字
使用 export 命令定义了模块的对外接口以后，其他 JS 文件就可以通过 import 命令加载这个模块;
```
import {firstName, secondName as sn} from './profile.js'
```
import 命令接受一对大括号，里面指定要从其他模块导入的变量名。大括号里面的变量名，必须与被导入模块（profile.js）对外接口的名称相同;
import from 后接收模块路径(相对路径, 绝对路径, 模块名), 当接收模块名时, 需要在 `package.json` 中做相应配置告诉 JavaScirpt 引擎默认的模块导入路径;
import 可以通过 as 关键字对接收的变量重命名;

* import 接收的变量都是**只读**的, 本质是因为 import 接收的是 export 抛出的接口, ES6 不允许修改模块的接口(地址);
``` 
import {a} from 'module';
a = {} // 报错, a 代表模块接口, 修改它相当于修改了接口地址指向, 导致无法正确读取接口动态绑定的模块变量值;

a.foo = {} // 不报错, a.foo 代表从 a 接口调用输出模块内的 foo 变量值, 因此可正常修改, 但建议凡是输入变量均当作只读属性处理, 避免因修改其他模块数据导致的错误;
```
* import 具有提升效果, 会提升至模块的头部首先执行, 本质是因为 import 命令在编译阶段执行, import 位于模块顶层即可;
```
foo();
import {foo} from 'module';

// 等价于
import {foo} from 'module'; // 编译阶段执行
foo(); // 运行阶段执行
```
* import 静态执行, 不能使用**表达式/变量/块级作用域**等运行时才能得到结果的语法结构;
```
// 下述 import 均报错

// 表达式
import {'f'+'oo'} from 'module';

// 变量
let myModule = 'module';
import {foo} from myModule;

// if 结构
if (true) {
  import {foo} from 'module';
}
```
* 多次执行**相同模块路径**的 import 语句, 最终只会**执行一次**, 不会执行多次;
```
import {foo} from 'module';
import {bar} from 'module';
// 上述操作被合并, 并只执行一次, 等价于:
import {foo, bar} from 'module';
```
```
import {foo} from 'module';
import {foo} from 'module';
import {foo} from 'module';
// 重复操作只执行一次, 等价于:
import {foo} from 'module';
```

## 模块整体加载
语法: 用 `*` 指定一个对象, 所有输出值都加载在该对象上;
```
import * as circle from './circle.js';

console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(14));
```
本质: 用 `*` 创建一个对象, 重命名为 `circle`, 将输出模块的**所有接口**都挂载(添加)到该对象上统一管理;
```
import * as circle from './circle';

// 下面两行都是不允许的
circle.foo = 'hello';
circle.area = function () {};
```
> 注意: 对象内存储的还是对外接口, 因此仍为**只读**属性, 调用接口时不能对其进行修改;

## export default
作用: 为模块指定默认输出, 其他模块加载该模块时，import命令可以为输出接口指定任意名字;
一个模块只能有一个默认输出，因此export default命令只能使用一次;
```
function add(x, y) {
  return x * y;
}
export {add as default}; // 等同于 export default add;

import {default as foo} from 'module'; // 等同于 import foo from 'module'
```
本质上，export default就是输出一个叫做 default 的变量或方法，然后系统允许你为它取任意名字;

# import()
静态 import 加载有利于提高编译器效率, 但也导致无法在运行时加载模块;
ES2020 提案引入了 `import()` 函数来支持动态加载模块(运行时执行);
1. `import()` 接收模块路径, 返回一个 Promise 对象(证明 import 是异步加载);
2. `import()` 可在任何地方使用, 不仅局限于模块, 非模块脚本也可以使用;
3. `import()` 类似于 CommonJS 的 `require()`, 区别在于前者是异步加载, 后者是同步加载;
4. `import()` 加载模块成功后, 加载的模块结果会作为一个对象(类似于`*`, 内部包含多个对外接口), 作为 then 方法的参数传入(可用对象解构获取输出接口); `import()` 与加载的模块间没有静态连接关系, 但是本质上仍是值的引用, 通过对象内包裹的接口可获取对应模块内最新的值; 
```
import('./myModule.js')
.then(({export1, export2}) => {
  // ...·
});
```
5. `import()` 的 default 输出接口, 通过 `xxx.default` 获得;
```
import('./myModule.js')
.then((module) => {
  console.log(module.default);
});
```
6. 用 `Promise.all()` 同时动态加载多个模块:
```
Promise.all([
  import('./module1.js'),
  import('./module2.js'),
  import('./module3.js'),
])
.then(([module1, module2, module3]) => {
   ···
});
```

## 使用场景
1. 按需加载
```
button.addEventListener('click', event => {
  import('./index.js')
  .then(module => {
    /* Success handling */
  })
  .catch(error => {
    /* Error handling */
  })
});
```
2. 条件加载
```
if (condition) {
  import('moduleA').then(...);
} else {
  import('moduleB').then(...);
}
```
3. 动态模块路径
```
import('./f' + 'oo.js').then(...);
```