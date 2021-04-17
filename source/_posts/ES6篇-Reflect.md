---
title: ES6篇 - Reflect
date: 2021-04-17 17:59
tags: 
- JavaScript
- ES6
- 面试
categories: JavaScript
excerpt: 文章是以"阮一峰ES6入门"为参考而著的一篇学习总结, 请酌情参考;
---

# Reflect
## 概念
Reflect 对象: ES6 为**操作对象**而提供的新 API, 用于替代 Object 的一些操作方法:
1. 将Object对象的一些明显属于**语言内部的方法**（比如`Object.defineProperty`），放到Reflect对象上。
> 现阶段，某些方法同时在Object和Reflect对象上部署，未来的新方法将只部署在Reflect对象上。
2. 修改某些Object方法的返回结果，让其变得更合理。比如，`Object.defineProperty(obj, name, desc)`在无法定义属性时，会抛出一个错误，而`Reflect.defineProperty(obj, name, desc)`则会返回`false`。
```
// Object.defineProperty()
try {
  Object.defineProperty(target, property, attributes);
  // success
} catch (err) {
  // failure
}

// Reflect.defineProperty()
if (Reflect.defineProperty(target, property, attributes)) {
  // success
} else {
  // failure
}
```
3. 统一所有 Object 操作为函数式行为;
> 例如: `name in obj`, `delete obj[name]` 等 Object 操作是命令式的, 现用 Reflect 将其统一成函数式, `Reflect.has(obj, name)`, `Reflect.deleteProperty(obj, name)`
```
// 命令式 Object 操作
'assign' in obj;

// 函数式 Object 操作
Reflect.has(obj, 'assign');
```
4. Reflect 对象方法与 Proxy 对象方法一一对应; 使得 Proxy 对象可以方便调用 Reflect 对象方法, 完成默认行为, 作为修改行为的基础; 无论 Proxy 怎么修改该默认行为, 都可以通过 Reflect 获取原生的默认行为;
```
let proxy = new Proxy(target, {
  set(target, name, value, receiver) {
    let success = Reflect.set(target, name, value, receiver);
    return success;
  }
})
```

## Reflect 对象的静态方法
> Reflect.get(target, name, receiver)
> Reflect.set(target, name, value, receiver)
> Reflect.has(obj, name)
> Reflect.apply(target, thisArg, args)

1. Reflect.get(target, name, receiver)
查找并返回 target 对象 name 属性所对应的值, 如果没有该属性, 则返回 undefined;
> target: 目标对象, 若不是对象类型, 报错;
> name: 对象键名 key;
> receiver: 当 name 属性部署了 getter 时的 this 指向;

```
const obj = {
  name: 'Siri',
  age: 18,
  action: [
    'eat',
    'run',
    'sleep',
  ],
  get dosomething() {
    return `Hi ${this.name}`
  }
}

const myObj = {
  name: 'Wang',
  age: 20,
}

let name = Reflect.get(obj, 'name'); // Siri
let height = Reflect.get(obj, 'height'); // undefined
let res = Reflect.get(obj, 'dosomething'); // Hi Siri
res = Reflect.get(obj, 'dosomething', myObj); // Hi Wang
let err = Reflect.get(1, 'name') // TypeError: Reflect.get called on non-object; 第一参数不是对象, 无法读取属性;
```
2. Reflect.set(target, name, value, receiver)
设置target对象的name属性等于value; 若不存在 name 属性则先创建再赋值, 若存在则覆盖 name 属性值;
> target: 目标对象, 若不是对象类型, 报错;
> name: 对象键名 key;
> value: 期望设置的键值 value;
> receiver: 当 name 属性部署了 setter 时的 this 指向;

