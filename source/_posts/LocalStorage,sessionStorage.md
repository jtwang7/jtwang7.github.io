---
title: LocalStorage,sessionStorage
date: 2021-03-28 15:26
tags: JavaScript
categories: JavaScript
excerpt:  Web 存储对象 localStorage 和 sessionStorage, 允许我们在浏览器上保存键/值对。在页面刷新后（对于 sessionStorage）甚至浏览器完全重启（对于 localStorage）后，数据仍然保留在浏览器中。
---

# LocalStorage,sessionStorage
## LocalStorage,sessionStorage 与 Cookie 的区别
1. **Web存储对象(LocalStorage,sessionStorage)不会随每个请求被发送到服务器。可以保存更多数据**。大多数浏览器都允许保存至少 2MB 的数据（或更多），并且具有用于配置数据的设置。
2. **服务器无法通过 HTTP header 操纵存储对象**。对于Web存储对象的操作一切都是在 JavaScript 中完成的。
3. 存储对象绑定到源（域/协议/端口三者）。不同协议或子域 对应 不同的存储对象，它们之间无法访问彼此数据。

## LocalStorage,sessionStorage 存储对象的方法和属性
1. setItem(key, value) —— 存储键/值对。
2. getItem(key) —— 按照键获取值。
3. removeItem(key) —— 删除键及其对应的值。
4. clear() —— 删除所有数据。
5. key(index) —— 获取该索引下的键名。
6. length —— 存储的内容的长度。

## LocalStorage
### 特点：
1. 在**同源(协议/域/端口均相同)**的所有标签页和窗口之间**共享数据**。(url路径可以不同，即源下路径不同，也可以获取数据)
2. **数据不会过期**。它在浏览器重启甚至系统重启后仍然存在。

## sessionStorage
### 特点：
1. sessionStorage 的数据**只存在于当前浏览器标签页**。不同标签页存储数据不同。
2. 数据在页面刷新后仍然保留，但在关闭/重新打开浏览器标签页后不会被保留。

## 存储对象遍历
存储对象不可迭代(`for..of..`不适用)

### for(let i=0; i<localStorage.length; i++)
用最普通的for循环遍历：
```
for(let i = 0; i < localStorage.length; i++) {
  let key = localStorage.key(i);
  alert(`${key}: ${localStorage.getItem(key)}`);
}
```

### for(let key in localStorage)
用遍历对象键的方式：
缺点：遍历所有键，会输出 localStorage 或 sessionStorage 的内建字段
```
for(let key in localStorage) {
  alert(key); // 显示 getItem，setItem 和其他内建的东西
}
```

#### 通过 hasOwnProperty 或使用 Object.keys 过滤原型内建字段
**`.hasOwnProperty()`**
```
for(let key in localStorage) {
  if (!localStorage.hasOwnProperty(key)) {
    continue; // 跳过像 "setItem"，"getItem" 等这样的键
  }
  alert(`${key}: ${localStorage.getItem(key)}`);
}
```
**`Object.keys()`**
更佳的选择，不需要写额外判断条件，Object.keys()只返回属于对象的键，会忽略原型上的。
```
let keys = Object.keys(localStorage);
for(let key of keys) {
  alert(`${key}: ${localStorage.getItem(key)}`);
}
```

## 存储格式
Web存储对象(localStorage,sessionStorage)的键值对都必须是字符串，其内部会进行自动转化。
因此，对象类型的值存储要手动通过 `JSON.stringify()` 处理，以免出错。
或者可以对整个存储对象进行字符串化处理。

## Storage 事件
### 触发条件：
当 localStorage 或 sessionStorage 中的数据更新后，storage 事件就会触发。
**事件会在所有可访问到存储对象的 window 对象上触发，导致当前数据改变的 window 对象除外。**

### 应用：
**其允许同源的不同窗口交换消息**
常用于监听同源下不同窗口 LocalStorage 变化，当一个窗口数据更新时，另一个窗口及时进行反应。
```
// 等同于 window.addEventListener('storage', () => {})
window.onstorage = event => {
  if (event.key != 'now') return;
  alert(event.key + ':' + event.newValue + " at " + event.url);
};

localStorage.setItem('now', Date.now());
```

### 事件属性：
1. key —— 发生更改的数据的 key（如果调用的是 .clear() 方法，则为 null）。
2. oldValue —— 旧值（如果是新增数据，则为 null）。
3. newValue —— 新值（如果是删除数据，则为 null）。
4. url —— 发生数据更新的文档的 url。
5. storageArea —— 发生数据更新的 localStorage 或 sessionStorage 对象。