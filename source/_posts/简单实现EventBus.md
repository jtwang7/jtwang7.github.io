---
title: 简单实现EventBus
date: 2021-04-02 09:58
tags: 
- JavaScript
- Vue
categories: JavaScript
excerpt: 用js简单实现Vue中的EventBus通信方式, 简要记录。
---

# EventBus 原理
通过维护类内的一个的受保护属性, 将`emit`发送的事件及参数存储, 并通过`on`监听事件并接收参数执行回调.
事件的存储和检索都通过 Map 来完成, Map 可以存储任何类型的键名, 非常适合检索工作.

# 实现
```
// 定义EventEmitter类
class EventEmitter {
  constructor() {
    this._events = new Map(); // 存储事件
    this._maxListnersNum = 10; // 设置最大监听数上限
  }
  // 发送事件 (同Vue.$emit())
  emit(event, ...args) {
    if (!this._events.get(event)) {
      this._events.set(event, args); // 将事件名和对应传递参数保存
    }
  }

  // 监听事件并出发回调 (同Vue.on())
  on(event, callback) {
    let payload = this._events.get(event); // 执行时触发,监听事件名是否存在
    if (payload && payload.length > 0) {
      callback.apply(this, payload); // 通过 call/apply 方法触发回调
    } else {
      callback.call(this, payload);
    }
  }

  // 移除事件监听
  off(event) {
    if(this._events.has(event)) {
      this._events.delete(event);
    } else {
      console.log('No Event');
    }
  }
}

module.exports = EventEmitter;
```

# 验证
```
const EventEmitter = require('./eventEmitter');

const eventEmitter = new EventEmitter();

eventEmitter.emit('sayHi', {name:'Siri',text:'Hi'});
eventEmitter.on('sayHi', function({name,text}){
  console.log(`${text} ${name}`);
})

eventEmitter.off('sayHi')
eventEmitter.on('sayHi', function(val){
  console.log(val);
})

// result
// Hi Siri
// undefined
```