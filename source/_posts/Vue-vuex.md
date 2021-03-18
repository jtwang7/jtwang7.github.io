---
title: Vue.js - vuex
date: 2021-03-18 13:53
tags: 
- Vue
- vuex
categories: Vue
excerpt: Vue框架学习笔记：vuex全局状态管理。vue内组件间的信息传递一般通过父子组件层层通信，对于某些需要全局使用且频繁调用的状态而言，通过父子层级传递太过复杂。Vuex全局状态管理则在一定程度上解决了这一问题。
---

# Vue-vuex
本文通过练习项目来整体梳理Vue中vuex的组织结构以及使用方式。

## 文章组织结构
1. 项目结构
2. vuex 安装 | 引入 | 挂载
3. vuex
4. vuex 分离管理

## 项目结构
```
learnvuex
├─ .browserslistrc
├─ .eslintrc.js
├─ .gitignore
├─ babel.config.js
├─ package-lock.json
├─ package.json
├─ public
│  ├─ favicon.ico
│  └─ index.html
├─ README.md
└─ src
   ├─ App.vue
   ├─ assets
   │  └─ logo.png
   ├─ components
   │  ├─ OrgCounter.vue
   │  ├─ PCCounter.vue
   │  └─ VuexCounter.vue
   ├─ main.js  //项目入口文件(Vue实例)
   └─ store  //vuex文件夹
      ├─ actions.js
      ├─ getters.js
      ├─ index copy.js
      ├─ index.js  //vuex入口文件
      ├─ modules
      │  └─ moduleA.js
      ├─ mutations-types.js
      └─ mutations.js
```

## Vuex 安装 | 引入 | 挂载
1. npm 安装 vuex: `npm install vuex --save`
2. 引入 vuex 模块，注册vuex插件: 

```
// /store/index.js
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);
```

3. vuex 实例化

```
const store = new Vuex.Store({
  state:{},
  mutations: {},
  actions: {},
  getters: {},
  modules: {},
})

export default store;
```

4. 在项目入口文件的Vue实例中挂载 store 实例

```
import Vue from 'vue'
import App from './App.vue'
import store from './store/index'

Vue.config.productionTip = false

new Vue({
  store,
  render: h => h(App),
}).$mount('#app')
```

## Vuex
### 概述
**作用**：存放多界面共享的数据，同时将其添加到Vue的响应式系统。
> 注！在 Vuex 中初始化的数据会被添加进入 Vue 的响应式系统，当其管理的数据状态发生变化时，页面也会实时渲染刷新。Vue 的响应式是十分关键和必须要重视的，有些数据更改方式若没有遵循Vue的更改规则，可能不会被响应式系统所响应。

**存储路径**：项目中 vuex 往往存放在名为 `store` 的文件夹内，并以 index.js 作为 vuex 的入口文件。

### store 实例
```
const store = new Vuex.Store({
  state:{},
  mutations: {},
  actions: {},
  getters: {},
  modules: {},
})
```
**vuex 创建实例调用的是 `Vuex` 的 `Store` 方法！！！**这也是为什么将vuex的存储路径文件夹取名为store的原因。
在 store 实例下，包含5个配置项属性(属性名由Vuex规定，不能随意修改)：

1. state: 存放全局共享和管理的状态变量
2. mutations: 定义同步操作的函数。(注：mutations 内定义的函数必须为同步函数，且 mutations 往往是对 state 中状态做一系列操作，没有返回值，若在mutations的定义函数中返回值，最终调用mutations内方法时，接收的值将会是undefined)
3. actions: 定义异步操作的函数。所有放在 store 实例中管理的异步操作都要定义在 actions 中，并且通过`ctx.commit()`注册mutations内的方法来执行异步操作状态落定后的一系列同步处理操作。
4. getters: 类似vue的computed计算属性，对 state 内的状态变量做一定的预处理后再将其返回，getters 内定义函数调用时也同 computed 一样，以属性的方式调用。(mutations 内方法无法返回值，若有返回值的需求，可以考虑在 getters 内定义)
5. modules: Vuex 创建 state 时就指定它为单一状态树。这也决定了我们只能创建一个 store 仓库。尽管 Vuex 有且仅能存在一个 store，它内部仍能通过 modules 进行代码分离。