```
const obj = {
  name: 'Siri',
  age: 18,
  action: [
    'eat',
    'run',
    'sleep',
  ],
  get dosomething() {
    console.log(`Hi ${this.name}`);
  },
  set setAge(value) {
    if (value !== this.age) {
      console.log("Can't Change Your Age");
    } else {
      console.log(this.age);
    }
  }
}

const myObj = {
  name: 'Wang',
  age: 20,
}

let res = Reflect.get(obj, 'height'); // undefined
Reflect.set(obj, 'height', 1.6);
res = Reflect.get(obj, 'height'); // 1.6
Reflect.set(obj, 'height', 1.8);
res = Reflect.get(obj, 'height'); // 1.8

Reflect.set(obj, 'setAge', 20); // Can't Change Your Age
Reflect.set(obj, 'setAge', 20, myObj); // 20
```
3. Reflect.has(obj, name)
查找 obj 对象内是否存在 name 属性, 若存在返回 true, 不存在返回 false; 等价于 `name in obj` 的 in 运算符, 是命令行为向函数行为统一的体现;
> obj: 目标对象, 若不是对象类型, 报错;
> name: 所要查找的属性名

```
const myObj = {
  name: 'Wang',
  age: 20,
}

let bool = Reflect.has(myObj, 'name'); // true
bool = Reflect.has(myObj, 'height'); // false
```

4. Reflect.deleteProperty(obj, name)
等同于`delete obj[name]`，用于删除对象的属性; 返回一个布尔值。如果删除成功，或者被删除的属性不存在，返回true；删除失败，被删除的属性依然存在，返回false;
> obj: 目标对象, 若不是对象类型, 报错;
> name: 所要删除的属性名

```
const myObj = {
  name: 'Wang',
  age: 20,
}

let bool = Reflect.deleteProperty(myObj, 'age'); // true
bool = Reflect.deleteProperty(myObj, 'height'); // true
console.log(myObj); // { name: 'Wang' }
```

5. Reflect.construct(target, args)
等同于`new target(...args)`，提供了一种不使用 new 来调用构造函数的方法; 返回构造函数实例;
> target: 目标对象, 若不是对象类型, 报错;
> args: 构造函数初始化参数;

```
function F(name, age) {
  this.name = name;
  this.age = age;
}

const obj = Reflect.construct(F, ['Siri', 18]); // F { name: 'Siri', age: 18 }
```

6. Reflect.getPrototypeOf(obj)
> obj: 目标对象, 若不是对象类型, 报错;
读取并返回对象的__proto__属性，对应 `Object.getPrototypeOf(obj)`; `Reflect.getPrototypeOf` 和 `Object.getPrototypeOf` 的一个区别是: 如果参数不是对象，`Object.getPrototypeOf` 会将这个参数转为对象，然后再运行，而 `Reflect.getPrototypeOf` 会报错。(体现了开头介绍的 Reflect 特性, 其修改了原先 Object 方法的一些不合理操作);
```
const obj = {
  name: 'Siri',
  age: 18,
}

let res = Reflect.getPrototypeOf(obj); // Object.prototype

// 区别
let msg1 = Object.getPrototypeOf(1); // [Number: 0]
let msg2 = Reflect.getPrototypeOf(1); // TypeError: Reflect.getPrototypeOf called on non-object
```

7. Reflect.setPrototypeOf(obj, newProto)
设置目标对象的原型（prototype），对应 `Object.setPrototypeOf(obj, newProto)` 方法。它返回一个布尔值，若成功则返回 true, 设置失败返回 false;
`Reflect.setPrototypeOf` 与 `Object.setPrototypeOf` 区别: 如果第一个参数不是对象，`Object.setPrototypeOf` 会返回第一个参数本身，而 `Reflect.setPrototypeOf` 会报错; 如果第一个参数是 undefined 或 null，`Object.setPrototypeOf` 和 `Reflect.setPrototypeOf` 都会报错;
> obj: 目标对象, 若不是对象类型, 报错;
> newProto: 期望设置的原型;

```
const obj = {
  name: 'Siri',
  age: 18,
}

console.log(obj.length); // undefined

let bool = Reflect.setPrototypeOf(obj, Array.prototype); // true
console.log(obj.length);  // 0
```

8. Reflect.apply(func, thisArg, args)
等同于 `Function.prototype.apply.call(func, thisArg, args)` ，用于绑定 this 对象后执行回调函数。
一般来说，如果要绑定一个函数的this对象，用 `fn.apply(obj, args)` 即可，但是如果函数定义了自己的 apply 方法，就只能写成 `Function.prototype.apply.call(fn, obj, args)`，而采用Reflect对象可以简化这种操作。
> func: 绑定 this 对象后触发的回调函数
> thisArg: 期望绑定的 this 对象
> args: 传递给 func 的函数参数

