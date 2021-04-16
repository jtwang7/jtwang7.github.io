---
title: ES6篇 - Set & Map
date: 2021-04-16 11:53
tags: 
- JavaScript
- ES6
- 面试
categories: JavaScript
excerpt: 文章是以"阮一峰ES6入门"为参考而著的一篇学习总结, 请酌情参考;
---

# Set
成员值唯一的数据结构; 通过 Set() 构造函数生成实例;
Set() 构造函数可接收一个**数组(或具有 iterator 接口的其他数据结构)**作为参数进行初始化;
```
const set = new Set([1,2,2,3]) // Set { 1, 2, 3 }

const arr = {
  '0': 1,
  '1': 2,
  '2': 2,
  '3': 3,
  'length': 4,
  [Symbol.iterator]() {
    let idx = 0;
    const ctx = this;
    return {
      next() {
        return idx < ctx.length ?
          { value: ctx[idx++] } :
          { value: undefined, done: true }
      }
    }
  }
}

const set = new Set(arr); // Set { 1, 2, 3 }
```
Set() 内部判断两个值是否相同, 使用 **"Same-value-zero equality"** 算法, 其类似于严格运算符, 与 "===" 主要区别在于, **"Same-value-zero equality" 算法认为 NAN 等于自身**, "===" 则认为 NAN 不等于自身;

## Set 实例的属性
1. `Set.prototype.constructor`: 构造函数属性, 指向 Set();
2. `Set.prototype.size`: 返回 Set 实例的成员总数;

```
const set = new Set([1,2,2])
console.log(Set.prototype.constructor === Set); // true
console.log(set.size); // 2
```

## Set 实例的方法
### Set 实例的操作方法
1. `Set.prototype.add(value)`: **添加**某个值, **返回 Set 结构本身(可链式调用)**;
2. `Set.prototype.delete(value)`: **删除**某个值, **返回一个布尔值**, 表示删除是否成功;
3. `Set.prototype.has(value)`: **返回一个布尔值**, 表示该值是否为Set的成员;
4. `Set.prototype.clear()`: **清除**所有成员, 没有返回值;

```
const set = new Set();

set.add(1).add(3).add(1).add(5);
console.log(set.add(6)); // Set {1, 3, 5, 6}
console.log(set.delete(6)); // true
console.log(set.has(6)); // false
console.log(set.clear()); // undefined
console.log(set); // Set {}
```

### Set 实例的遍历方法
1. `Set.prototype.keys()`: 返回键名的**遍历器**
2. `Set.prototype.values()`: 返回键值的遍历器
3. `Set.prototype.entries()`: 返回键值对的遍历器
4. `Set.prototype.forEach()`: 使用回调函数遍历每个成员

**Set 实例的遍历顺序 = 成员添加顺序**, 我们可以使用 Set 作为保存回调函数的列表, 利用该特性可以保证回调函数按照添加的顺序调用;
**Set 结构键名和键值是同一个值**
Set 实例默认可被遍历, 是因为 Set 实例的 `[Symbol.iterator]` 为 values 遍历器生成函数; 这意味着可以省略 values 方法, 直接遍历 Set 实例; `console.log(Set.prototype[Symbol.iterator]) // [Function: values]`

## Set 应用
* 数组去重 (Set 实例 iterator 接口为 values(), 可被遍历)
```
const arr = [1,2,2,3,3,4];
const unique = [...new Set(arr)] // ... 内部机制为 for...of
const unique2 = Array.from(new Set(arr))
```
* 并集; 交集; 差集
```
const arr1 = [1, 2, 3, 3]
const arr2 = [3, 4, 5]
// 并集
const union = [...new Set([...arr1, ...arr2])]
console.log(union);

// 交集
const intersect = [...new Set(arr1.filter(item => arr2.includes(item)))]
console.log(intersect);

// 差集
const difference = [...new Set(arr1.filter(item => !arr2.includes(item)))]
console.log(difference);
```

# WeakSet
WeakSet 与 Set 类似, 是不重复值的集合, 存在两点区别:
1. WeakSet **成员只能是对象**, 不能是其他类型的值;
2. WeakSet 中的对象都是**弱引用**, 即垃圾回收机制不考虑 WeakSet 对该对象的引用; 如果其他对象都不再引用该对象, 那么垃圾回收机制会自动回收该对象所占用的内存, 不考虑该对象还存在于 WeakSet 之中;