#### state
作用：存储"共享变量"
属性值：接收 Object 对象类型，对象内声明并初始化变量。
```
state: {
    shareCount: 0,
    students: [
        { id: 1, name: 'zhangsan', age: 18 },
        { id: 2, name: 'lisi', age: 30 },
        { id: 3, name: 'wangwu', age: 22 },
    ],
},
```
组件内state使用：
通过挂载在全局的 store 实例获取。
`$store.state.shareCount`

#### mutations
作用：定义"处理共享变量的同步操作"
属性值： 接收 Object 对象类型，对象内声明同步方法。
**方法默认传参**：定义方法时默认传入两个参数且只接受这两个参数：
在参数中，第一参数为当前store的`state`；第二参数为mutation的载荷`payload`，用于接受传递的额外数据。

1. `state` : 对应store仓库内的state，因此mutations方法内不需要通过this.state调用共享变量
2. `payload` : 组件在注册(后续会提及mutations在组件中的使用)时，需要传递的参数通过 payload 传递。若只传一个参数，则 payload 及等价于该参数，若传递多个参数，则需要用 `{}` 包裹成对象赋值给 payload。

```
mutations: {
    addCount(state) {
        state.shareCount++;
    },
    subCount(state) {
        state.shareCount--;
    },
    // mutations 中定义的函数可以看作两部分：
    // 1.type(事件类型)： 即 addFiveCount
    // 2.回调函数：即(state,payload) {...}
    [ADDFIVECOUNT](state, payload) {
        state.shareCount += payload
    },
    [ADDMEMBERINFO](state, payload) {
        // Vuex 创建 state 时初始化的值被添加到 Vue 的响应式系统中
        // 通过类似于 xx = xxx 等后续添加不能添加至响应式系统，因此页面不会更改。
        // 需要运用一些 Vue 支持响应式的方法或者 Vue.set()  Vue.delete() 对数据进行更新操作

        // state.students.push(payload)
        Vue.set(state.students, state.students.length, payload)
        // Vue.set(第一参数:传入需要改变的对象, 第二参数:(Number:传入需要发生改变的位置索引 | String: 传入key), 第三参数:(传入更新的数据))
    },
    [SUBMEMBERINFO](state) {
        Vue.delete(state.students, 1)
    },
},
```
##### mutations 在组件内的使用
在组件中，需要定义一个函数，在该函数内通过`$store.commit('xxx', a)`注册 store 的 mutations 方法。
当存在多个传参时，通过 `{}` 包裹传值：`$store.commit('xx',{a:2,b:4})`

##### `['string']() {}`
在 mutations 中定义的函数可以视为两部分：

1. type(事件类型)： 即函数名(String类型)
2. 回调函数：即函数参数 + 函数体

理解mutations内函数的这两部分后，我们可以讲下 store 中函数的一般规范写法：
即用 `[ADDFIVECOUNT](state, payload) {...}` 代替 `addFiveCount(state, payload) {...}`
完整代码如下：
```
// index.js
import { ADDCOUNT } from './mutations-types'

const store = new Vuex.Store({
  ...
  mutations: {
    [ADDCOUNT](state,payload) {
      ...
    }
  }
})

// mutations-types.js
export const ADDCOUNT = 'addCount';
```
这么做的**原因**：
为防止书写错误引发一系列问题,通常将mutations的事件类型(type)赋给常量，后续调用 mutations 内的方法时,引用常量名而非变量名。 这样做的好处在于,若常量书写错误,可以很直观地从错误提示中找到。
**重点：**
在导出这些事件类型常量后,在使用时需要对其进行调用.
注: 方法可以写成如下形式:
`['string-text'](){...}`
因此,调用这些常量时, 不能直接写 ADDCOUNT() {},而应该写成 [ADDCOUNT](){}
这也是之前为什么强调 mutations 方法可以分为 1.事件类型 2.回调函数  两个部分.