```
const ages = [11, 33, 12, 54, 18, 96];

// 旧写法
const youngest = Math.min.apply(Math, ages);
const oldest = Math.max.apply(Math, ages);
const type = Object.prototype.toString.call(youngest);

// 新写法
const youngest = Reflect.apply(Math.min, Math, ages);
const oldest = Reflect.apply(Math.max, Math, ages);
const type = Reflect.apply(Object.prototype.toString, youngest, []);
```

9. Reflect.defineProperty(target, propertyKey, attributes)
等同于 `Object.defineProperty`，为对象定义属性, 若属性存在则覆盖原属性值, 返回一个布尔值。未来，后者会被逐渐废除，请从现在开始就使用 `Reflect.defineProperty` 代替它; 
`Reflect.defineProperty()` 与 `Reflect.set()` 使用场景区别: `Reflect.defineProperty()` 用于赋值和修改对象属性的属性描述(value, get, set, writable等), `Reflect.set()` 只能用于修改属性值(value);
> target: 目标对象, 若不是对象类型, 报错;
> propertyKey: 需要赋值的对象属性;
> attributes: 属性描述对象;

```
const obj = {
  name: 'Siri',
  age: 20,
}

Reflect.defineProperty(obj, 'age', {
  get() {
    console.log(`I'm 18 years old`);
  },
})

obj.age; // I'm 18 years old
```
10. Reflect.getOwnPropertyDescriptor(target, propertyKey)
等同于 `Object.getOwnPropertyDescriptor`，返回指定属性的描述对象，将来会替代掉后者;
`Reflect.getOwnPropertyDescriptor` 和 `Object.getOwnPropertyDescriptor` 的区别: 如果第一个参数不是对象，`Object.getOwnPropertyDescriptor()` 不报错，返回 undefined，而 `Reflect.getOwnPropertyDescriptor()` 会抛出错误，表示参数非法;
> target: 目标对象, 若不是对象类型, 报错;
> propertyKey: 属性键名;

```
const obj = {
  name: 'Wang'
}

Reflect.defineProperty(obj, 'name', {
  value: 'Siri',
  writable: false,
})

let objDescriptor = Reflect.getOwnPropertyDescriptor(obj, 'name');

/*
{
  value: 'Siri',
  writable: false,
  enumerable: true,
  configurable: true
}
*/
console.log(objDescriptor);
console.log(obj); // { name: 'Siri' }
```

11. Reflect.isExtensible(target)
对应 `Object.isExtensible`，返回一个布尔值，表示当前对象是否可扩展; 与 `Object.isExtensible` 区别: 如果参数不是对象，`Object.isExtensible` 会返回 false，因为非对象本来就是不可扩展的，而 `Reflect.isExtensible` 会报错;
> target: 目标对象, 若不是对象类型, 报错;

```
const myObj = {};

Reflect.isExtensible(myObj); // true
```

12. Reflect.preventExtensions(target)
对应 `Object.preventExtensions` 方法，用于让一个对象变为不可扩展。返回一个布尔值，表示是否操作成功, 若操作成功则返回 true, 反之返回 false;
与 `Object.preventExtensions` 的区别: 如果参数不是对象，`Object.preventExtensions` 在 ES5 环境报错，在 ES6 环境返回传入的参数，而 `Reflect.preventExtensions` 会报错;
> target: 目标对象, 若不是对象类型, 报错;

```
const myObject = {};

Reflect.preventExtensible(myObject); // true
Reflect.isExtensible(myObject); // false
```

13. Reflect.ownKeys(target)
返回对象的所有属性(用数组包裹)，基本等同于 `Object.getOwnPropertyNames` 与 `Object.getOwnPropertySymbols` 之和;
> target: 目标对象, 若不是对象类型, 报错;

```
const obj = {
  name: 'Siri',
  age: 18,
  [Symbol('action')]: 'eat',
}

let arr = Reflect.ownKeys(obj); // [ 'name', 'age', Symbol(action) ]
```