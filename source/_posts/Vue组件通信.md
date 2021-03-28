---
title: Vue组件通信
date: 2021-03-28 13:35
tags: Vue
categories: Vue
excerpt:  Vue 作为一个轻量级的前端框架，其核心就是组件化开发。组件实例的作用域是相互独立的，这就意味着不同组件之间的数据无法相互引用。在实际项目开发过程中，对于多层级的状态管理我们可以使用vuex，但当需要单独封装通用组件时，我们需要访问其他组件的数据，就需要解决组件间通信的问题。在 vue 中组件之间的关系有：父子，兄弟，隔代。针对不同的关系，怎么实现数据传递，是本文重点记录的。
---

# Vue组件通信
参考链接：[组件之间相互传值的方式](https://segmentfault.com/a/1190000022700216)

## 父传子(props)
一：在父组件的子组件标签中绑定自定义属性
二：子组件中使用props接收
**父组件**
```
<user-detail :myName="name" :myObject="object" />
    
export default {
    components: {
        UserDetail
    }
    ......
}
```
**子组件**
```
export default {
    props: {
      myName: {
        type: String,  // 变量类型
        default: '', // 默认值
        requires: true, // 表示必填选项
      },
      myObject: {
        type: Object,
        default() {
          return null;
        }, // 对象或数组默认值需要用函数返回
      },
    }
}
```
子组件接受的父组件的值分为**引用类型**和**普通类型**：
**普通类型：**字符串（String）、数字（Number）、布尔值（Boolean）、空（Null）
**引用类型：**数组（Array）、对象（Object）
vue 单向数据流，规定了组件之间的数据是单向流通的，子组件是不允许直接对父组件传来的值进行修改的。我们需要先把传过来的值重新赋值给data中的一个变量，然后再更改那个变量。
当子组件接收的是普通类型数据时，修改该存储变量不会更改父组件相应的值；当子组件接收的是引用类型时，在子组件中修改存储的变量后，父组件的也会修改。这是因为引用类型变量保存的是对象的地址，父组件的变量，子组件接受的变量以及子组件另存为的变量都指向同一对象地址，因此会共享数据。除非有特殊需要，否则不要轻易修改传递的对象值。

## 子传父
### ($emit)
一：子组件绑定一个事件，当事件触发时，通过`$emit`向父组件发送一个事件(可携带参数传递，多个参数用对象包裹)
二：在父组件对应子组件上定义并绑定对应于`$emit`发送的事件(可接受携带的参数)
**子组件**
```
<button @click="changeParentName">改变父组件的name</button>
​
export default {
    methods: {
        //子组件的事件
        changeParentName: function() {
            this.$emit('handleChange', 'Jack') // 向父组件发送handleChange事件并传参Jack
            // 注：此处事件名称与父组件中绑定的事件名称要一致
        }
    }
}
```
**父组件**
```
<child @handleChange="changeName"></child>
​
methods: {
    changeName(name) {  // name形参是子组件中传入的值Jack
        this.name = name
    }
}
```
父组件用`@`接收子组件发送的事件。父组件需要定义一个函数来接受子组件发送的事件。子组件向父组件发送事件，需要特定的条件触发(若要实现子组件变量修改后，立刻改变父组件变量，可以用watch监听子组件变量，并在watch内定义函数发送自定义事件)。

### (callback)
通过“父传子”回调函数实现“子传父”的功能
一：在父组件中定义一个callback函数，并把 callback 函数传过去
二：在子组件中接收，并执行 callback 函数
**父组件**
```
<child :callback="callback"></child>
​
methods: {
    callback: function(name) {
        this.name = name
    }
}
```
**子组件**
```
<button @click="callback('Jack')">改变父组件的name</button>
​
props: {
    callback: Function,
}
```

## $refs
父组件可以通过`$refs`访问指定的子组件实例，从而调用组件的方法或访问数据。
**这种方式的组件通信不能跨级**
**使用`$refs`时要注意，此时子组件可能未完全挂在完成；因此我们一般使用`$refs`调用子组件的data和methods(在子组件created阶段就完成了)**
此外还有`$parent`,`$children`，但不建议使用。
**子组件**
```
export default {
    data () {
        return {
            title: '子组件'
        }
    },
    methods: {
        sayHello () {
            console.log('Hello');
        }
    }
}
```
**父组件**
```
<template>
  <child ref="childRef" />
</template>
​
<script>
  export default {
    created () {
      // 通过 $ref 来访问子组件
      console.log(this.$refs.childRef.title);  // 子组件
      this.$refs.childRef.sayHello(); // Hello

      // 通过 $children 来调用子组件的方法
      this.$children.sayHello(); // Hello
    }
  }
</script>
```

## 兄弟通信
### props 和 $emit 结合使用
同级子组件通过父组件作为中转站传值。一个子组件通过 `$emit` 传递参数到父组件，父组件接收传递的值，并通过 props 传递给另一个子组件。
**该方法跨级传递时过于繁琐，需要从底层一步步向上传递，到根组件后再一步步向下传递。后续介绍的空实例中转可以解决该问题**
**父组件**
```
<child-a :myName="name" />
<child-b :myName="name" @changeName="editName" />  
    
export default {
    data() {
        return {
            name: 'John'
        }
    },
    components: {
        'child-a': ChildA,
        'child-b': ChildB,
    },
    methods: {
        editName(name) {
            this.name = name
        },
    }
}
```
**子组件**
```
// child-a 组件
<p>姓名：{{ newName }}</p>
    
<script>
export default {
    props: ["myName"],
    computed: {
        newName() {
            if(this.myName) { // 判断是否有值传过来
                return this.myName
            }
            return 'John' //没有传值的默认值
        }
    }
}
</script>

// child-b 组件
<p>姓名：{{ myName }}</p>
<button @click="changeName">修改姓名</button>
    
<script>
export default {
    props: ["myName"],
    methods: {
        changeName() {
            this.$emit('changeName', 'Lily')   // 触发事件并传值
        }
    }
}
</script>
```

### 空Vue实例中转(适用于任何场景)
一：创建一个 EventBus.js 文件，暴露一个 vue 实例
```
import Vue from 'Vue'  
export default new Vue()
```
二：在要传值的文件里导入这个空 vue 实例，定义方法并通过 `$emit` 发送事件函数（也可以在 main.js 中全局引入该 js 文件）
三：在接收传值的组件中也导入 vue 实例，通过 `$on` 监听回调，回调函数接收所有触发事件时传入的参数
**注意：空Vue实例的this指向当前父级块作用域，一般为组件实例**
```
// 发送事件
<template>
    <div>
        <p>姓名: {{ name }}</p>
        <button @click="changeName">修改姓名</button>
    </div>
</template>
​
<script>
import { EventBus } from "../EventBus.js"
​
export default {
 data() {
     return {
         name: 'John',
     }
  },
  methods: {
      changeName() {
          this.name = 'Lily'
          EventBus.$emit("editName", this.name) // 触发全局事件,并且把改变后的值传入事件函数
      }
    }
}
</script>
```
```
// 监听事件并触发回调
import { EventBus } from "../EventBus.js"
​
export default {
    data() {
        return {
            name: ''
        }
    },
    created() {
         EventBus.$on('editName', (name) => {
             this.name = name
         })
    }
}
```

## provide & inject 跨级通信
[provide/inject -- Vue3官方文档](https://v3.cn.vuejs.org/guide/component-provide-inject.html)
Vue 高阶的方法：(provied & inject) 选项需要一起使用：允许一个祖先组件通过`provide`向其所有子孙后代注入一个依赖，不论组件层次有多深，子组件都能通过`inject`接收，该关系在起上下游关系成立的时间里始终生效
**祖先组件**
```
export default {
  provide: { // 作用：将注入的变量提供给它的所有子组件。
    name: 'Jack'
  }
}
```
**子孙组件**
```
export default {
  inject: ['name'], // 获取 provide 注入的变量
  mounted () {
    console.log(this.name);  // Jack
  }
}
```
**provide 和 inject 绑定并不是可响应的。即父组件的name变化后，子组件不会跟着变，然而，如果你传入了一个响应式的对象，那么其对象的 property 仍是响应式的**
在使用 provide / inject 时通常要处理响应问题：
在 Vue2 中：

1. 直接provide祖先实例(this)，然后在子孙组件中注入依赖，这样就可以在后代组件中直接修改祖先组件的实例的属性。缺点是挂载的无效东西太多
2. 使用 [Vue.observable -- Vue2](https://cn.vuejs.org/v2/api/#Vue-observable) 优化响应式 provide. (即用Vue.observable()作用到对象，将其注册为响应式)

在 Vue3 中：provide 需要传递组件实例data的property时，需要将provide变为返回对象的函数。
Vue3 对于 provide 响应式的处理，主要通过传递一个 ref property 或 reactive 对象给 provide 来改变这种行为。而 ref 对象通过 `Vue.computed()` 创建（它接受 getter 函数并为 getter 返回的值返回一个不可变的响应式 ref 对象，或者，它可以使用一个带有 get 和 set 函数的对象来创建一个可写的 ref 对象。）。
[Vue3文档--响应式计算和侦听](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#计算值)
[Vue3文档--provide/inject](https://v3.cn.vuejs.org/guide/component-provide-inject.html#处理响应性)