---
title: ES6篇 - class 继承
date: 2021-04-19 13:26
tags: 
- JavaScript
- ES6
- 面试
categories: JavaScript
excerpt: 文章是以"阮一峰ES6入门"为参考而著的一篇学习总结, 请酌情参考;
---

请结合 [Class 的继承](https://es6.ruanyifeng.com/#docs/class-extends) 一起阅读本文;

# 简介
```
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString()
  }
}
```
Class 类通过 **extends** 关键字实现继承;
extends 可看作是构造函数寄生组合式继承的封装; 它能让子类继承父类的所有属性和方法;

# super 关键字
super 是 class 类继承的一个重点, 它既可以当作**函数**使用，也可以当作**对象**使用; 使用super的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错;

## super()
当 super 作为函数调用时，代表父类的构造函数;
```
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```
ES6 规定**在子类 constructor 方法中必须调用 super 方法**, 否则新建实例时会报错; 在子类的构造函数中，只有调用 `super()` 之后，才可以使用 this 关键字，否则会报错;
**原因:** 子类自己的 this 对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用 `super()`，子类就得不到 this 对象;

与 ES5 继承的**区别**: 
* ES5 的继承实质: 先创造子类的实例对象 this，然后再将父类的方法添加到 this 上面 `Parent.apply(this)`;
* ES6 的继承机制实质: 先将父类实例对象的属性和方法，加到this上面（所以必须先调用 `super()`），然后再用子类的构造函数修改 this;

`super()` 虽然代表了父类 A 的构造函数，但是返回的是子类 B 的实例，即 `super` 内部的 this 指的是 B 的实例，因此 `super()` 在这里相当于 `A.prototype.constructor.call(this)`;
> super() <=> A.prototype.constructor.call(this, ...args)

作为函数时，`super()` 只能用在**子类的构造函数**之中，用在其他地方就会报错;

## super 对象
```
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // super 作为对象使用, 输出 2
  }
}

let b = new B();
```
super 作为对象时，在**普通方法**中，指向**父类的原型对象**; 在**静态方法**中，指向**父类**;

```
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
```
注: 定义在父类实例上的方法或属性，是无法通过 super 对象调用的;

```
class A {
  constructor() {
    this.x = 2;
  }

  print() {
    console.log(this.x)
  }
}

class B extends A {
  constructor() {
    super()
    this.x = 3;
    super.print(); // 输出 3
  }
}

let b = new B();
```
ES6 规定，在子类普通方法中通过 super 对象调用父类的方法时，父类方法内部的 this 指向当前的子类实例;
> super.print() <=> super.print.call(this)

```
class Parent {
  static myMethod(msg) {
    console.log('static', msg);
  }

  myMethod(msg) {
    console.log('instance', msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    super.myMethod(msg);
  }

  myMethod(msg) {
    super.myMethod(msg);
  }
}

Child.myMethod(1); // static 1
```
如果 super 作为对象，用在静态方法之中，这时 super 将指向父类，而不是父类的原型对象; 在子类的静态方法中通过 super 调用父类的方法时，方法内部的 this 指向当前的子类，而不是子类的实例。


# 类的 prototype 属性和 __proto__ 属性
class 类同时有 `prototype` 属性和` __proto__` 属性, 同时存在两条继承链;
1. 子类的`__proto__`属性，表示构造函数的继承，总是指向父类; `B.__proto__ === A`;
2. 子类prototype属性的`__proto__`属性，表示方法的继承，总是指向父类的`prototype`属性; `B.prototype.__proto__ === A.prototype`;

这两条继承链，可以这样理解：
* 作为一个对象，子类的原型（__proto__属性）是父类;
* 作为一个构造函数，子类的原型对象（prototype属性）是父类的原型对象（prototype属性）的实例;

其原因与 ES6 类的继承实现模式有关:
```
class A {}
class B {}

// 原型链继承(继承共享方法)
Reflect.setPrototypeOf(B.prototype, A.prototype);
// 静态属性继承(继承静态定义)
Reflect.setPrototypeOf(B, A);
```
class 继承时, 实例属性通过 `constructor()` 创建 this 对象并挂载父类实例属性实现, 共享方法通过原型链继承实现, 静态方法和属性通过类继承实现; 
> 个人认为类之所以有两条继承链, 是因为类既可以被视为对象, 又可以被视为构造函数, 类作为对象时具有了添加静态属性和方法的能力, 类作为构造函数时实现了 ES5 构造函数的功能;


# 实例的 __proto__ 属性
```
var p1 = new Point(2, 3);
var p2 = new ColorPoint(2, 3, 'red');

p2.__proto__ === p1.__proto__ // false
p2.__proto__.__proto__ === p1.__proto__ // true
```
子类实例的`__proto__`属性的`__proto__`属性，指向父类实例的`__proto__`属性; 即子类实例的原型的原型，是父类实例的原型; 
> 看似很绕口, 其实很好理解: 子类实例的原型实际就是父类实例(参考ES5 构造函数组合式继承)`p2.__proto__ === p1`;
> 另一种理解: 类实例的原型同于类的原型`p1.__proto__ === Point.prototype`, 因此上等式又可以写为 `ColorPoint.prototype.__proto__ === Point.prototype`, 是不是很熟悉, 就是类的其中一条继承链;

# 原生构造函数继承
## 原生构造函数
Boolean()
Number()
String()
Array()
Date()
Function()
RegExp()
Error()
Object()

## ES5 和 ES6 原生构造函数继承的区别
**ES5 的原生构造函数无法继承;** 这是因为原生构造函数会忽略 apply 等 thisArgs 绑定方法传入的 this, 也就是说, 原生构造函数的 this 无法绑定, 导致拿不到父类的内部属性;
**ES6 允许继承原生构造函数定义子类**
产生上述差异的原因: ES5 是先新建子类的实例对象this，再将父类的属性添加到子类上，由于父类的内部属性无法获取，导致无法继承原生的构造函数; ES6 是先新建父类的实例对象this，然后再用子类的构造函数修饰this，使得父类的所有行为都可以继承;
因此, **ES6 的 class extends 能够自定义原生数据结构的子类**, 相较于 ES5 有较大的提升;

# Mixin 模式实现
Mixin 指的是多个对象合成一个新的对象，新对象具有各个组成成员的接口;
> 即将多个对象整合到一个对象内, 并提供统一的调用接口;

## 简单实现
```
const a = {
  name: 'Siri',
}

const b = {
  age: 18,
}

const c = {...a, ...b} // {name: 'Siri', age: 18}
```

## 类的 Mixin 模式完整实现
```
function mix(...MixClasses) {
  class Mix {
    constructor() {
      for (let MixClass of MixClasses) {
        copyProperties(this, new MixClass()); // 克隆实例属性
      }
    }
  }

  for (let MixClass of MixClasses) {
    copyProperties(Mix, MixClass); // 克隆静态属性
    copyProperties(Mix.prototype, MixClass.prototype); // 克隆原型属性
  }

  return Mix;
}

// 私有方法, 将 source 克隆至 target;
function copyProperties(target, source) {
  // 遍历 source 所有属性
  for (let key of Reflect.ownKeys(source)) {
    if (key !== 'constructor'
      && key !== 'name'
      && key !== 'prototype'
    ) {
      // 克隆属性描述对象
      let desc = Reflect.getOwnPropertyDescriptor(source, key);
      Reflect.defineProperty(target, key, desc);
    }
  }
}

class A {}
class B {}
class SubClass extends mix(A, B) {
  // ...
}
```