##### 从 mutations 理解 Vue 的响应式系统
本文一开始就提及，Vuex 在创建 state 时，就将所有初始化的值添加到 Vue 的响应式系统中了。在状态管理过程中，若这些初始化的值发生了变化，页面可以实时的做出相应的改变。这便是 Vue 的响应式体现。但是在某些情况下，用错误的方法改变值可能触发不了 Vue 的响应式。

以对象为例：
若在 state 中声明了对象`obj = {num:2}`，后续对该对象进行添加/删除或更改等操作时，若通过`obj.age = 18`等类似操作添加属性，Vue 的响应式系统是不会触发的。但是！若初始化的变量是个值 `aaa = 1`，那么通过`aaa = 4`是可以被响应式系统监测的！why?

[Vue 官方文档](https://cn.vuejs.org/v2/guide/reactivity.html) 解释了这点：
**检测变化的注意事项：**
由于 JavaScript 的限制，**Vue 不能检测数组和对象的变化**。尽管如此我们还是有一些办法来回避这些限制并保证它们的响应性。
**对于对象**
Vue 无法检测 property 的添加或移除。由于 Vue 会在初始化实例时对 property 执行 getter/setter 转化，所以 property 必须在 data 对象上存在才能让 Vue 将它转换为响应式的。
```
let vm = new Vue({
  data:{
    a:1
  }
})
// `vm.a` 是响应式的

vm.b = 2
// `vm.b` 是非响应式的
```
对于已经创建的实例，Vue 不允许动态添加根级别的响应式 property。但是，可以使用 Vue.set(object, propertyName, value) 方法向嵌套对象添加响应式 property。
```
Vue.set(vm.someObject, 'b', 2)
// 还可以使用 vm.$set 实例方法，这也是全局 Vue.set 方法的别名
this.$set(this.someObject,'b',2)
```

**对于数组**
Vue 不能检测以下数组的变动：

1. 当你利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`
2. 当你修改数组的长度时，例如：`vm.items.length = newLength`

解决方法：
```
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
vm.$set(vm.items, indexOfItem, newValue)

// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```

由于 Vue 不允许动态添加根级响应式 property，所以你必须在初始化实例前声明所有根级响应式 property，哪怕只是一个空值(即要提前声明和初始化变量，不能在Vue实例创建完成后添加)。

```
[ADDMEMBERINFO](state, payload) {
    // 
    // 通过类似于 xx = xxx 等后续添加不能添加至响应式系统，因此页面不会更改。
    // 需要运用一些 Vue 支持响应式的方法或者 Vue.set()  Vue.delete() 对数据进行更新操作

    // state.students.push(payload)
    Vue.set(state.students, state.students.length, payload)
    // Vue.set(第一参数:传入需要改变的对象, 第二参数:(Number:传入需要发生改变的位置索引 | String: 传入key), 第三参数:(传入更新的数据))
}
```

#### actions
作用：定义"异步操作"
属性值： 接收 Object 对象类型，对象内声明异步方法。
**方法默认传参**：定义方法时默认传入两个参数且只接受这两个参数：
在参数中，第一参数为`ctx`；第二参数为载荷`payload`，用于接受传递的额外数据。
**`ctx`**: context，意思是上下文，功能同 store 类似。
context 与 store 区别主要在`modules`中:
context 针对上下文,若在 modules 中，context 只能调用 modules 内的 state,而不能直接调用 store 的 state
context 和 store 都是对象,但在modules中, context 中还包括了 rootState, rootGetters等. 而在store中的 context 则不包括,这就是上下文的体现.

```
actions: {
    asyncChangeName(context, payload) {
        return new Promise((resolve)=>{
            setTimeout(()=>{
                context.commit('changeName',payload)
            },2000)
            resolve()
        })
    }
},
```

##### actions 在组件内的使用
**!!! 异步操作包裹在 actions 方法内. 但异步操作中的同步操作放在 mutations 中,通过 `context.commit` 进行提交. 调用异步操作时,通过 `$store.dispatch()` 进行注册**
```
// 在组件内
methods: {
  aaa() {
    this.$store.dispatch('xxx',{a:1,b:2})
  }
}

// 在store - actions内
actions: {
  xxx(ctx, payload) {
    //异步操作
    ctx.commit('yyy', res)
  }
}

// 在store - mutations内
mutations: {
  yyy(state,payload) {
    //接收actions异步操作的返回值，执行同步操作
  }
}
```

##### 异步操作的Promise包装
异步操作可以用 Promise 进行包装,进而分离异步代码和后续处理代码，将异步操作封装后返回一个Promise,后续dispatch注册时就会得到Promise对象,进而接收resolve()传递的内部值做后续处理.
```
updateInfo(context, payload) {
    return new Promise((resolve) => {
        setTimeout(() => {
            context.commit(SUBMEMBERINFO);
            resolve(payload);
        }, 2000)
    })
}
```

#### getters
作用：存放"共享变量预处理操作(相似于计算属性，也以属性方式调用)"
属性值： 接收 Object 对象类型。
**方法默认传参**：定义方法时默认传入两个参数且只接受这两个参数：
在参数中，第一参数为`state`；第二参数为`getters`，可以通过 getters 调用当前已有的 getters 内部属性。
可以发现，在getters中没有payload参数，即我们不可以像mutations和actions一样传递参数。实际上getters作为共享变量的预处理操作，本身也没有接收参数的必要。但若要实现动态预处理变量，我们也可以通过一定的方法实现：**函数封装**

```
// getters: 
    getters: {
        stuFilter: state => {
            return state.students.filter((value) => (value.age > 20));
        },
        stuNum: (state, getters) => {
            return getters.stuFilter.length;
        },
        // 若要动态传入额外数据，则要用函数进行封装。
        dymStuFilter: state => {
            return (age) => {
                // 注意此处不需要传入state，因为该箭头函数在 (state)=>{} 的函数作用域内。
                return state.students.filter((value) => (value.age > age))
            }
        }
    },
```

##### getters 在组件内的使用
getters 的使用方式同 state 使用类似，通过 `$store.getters.xxx` 获取暴露出的getter。
若 getters 内定义了动态传参，则通过 `$store.getters.xxx` 得到的是一个函数，我们可以在函数后加入参数，例如：
```
<h2>{{$store.getters.dymStuFilter(18)}}</h2>
```
其中 `$store.getters.dymStuFilter` 是 getters 的一般调用方法，此时该值实际为形如 `(age)=>{}` 的函数，之后根据函数的使用方法 `xxx(18)`传入参数就可以了。

#### modules
作用：对store进一步进行分离
属性值： 接收 Object 对象类型，对象内定义模块属性名和属性值，属性值内部配置项同store一样。
```
const moduleA = {
  // modules内state调用方式: $store.state.a.xxx
  // 解释: modules 中的state在编译后,会以对象的形式添加到 store 的 state 中
  state:{},

  // modules内mutations调用方式与store内mutations调用方式相同
  mutations: {},

  // 调用方法同store内的actions
  actions: {},

  // modules内getters调用方式与store内getters调用方式类似
  // modules内getters与store内getters存在的不同在于,
  // modules内的getters方法存在第三个参数 rootState, 其可以访问和调用 store 内的 state
  getters: {},

  modules: {},
}

modules: {
  a: moduleA,
}
```

## Vuex 抽离管理
至此，我们已经学习了vuex的一些基础。在项目中，光依靠 `/store/index.js` 存放大量的代码是不易于管理的。我们通常还需要对`index.js`内的代码做一些抽离。
抽离原则：
state 抽离到变量保存，仍存放于 index.js 中
其余各项按照属性名建立响应的 js 文件存储，同样将主要配置抽离成变量保存。
抽离的目录结构如下：
```
store  //vuex文件夹
  ├─ actions.js // 管理actions
  ├─ getters.js  // 管理getters
  ├─ index.js  // vuex入口文件，管理state
  ├─ modules  //管理modules
  │  └─ moduleA.js
  ├─ mutations-types.js  // 存放常量变量名
  └─ mutations.js  // 管理mutations
```

### 完整代码(抽离后)
#### index.js
```
import Vue from 'vue';
import Vuex from 'vuex';

// 抽离的文件记住要导入
import mutations from './mutations'
import actions from './actions'
import getters from './getters'
import moduleA from './modules/moduleA'

Vue.use(Vuex);

// state 单独抽离成变量，放在 index.js 中管理
const state = {
    shareCount: 0,
    students: [
        { id: 1, name: 'zhangsan', age: 18 },
        { id: 2, name: 'lisi', age: 30 },
        { id: 3, name: 'wangwu', age: 22 },
    ],
}

const store = new Vuex.Store({
    state,
    mutations,
    getters,
    actions,
    modules: {
        a: moduleA,
    }
});

export default store;


// store文件夹通常要进行抽离
// 其中 store 的 state 抽离成变量保存在 index.js 中
// 其余部分均抽离成文件模块,并通过导出导入简化 index.js
```

#### mutations & mutations-types
```
export const ADDCOUNT = 'addCount';
export const SUBCOUNT = 'subCount';
export const ADDFIVECOUNT = 'addFiveCount';
export const ADDMEMBERINFO = 'addMemberInfo';
export const SUBMEMBERINFO = 'subMemberInfo';
```
```
import Vue from 'vue'
import { ADDCOUNT, SUBCOUNT, ADDFIVECOUNT, ADDMEMBERINFO, SUBMEMBERINFO } from './mutations-types'


export default {
    [ADDCOUNT](state) {
        state.shareCount++;
    },
    [SUBCOUNT](state) {
        state.shareCount--;
    },
    [ADDFIVECOUNT](state, payload) {
        state.shareCount += payload
    },
    [ADDMEMBERINFO](state, payload) {
        Vue.set(state.students, state.students.length, payload)
    },
    [SUBMEMBERINFO](state) {
        Vue.delete(state.students, 1)
    },
}
```

#### actions
```
import {SUBMEMBERINFO} from './mutations-types'

export default {
    updateInfo(context, payload) {
        return new Promise((resolve) => {
            setTimeout(() => {
                context.commit(SUBMEMBERINFO);
                resolve(payload);
            }, 2000)
        })
    }
}
```

#### modules
```
import Vue from 'vue'

export default {
    state: {
        name: 'jim'
    },
    mutations: {
        changeName(state,payload) {
            Vue.set(state, 'name', payload)
        }
    },
    getters: {
        fullName(state) {
            return lastName => state.name + lastName;
        },
        fullName2(state,getters,rootState) {
            return lastName => getters.fullName(lastName) + rootState.shareCount
        }
    },
    actions: {
        asyncChangeName(context, payload) {
            return new Promise((resolve)=>{
                setTimeout(()=>{
                    context.commit('changeName',payload)
                },2000)
                resolve()
            })
        }
    },
    modules: {},
}
```

#### getters
```
export default {
    stuFilter: state => {
        return state.students.filter((value) => (value.age > 20));
    },
    stuNum: (state, getters) => {
        return getters.stuFilter.length;
    },
    dymStuFilter: state => {
        return (age) => {
            return state.students.filter((value) => (value.age > age))
        }
    }
}
```