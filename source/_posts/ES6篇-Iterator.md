---
title: ES6篇 - Iterator
date: 2021-04-15 16:48
tags: 
- JavaScript
- ES6
- 面试
categories: JavaScript
excerpt: 文章是以"阮一峰ES6入门"为参考而著的一篇学习总结, 请酌情参考, 内容包括 Iterator 概念, Iterator API, 应用场景等;
---

# Iterator
参考文章: [ES6阮一峰 - Iterator 和 for...of 循环](https://es6.ruanyifeng.com/#docs/iterator)

## Iterator 概念
Iterator (遍历器**对象**) 是为各种数据结构(Array, Object, Map, Set, ...)提供的一个统一简便的**访问接口**;
Iterator 接口主要供 ES6 的**遍历**命令 for...of **消费**, 任何部署了 Iterator 接口的数据结构都可以完成遍历操作(程序可依次处理该数据结构的所有成员);
Iterator 能够将数据结构的成员按照某种次序进行**排序**;

***Iterator 为程序遍历各种数据结构提供了统一的方法, ES6 的 for...of 通过访问和消耗 Iterator 接口, 实现对数据结构内所有成员的遍历操作***

遍历 Iterator 的过程:
1. 创建一个**指针对象**, 指向当前数据结构的**起始位置**;
2. 调用指针对象的 **next** 方法, 将指针**指向**数据结构的第一个**成员**, **返回成员信息**; 每次调用 next 方法都会移动指针, 指向下一个成员, 并返回成员信息;
3. 不断调用指针对象的 next 方法, 直到**指向**数据结构**结束位置, 停止遍历**;

**注意:**
***Iterator 开始遍历时创建的指针对象, 其指向的起始位置不是数据结构的第一个成员;***
***Iterator 每一次调用 next 方法, 都会返回当前指向的数据结构成员的成员信息(一个包含 value 和 done 属性的对象: `{value: any, done: boolean}`)***
***Iterator 结束位置不是数据结构的最后一个成员, Iterator 遍历结束的信号是当 `done === true` 时;***

**Code**
要实现一个 Iterator 遍历器对象, 则主要实现三个要求:
* 返回一个对象
* 对象内包含 next 方法; **next() {...} 是一个无参函数**
* next 方法返回一个 `{value: any, done: boolean}` 对象

***Iterator 实现用到了闭包的思想, 当前遍历的索引变量被保留在了内存中, 没有随着 makeIterator 函数执行结束后上下文销毁而消失;***
```
function makeIterator(arrayLike) {
  const lens = arrayLike.length;
  let idx = 0;
  return {
    next() {
      // done: false 和 value: undefined 可以被省略;
      return idx < lens ? {
        value: arrayLike[idx++], done: false 
      } : {
        value: undefined, done: true
      }
    }
  }
}
```
Iterator 遍历操作实现:
```
const arrayLike = {
  '0': 'Hello',
  '1': 'World',
  'length': 2,
}
const iterator = makeIterator(arrayLike);
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
// done === true 后仍可以手动调用 next() 移动指针, 后续返回对象均为 { value: undefined, done: true }
console.log(iterator.next());
/**
 * { value: 'Hello', done: false }
 * { value: 'World', done: false }
 * { value: undefined, done: true }
 * { value: undefined, done: true }
 */
```

## 可迭代对象
上述我们实现了手动调用 Iterator 遍历器对象的 next 方法, 模拟对数据结构所有成员的遍历过程, ES6 通过 for...of 遍历数据结构时, 会**自动寻找 Iterator 接口**, 我们称**部署了 Iterator 接口的数据结构**为"可遍历对象(Iterable)"
**注: Iterator 遍历器对象是不可以被 for...of 遍历的, 因为它只是一个指针对象, 只有部署了它的数据结构才是 for...of 遍历的目标;**
ES6 规定, 默认的 Iterator 接口要部署在数据结构的 `Symbol.iterator` 属性上; `Symbol.iterator` 属性是当前数据结构默认的**遍历器生成函数**; `Symbol.iterator` 是一个表达式, 返回 Symbol 对象的 iterator 属性, 作为属性名时用方括号引用;

***for...of 只能遍历实现了 Symbol.iterator 属性部署的数据结构 (若目标原型链上具有 Symbol.iterator 属性也可被遍历); for...of 每次循环调用 next 方法后, 都会检查返回值 done 属性, 若遍历未结束 done === false, 则继续调用 next 方法循环, 若检查到 done === true, 则结束遍历, 且不会将 value 赋值给临时变量;***

**Code**
实现一个可迭代对象(可被遍历的数据结构), 需满足以下几点:
1. 数据结构内包含 `Symbol.iterator` 属性;
2. `Symbol.iterator` 属性值是一个**函数**, 返回一个**遍历器对象**(上述已实现)
```
const arrayLike = {
  '0': 'Hello',
  '1': 'World',
  'length': 2,
  [Symbol.iterator]() {
    const ctx = this;
    let idx = 0;
    return {
      next() {
        return idx < ctx.length ?
          { value: ctx[idx++], done: false } :
          { value: undefined, done: true }
      }
    }
  }
}
```

ES6 原生具备 Iterator 接口的数据结构:
1. Array
2. Map
3. Set
4. String
5. TypedArray
6. arguments
7. NodeList

***普通对象没有原生 Iterator 接口部署, 需要自己在 Symbol.iterator 属性上部署, 才能被 for...of 循环遍历; (对象 Object 之所以没有默认部署 Iterator 接口, 是因为对象属性遍历的顺序是不确定的, 需要开发者手动指定)***


## 遍历 Iterator 接口的场景
除了 for...of 循环外, 还有几个操作也要求数据结构部署 Iterator 接口;
1. 解构赋值
2. 扩展运算符
3. `yield*`
4. Array.from()
5. Map(), Set(), WeakMap(), WeakSet()
6. Promise.all()
7. Promise.race()


## for...of 循环
ES6 遍历所有数据结构的统一方法; 
前提: 数据结构具有 `Symbol.iterator` 属性, 即具有 iterator 接口;
本质: for...of 循环内部调用数据结构的 `Symbol.iterator` 方法, 其返回一个迭代器对象, 并执行 next 方法, next 方法会返回 `{value: any, done: boolean}` 对象, 检查是否 `done === true`, 若不是则将 value 值赋值给 for...of 临时变量, 若是则终止迭代, 且不执行赋值操作;