语法:
接收一个数组(任何具有 Iterator 接口的对象)作为参数, WeakSet() 会遍历该参数对象, 并将其所有成员作为 WeakSet 实例的成员
```
const ws = new WeakSet([[1,2], [3,4]]); // WeakSet {[1, 2], [3, 4]}
```
***注意: WeakSet 成员只能是对象类型;***
```
const ws = new WeakSet([1, 2]) // Uncaught TypeError: Invalid value used in weak set(…)
```

## WeakSet 属性和方法
1. WeakSet 没有 size 属性;
2. WeakSet 没有遍历方法;
原因: WeakSet 成员都是弱引用, 随时可能消失, 遍历机制无法保证成员的存在; WeakSet 的成员是不适合引用的, WeakSet 内部有多少个成员, 取决于垃圾回收机制有没有运行, 运行前后很可能成员个数是不一样的, 而垃圾回收机制何时运行是不可预测的, ES6 规定 WeakSet 不可被遍历;
3. WeakSet 没有 clear() 方法, 即无法清空;

## WeakSet 应用场景
临时存放一组对象, 常用于存储 DOM 节点, 不用担心节点在文档中被移除时, 引发的内存泄漏问题; (用 Set 结构存储, DOM 节点移除后, Set 仍对 DOM 保持引用, 因此会引发内存泄漏)


# Map
传统 Object 对象只能用字符串作为键名; ES6 为解决这一限制, 提供了 Map 数据结构, 其本质上也是键值对集合(Hash 结构), 但 Map 键的范围不局限于字符串, 可以是任何原始类型的值, 是一种更加完善的 Hash 结构实现;
Map 实例通过 Map() 构造函数创建, 其接受一个数组(任何具有 Iterator 接口, 且每个成员都是一个双元素的数组的数据结构, 例如 Set 实例)作为参数, 该数组成员是一个个表示键值对的数组;
```
const map = new Map([
  ['name', 'wang'],
  ['age', 18],
  [{
    'eat': true,
    'sleep': true,
    'play': false,
  }, true],
])

/*
Map {
  'name' => 'wang',
  'age' => 18,
  { eat: true, sleep: true, play: false } => true
}
*/
```

## Map 实例的属性和方法
### 属性
1. `Map.prototype.constructor`: 构造函数属性, 指向 Map();
2. `Map.prototype.size`: 返回 Map 实例的成员总数;

### Map 实例的操作方法
1. `Map.prototype.set(key, value)`: 设置键名 key 对应的键值为 value, 然后**返回整个 Map 结构(链式)**; 如果key已经有值, 则键值会被更新, 否则就新生成该键(通过 "Same-value-zero equality" 算法判断是否存在相同的 key 值, Map 实例的 key 若为引用类型, 则保存的是引用地址, 若数据内容相同但地址不同, 则仍被视为不同的 key);
2. `Map.prototype.get(key)`: 读取 key 对应的键值, 如果找不到 key, 返回 undefined;
3. `Map.prototype.has(key)`: **返回一个布尔值**, 表示某个键是否在当前 Map 对象之中;
4. `Map.prototype.delete(key)`: 删除某个键, 返回true; 如果删除失败, 返回false;
5. `Map.prototype.clear()`: **清除**所有成员, 没有返回值;

### Map 实例的遍历方法
1. `Map.prototype.keys()`: 返回键名的**遍历器**
2. `Map.prototype.values()`: 返回键值的遍历器
3. `Map.prototype.entries()`: 返回键值对的遍历器
4. `Map.prototype.forEach()`: 使用回调函数遍历每个成员

**同 Set 一样, Map 的遍历顺序与成员添加顺序相同;**
Map 结构的 iterator 接口(`Symbol.iterator`)默认为 entires() 遍历器生成方法;

# WeakMap
有时我们想在某个对象上面存放一些数据, 但是这会形成对于这个对象的引用, WeakMap 对键名对象的引用都是弱引用, 当该键名对象在外界没有再被引用时, 垃圾回收机制会自动释放该键名对象所占用的内存(即键名对象和对应的键值会被自动清除);
**WeakMap 弱引用的只是键名而不是键值, 即键名对象不再被外界引用时, 才会被清除;**
```
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};
wm.set(key, obj);

obj = null; // 删除对键值的引用
wm.get(key); // Object {foo: 1} 没有被垃圾回收机制清除
```

* WeakMap 只接受对象作为键名 (null 除外), 不接受其它类型的值作为键名;
* 同理, WeakMap 没有 size 属性, 遍历方法以及 clear();

## WeakMap 应用场景
1. 存储 DOM 节点作为键名, 存储其状态作为键值;
2. 部署私有属性;