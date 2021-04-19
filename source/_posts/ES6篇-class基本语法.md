---
title: ES6篇 - class 基本语法
date: 2021-04-18 22：29
tags: 
- JavaScript
- ES6
- 面试
categories: JavaScript
excerpt: 文章是以"阮一峰ES6入门"为参考而著的一篇学习总结, 请酌情参考;
---

请结合 [Class 的基本语法](https://es6.ruanyifeng.com/#docs/class) 一起阅读本文;

# class
## 类的由来
JavaScript 通过*构造函数*生成*实例对象*;
ES6 提出的 class 写法只是让对象原型的写法更加清晰、更像**面向对象编程**的语法而已, 可以看作是 ES5 构造函数的**语法糖**; 
```
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
```
`constructor()` 是类的默认方法实现, 是该类的**构造函数**; 类中的 `this` 关键字代表类的**实例对象**; 除构造函数外, 还能在 class 内自定义任意方法, 如 `toString()` 方法, **方法与方法之间不需要逗号分隔**;
> 还记得 class 是 ES5 构造函数的语法糖实现吗? 看上例代码可知, 从形式上看 class 无非是对构造函数和自定义方法做了一层包装, 同时规定构造方法名为 constructor;

```
class Point {
  // ...
}

typeof Point // "function"
Point === Point.prototype.constructor // true
```
类的数据类型: **函数**;
**类**本身就指向**构造函数**;

> new Point() 实际上就是 new Point.prototype.constructor(), 因此 ES6 的类，完全可以看作构造函数的另一种写法;
> 类与构造函数的一点**区别**: 类必须通过 new 操作符创建实例;

```
let pt = new Point(1, 2);
pt.x // 直接从实例对象获取: this.x
pt.toString(); // 通过原型链调用: Point.prototype.toString()
```
不得不提的一点是: class 类中有两种对象值得去关注, 1. **this** 实例对象 2. **prototype** 原型对象; **在 class 中定义的属性或方法, 若没有显式定义挂载到 this 对象上, 则都会被添加至类的 prototype 原型对象**;
> class 声明类时, 一般将方法添加到原型对象上, 将属性添加至 this 实例对象上; 这也符合 ES5 构造函数添加方法和属性的习惯, 创建的实例拥有独有的属性, 同时共享原型对象的方法;

```
class Point {
  constructor(){
    // ...
  }
}

Object.assign(Point.prototype, {
  toString(){},
  toValue(){}
});
```
类的所有方法都定义在类的 prototype 属性上面, 因此在添加新的方法时, 通过 `Object.assign()` 方法一次向类添加多个方法;

```
class Point {
  constructor(x, y) {
    // ...
  }

  toString() {
    // ...
  }
}

Object.keys(Point.prototype)
// []
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```
类内部定义的所有方法都是**不可枚举**的; ES5 构造函数通过 `xxx.prototype.funcName = function() {...}` 添加方法是可枚举的;

## constructor()
```
class Point {}
// 等同于
class Point {
  constructor() {}
}
```
constructor()方法是类的**默认方法**; 一个类必须有 `constructor()` 方法，如果没有显式定义，一个空的 `constructor()` 方法会被**默认添加**。

```
let pt = new Point()
// 等同于
let pt = new Point.prototype.constructor()
```
通过 new 命令生成对象实例时，自动调用`constructor()`。
> 前面提到: 类本身指向构造函数, 因此 new 实例化类实际上就是调用了类内的 constructor();

```
class Foo1 {
  constructor() {
    this.name = 'Siri';
    // return this;
  }
}

class Foo2 {
  constructor() {
    this.name = 'Siri';
    return Object.create({
      name: 'John',
    });
  }
}

class Foo3 {
  constructor() {
    this.name = 'Siri';
    return 1;
  }
}

let foo1 = new Foo1();
let foo2 = new Foo2();
let foo3 = new Foo3();

console.log(foo1.name); // Siri
console.log(Reflect.getPrototypeOf(foo1) === Foo1.prototype); // true
console.log(foo2.name); // John
console.log(Reflect.getPrototypeOf(foo2) === Foo2.prototype); // false
console.log(foo3.name); // Siri
```
constructor()方法**默认返回实例对象 this**，当然也可以返回自定义对象; constructor 方法会隐式返回类的实例对象, this 对象原型为类(构造函数)的原型(见`foo1`); 若显式 return 值, 当 return 一对象时, 生成的实例即为该返回值对象, 它不在类的原型链上(见`foo2`), 当 return 非对象类型的返回值时, 会忽略该返回值(见`foo3`);
> 同 new 操作符内部实现一致, 无论是普通构造函数还是类构造函数生成实例对象, 都依赖于 new 的内部实现;

## 类的实例
```
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
```
用构造函数实现如下:
```
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function() {
  return '(' + this.x + ', ' + this.y + ')';
}
```
实例的属性除非显式定义在其本身（即定义在this对象上），否则都是定义在原型上（即定义在class上）;
> 如上例, 显式定义在 this 对象上的属性(或方法), 最终会挂载在创建的实例对象上, 为该实例对象所**独有**; 其余没有定义在 this 上的属性或方法(如`toString()`)都被定义在类原型(`Point.prototype`)上, 被该类的所有实例对象所**共享**(这是因为**类的所有实例共享一个原型对象**, 同 ES5);

## 取值函数(getter)和存值函数(setter)
```
class MyClass {
  constructor() {
    // ...
  }
  get prop() {
    return 'getter';
  }
  set prop(value) {
    console.log('setter: '+value);
  }
}

let inst = new MyClass();

inst.prop = 123; // setter: 123
inst.prop // 'getter
```
类内部可以使用 **get** 和 **set** 关键字，对某个属性设置存值函数和取值函数，**拦截该属性的存取行为**;
> 关于 getter 和 setter 可参考[数据属性](https://zh.javascript.info/property-descriptors)和[访问器属性](https://zh.javascript.info/property-accessors)

## 类属性名的表达式表示
```
const METHODNAME = 'methodName';

class Square {
  constructor(length) {...}

  [METHODNAME]() {
    ...
  }
}
```
类的属性名，可以采用表达式;

## class 的表达式定义
```
const MyClass = class Me {
  constructor() {}
  getClassName() {
    return Me.name;
  }
}
```
与函数一样，类也可以使用**表达式定义**(类本身就是函数类型); 上例中 `MyClass` 是类的外部名称, 用于外部引用, `Me` 是类内部名称, 只能在 Class 的内部可用, 如果类的内部没用到的话，可以省略类的內部命名;

```
let person = new class {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
}('张三');
```
应用: 与常规 class 声明相比, 采用 Class 表达式，我们可以写出**立即执行的 Class**;

## 注意事项
### 严格模式
考虑到未来所有的代码，其实都是运行在模块之中，所以 ES6 实际上把整个语言升级到了严格模式; 因此, **类和模块的内部，默认就是严格模式**。只要你的代码写在类或模块之中，就只有严格模式可用。

### 不存在提升
类不存在变量提升:  ES6 不会把类的声明提升到代码头部;
```
{
  let Foo = class {};
  class Bar extends Foo{};
}
```
原因: 在后续提及的类的继承中, 子类需要调用父类的构造函数来构建自己的实例对象 this, 因此就要保证父类在子类前被定义, 因此 ES6 规定类声明不能发生变量提升, 否则会导致子类在父类前定义的情况;如上例, Bar 子类由 class 声明, 若存在变量提升, 则会被提升到 let 命令(*let 不会发生变量提升*)之前, 导致 Bar 继承 Foo 时 Foo 还未被定义;

### name 属性
本质上，ES6 的类只是 ES5 的构造函数的一层包装，所以函数的许多特性都被Class继承，包括name属性;
```
let pt = class Point {}
console.log(pt.name); // Point
```
name 属性总是返回**紧跟在 class 关键字后面的类名(即类內部命名)**


### Generator 方法
```
class Foo {
  constructor(...args) {
    this.args = args;
  }

  * [Symbol.iterator]() {
    for (let arg of this.args) {
      yield arg;
    }
  }
}

for (let x of new Foo('hello', 'world')) {
  console.log(x);
}
// hello
// world
```
类内原型链上的普通方法定义不需要 function 关键字;
类内 Generator 方法定义即在普通函数基础上加上 `*`, 表示该方法是个 Generator 函数;

### this 指向
```
let logger = new class Logger {
  printName(name = 'there') {
    this.print(`Hello ${name}`);
  }

  print(text) {
    console.log(text);
  }
}();
logger.printName()
```
**类方法内部 this 默认指向类的实例**(因为通常通过类的实例来调用方法); 
```
const { printName } = logger;
printName(); // TypeError: Cannot read property 'print' of undefined
```
但是若单独使用类内方法, this 会指向方法运行的环境(class 严格模式下, this 指向 undefined);
> 结合 this 指向的知识点进行判断即可;

为防止类内方法因不当调用导致 this 指向出错的问题, 通常有两种方法解决:
1. 在构造函数`constructor()`中为方法绑定 this
```
class Logger {
  constructor(name) {
    this.name = name
    this.printName = this.printName.bind(this);
  }

  printName(name = 'there') {
    // 类内方法的 this 代表类实例, 通过 this 调用类方法, 再通过原型链找到 print() 并执行;
    this.print(`Hello ${name}`);
  }

  print(text) {
    console.log(text);
  }
}
```
2. 在构造函数`constructor()`中使用箭头函数
```
class Logger {
  constructor(name) {
    this.printName = (name = 'there') => {this.print(`Hello ${name}`)};
  }

  print(text) {
    console.log(text);
  }
}
```
3. 使用 Proxy 拦截, 获取方法时触发拦截, 并自动绑定 this
```
function selfish (target) {
  const cache = new WeakMap();
  const handler = {
    // 拦截属性访问
    get (target, key) {
      const value = Reflect.get(target, key);
      // 若不是函数则原值返回
      if (typeof value !== 'function') {
        return value;
      }
      // 是函数, 绑定实例对象, 并将添加记录到哈希表;
      if (!cache.has(value)) {
        cache.set(value, value.bind(target));
      }
      // 返回绑定后的函数
      return cache.get(value);
    }
  };
  const proxy = new Proxy(target, handler);
  // 返回 proxy 包装
  return proxy;
}

const logger = selfish(new Logger())
```

# class 静态方法
```
class Foo {
  static classMethod() {
    return 'hello';
  }
}

let foo = new Foo();

Foo.classMethod() // 'hello'
foo.classMethod() // TypeError: foo.classMethod is not a function
```
类内所有属性和方法都通过类生成的实例调用; 但如果在一个方法前，加上 **static** 关键字, 表示声明了一个**静态方法**，该方法不在实例调用，而是**直接通过类来调用**, 如果在实例上调用静态方法，会抛出一个错误，表示不存在该方法;

## 注意事项
1. 静态方法直接定义在类上, 而不是定义在类的原型上;
2. **静态方法的 this 关键字指向类**，而不是实例(因为调用时通过 `Foo.bar()` 调用, 根据 this 隐式调用规则可判断);
```
class Foo {
  static bar() {
    this.baz();
  }
  static baz() {
    console.log('hello');
  }
  baz() {
    console.log('world');
  }
}

Foo.bar() // hello
```
3. 静态方法可以与非静态方法重名;
4. 父类的静态方法，可以被子类继承;
5. 静态方法可从 super 对象上调用(super 对象代表父类的构造函数, 构造函数也就是类);

# 实例属性新写法
```
class foo {
  bar = 'hello';
  baz = 'world';

  constructor() {
    // ...
  }
}
```
实例属性除了定义在 `constructor()` 方法里面的 this 上面，也可以**定义在类的最顶层**;
好处: 所有实例对象自身的属性都定义在类的头部，看上去比较整齐，一眼就能看出这个类有哪些实例属性。
坏处: 之前提及类中除显式定义在 this 实力上的属性外, 其余都定义在类的原型上; 若省略 this 可能会将其判断到原型上;

# 静态属性
静态属性指的是 Class 本身的属性，即 `Class.propName`，而不是定义在实例对象（this）上的属性;
目前声明静态属性的方式(ES6 目前规定 Class 内部只有静态方法, 没有静态属性):
```
class Foo {}
Foo.prop = 1;
```
类的静态属性提案: 在类内部显式声明, 在实例属性前加上 static 关键字;
```
class Foo {
  static prop = 1;

  constructor() {
    // ...
  }
}
```
> 老写法的静态属性定义在类的外部。整个类生成以后，再生成静态属性。
> 缺点: 容易忽略静态属性，也不符合相关代码应该放在一起的代码组织原则。
> 提案是显式声明（declarative），而不是赋值处理，语义更好。

# 私有方法和私有属性
概念: 只能在类的内部访问的方法和属性，外部不能访问;
作用: 有利于代码的封装;
ES6 没有提供私有方法和私有属性的显式声明方式, 因此只能通过变通方法进行模拟实现;
1. 命名上区别私有和共有: 私有变量以 `_` 开头; 这种命名是不保险的，在类的外部，还是可以调用到这个方法。
```
class Widget {
  // 公有方法
  foo (baz) {
    this._bar(baz);
  }

  // 私有方法
  _bar(baz) {
    return this.snaf = baz;
  }

  // ...
}
```

2. 在类外部定义私有方法;
```
class Widget {
  foo (name) {
    print.call(this, name);
  }

  // ...
}

function print(name) {
  return this.name = name
}
```
> Widget 实例只能访问到 foo 方法, foo 实际是对 print 方法的封装, 保证了 print 方法的私有性;

3. 利用Symbol值的唯一性，将私有方法的名字命名为一个Symbol值; Symbol 值无法被一般的遍历方式获取, 基本达到了私有的效果, 但是不能排除通过 `Reflect.ownKeys()` 等方法获取;
```
const bar = Symbol('bar');

class MyClass {
  // 公有方法
  foo(baz) {
    this[bar](baz);
  }

  // 私有方法
  [bar]() {
    // ...
  }

}
```

## 私有属性和私有方法的提案
```
class Foo {
  #a;
  #b;
  constructor(a, b) {
    this.#a = a;
    this.#b = b;
  }
  #sum() {
    return this.#a + this.#b;
  
}

const foo = new Foo();
foo.#a // 报错
foo.#a = 42 // 报错
```
在属性名之前, 使用 `#` 关键字声明私有;
上面代码中，`#a` 就是私有属性, `#sum` 就是私有方法，**只能在类的内部使用**。如果在类的外部使用，就会报错。

# new.target 属性
`new.target` 返回 new 命令**当前**所作用的构造函数; 如果构造函数不是通过 new 命令或 Reflect.construct() 调用的，`new.target` 会返回 `undefined`;
**在函数外部使用 `new.target` 会报错**;

**约束构造函数必须经 new 操作符调用**
```
function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 命令生成实例');
  }
}
```

Class 类内部使用 `new.target`, **返回当前 Class**; (因为类本身就是构造函数)
```
class Person {
  constructor() {
    console.log(new.target === Person);
  }
}
const person = new Person(); // 输出 true
```
> 注意, new.target 返回的是当前的 Class, 当子类继承父类时, 父类内的 `new.target` 返回的是子类构造函数;

**基于 new.target 返回当前构造函数特性, 实现不能独立使用、必须继承后才能使用的类**
```
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error('本类不能实例化');
    }
  }
}

class Rectangle extends Shape {
  constructor(length, width) {
    super();
    // ...
  }
}
```
上例中, Shape 自身不能实例化, 只能通过继承的子类实例化;