---
title: ES6篇 - Symbol
date: 2021-04-15 20:44
tags: 
- JavaScript
- ES6
- 面试
categories: JavaScript
excerpt: 文章是以"阮一峰ES6入门"为参考而著的一篇学习总结, 请酌情参考;
---

# Symbol
ES6 新增的一种**原始数据类型**;
Symbol 值是**独一无二**的, 基于此特性, Symbol 值通常作为对象的属性名使用, 从根本上**防止属性名的冲突**;
Symbol 值不能与其他类型的值进行运算, 它可以显式转为字符串(`.toString()`), 也可转为布尔值, 但不能转为数值, 因此**不能做运算操作**;

## Symbol 创建
Symbol 值**通过 Symbol() 函数生成**; 
Symbol() 函数**不能使用 new 命令**, 因为生成的 Symbol 是一个原始类型的值, 不是对象, 其不能添加属性;
Symbol() 函数**可接受一个字符串作为参数**, 该字符串用于表示 Symbol 实例的描述, 便于 Symbol 值的区分; 
```
let s1 = Symbol();
let s2 = Symbol();
let s3 = Symbol('foo');

s1 // Symbol()
s2 // Symbol(), 虽然与 s1 两者不相同, 但在显示台不能区分;
s3 // Symbol(foo)
```
Symbol 实例的描述可以通过显示转为字符串或通过 ES2019 提供的实例属性 `Symbol.prototype.description` 获得, `sym.description` 可以直接返回 Symbol 的描述;
```
const sym = Symbol('foo');

String(sym) // "Symbol(foo)"
sym.toString() // "Symbol(foo)"
sym.description // "foo", 可以发现与显式转为字符串还是有区别的;
```

## Symbol 用作对象属性名
好处: 保证对象不会出现同名属性, 避免属性被改写或覆盖;
将 Symbol 值用于属性定义或属性调用时, Symbol 值必须放在**方括号**中; **(Symbol 值本身被存储在变量中, 因此要符合变量调用的方式)**
```
const mySymbol = Symbol();
// 写法一
const obj = {
  [mySymbol] = 'Hello',
}
// 写法二
obj[mySymbol] = 'World';
// 写法三
Object.defineProperty(obj, mySymbol, { value: 'Hello' })
```

Symbol 值作为属性名时, 属性是公开的而不是私有的:
* Symbol 属性名不会被 for...in, for...of 遍历, 也不会被 Object.keys(), Object.getOwnPropertyNames(), JSON.stringify() 查找; 
* Symbol 属性名只能被 Obeject.getOwnPropertySymbols() 和 Reflect.ownKeys() 查找, 因此它不是私有属性;

基于 Symbol 值的特性(不会被常规方法遍历访问), 我们可以用 Symbol 为对象定义一些**非私有的, 但只用于内部**的属性;
```
const age = Symbol('age');

const obj = {
  firstName: 'Siri',
  lastName: 'John',
  [age]: 18,
}

// 一般的遍历无法访问到 Symbol 属性名
for (let [key, value] of Object.entries(obj)) {
  console.log(key);
  /**
   * firstName
   * lastName
   */
}

// Object.getOwnPropertySymbols() 获取指定对象的所有 Symbol 属性名并返回一个数组, 成员是当前对象的所有用作属性名的 Symbol 值
console.log(Object.getOwnPropertySymbols(obj));
// [ Symbol(age) ]

// Reflect.ownKeys()方法可以返回所有类型的键名, 包括常规键名和 Symbol 键名
console.log(Reflect.ownKeys(obj));
// [ 'firstName', 'lastName', Symbol(age) ]
```

## Symbol.for()
`Symbol.for()` 接收一个字符串作为参数, 它会**在全局环境中搜索**是否存在该参数名的 Symbol 值, 若**存在**, 则**返回**该 Symbol 值, 若**不存在**, 则**新建**一个以该字符串为名称的 Symbol 值, 并**注册到全局**;
```
const foo = Symbol('desc')
const foo1 = Symbol.for('foo')
const foo2 = Symbol.for('foo')

console.log(foo === foo1); // false
console.log(foo2 === foo1); // true
```
*注: 通过 Symbol() 创建的 Symbol 值不会被登记在全局环境中, 因此不能被 Symbol.for() 搜索, 所以 `foo === foo1` 结果为 false; 通过 Symbol.for() 创建的 Symbol 值会被登记在全局环境中, `foo1` 通过 `Symbol.for('foo')` 创建后, 全局环境中被登记了一个名为 foo 的 Symbol 值, 因此 `foo2` 接收的就是该 Symbol 值, 所以两者相等;*

### Symbol.for() 与 Symbol()
共同点: 都用于生成 Symbol 值;
区别: 前者会被登记在全局环境中供搜索, 后者不会;
Symbol.for()不会每次调用就返回一个新的 Symbol 类型的值, 而是会先检查给定的key是否已经存在, 如果不存在才会新建一个值; 
由于Symbol()写法没有登记机制, 所以每次调用都会返回一个不同的值

## Symbol.keyFor()
返回一个已登记的 Symbol 类型值的 key;
```
const foo = Symbol('desc')
const foo1 = Symbol.for('foo')

console.log(Symbol.keyFor(foo)); // undefined
console.log(Symbol.keyFor(foo1)); // foo
```

## 内置的 Symbol 值
1. Symbol.hasInstance
2. Symbol.isConcatSpreadable
3. Symbol.species
4. Symbol.match
5. Symbol.replace
6. Symbol.search
7. Symbol.split
8. Symbol.iterator
9. Symbol.toPrimitive
10. Symbol.toStringTag
11. Symbol.unscopables
