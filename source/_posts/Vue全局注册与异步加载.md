---
title: Vue全局注册与异步加载
date: 2021-03-10 21:40
tags: 
- Vue
- async
- ES6
- Promise
categories: Vue
excerpt: 轨迹目的地预测系统项目开发，基于Vue.js+Egg.js+MySQL进行全栈开发。项目开发过程中，遇到异步加载插件无法正常显示的问题，后排查原因发现是插件未完成全局注册就被调用于实例挂载，特此记录。
---

# Vue全局注册与异步加载
## 应用需求描述
单独封装一个地图控件，实现开关控制地图插件的显示。当开关状态为ON时，在地图上显示相应的地图插件，当开关状态为OFF时，隐藏地图插件。
![](/img/posts_img/20210310215211593_26729.png)

## Vue 代码
**MapPlugin.vue:** 整体插件框封装
```
<template>
  <div id="map-plugin">
    //预留插槽，开发者可向内插入任意数目插件
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "MapPlugin",
  
};
</script>

<style>
#map-plugin {
  width: 180px;
  min-height: 100px;

  padding: 0 20px;
  margin-left: 10px;

  border: 3px ridge #515a6ece;
  border-radius: 5px 25px;

  display: flex;
  flex-direction: column;

  position: fixed;
  bottom: 40px;
  left: 0;
}
</style>
```

**PluginItem.vue:** 单个插件封装
```

<template>
  <div class="plugin-item">
    {{ text }}
    //使用了iView UI的 i-switch 封装组件
    <i-switch v-model="switch1" @on-change="change" size="large">
      <span slot="open">ON</span>
      <span slot="close">OFF</span>
    </i-switch>
  </div>
</template>

<script>
export default {
  name: "PluginItem",
  props: {
    text: {
      type: String,
      default: "",
    },
    callback: Function,
  },
  data() {
    return {
      switch1: false,
    };
  },
  methods: {
    change(status) {
      //   console.log(status);
      let cn = "";
      if (status) {
        cn = "开";
      } else {
        cn = "关";
      }
      this.$Message.info("开关状态：" + cn);
    },
  },
  //watch 侦听 switch1 变量状态变化，发生变化时调用函数 callback
  //callback 函数外部传入
  //watch 使用细节参考 Vue.js 官网
  watch: {
    switch1: function (newValue, oldValue) {
      this.callback(newValue);
    },
  },
};
</script>

<style>
.plugin-item {
  display: flex;
  justify-content: space-between;
  padding: 10px 10px;
}
</style>
```

### 定制封装个性化插件组件
```

<template>
  <MapPlugin>
    //向预留插槽内插入PluginItem.vue封装的插件
    <PluginItem text="比例尺" :callback="getScale" />
    <PluginItem text="鹰眼控件" :callback="getOverView" />
    <PluginItem text="工具条" :callback="getToolBar" />
  </MapPlugin>
</template>

<script>
import { MapPlugin, PluginItem } from "components/common/plugin";
import { getPlugin } from "./getPlugin";

export default {
  name: "ModelOnePlugin",
  components: {
    MapPlugin,
    PluginItem,
  },
  methods: {
    // 填坑1：this.$plugin 在页面挂在完成前没有创建，需要用异步方法，等待 this.$plugin 赋值后获取值并执行
    /**填坑2：
     * this 指向问题. Vue 方法等定义中，若用箭头函数，this 指向上级父作用域，因此值为 undefined (打印可知)
     * 获取 Vue 实例的 this，需要通过 function(){} 声明函数。同理在 getPlugin.js 中定义的箭头函数，this => undefined
     * 解决 this 指向：最有效的办法就是获取正确的 this 指向并将它保存到变量内使用。
     */
    getScale: async function (active) {
      console.log(this);
      let vueinstance = this;
      let obj = await getPlugin(vueinstance);
      if (active) {
        // .show() 插件展示
        obj.scale.show();
      } else {
        // .hide() 插件隐藏
        obj.scale.hide();
      }
    },
    getOverView: async function (active) {
      let vueinstance = this;
      let obj = await getPlugin(vueinstance);
      if (active) {
        // .show() 插件展示
        obj.overview.show();
      } else {
        // .hide() 插件隐藏
        obj.overview.hide();
      }
    },
    getToolBar: async function (active) {
      let vueinstance = this;
      let obj = await getPlugin(vueinstance);
      if (active) {
        // .show() 插件展示
        obj.toolbar.show();
      } else {
        // .hide() 插件隐藏
        obj.toolbar.hide();
      }
    },
  },
};
</script>

<style>
</style>
```

getPlugin 函数单独封装
```
const getPlugin = (vueinstance) => {
    console.log(vueinstance);
    // 此处箭头函数内 this 上级作用域仍为一个箭头函数，值为 undefined
    return new Promise((resolve, reject) => {
        if (vueinstance.$plugin) {
            resolve(vueinstance.$plugin)
        }
    })
}

export {
    getPlugin,
}
```

## 代码解析
MapPlugin.vue 和 PluginItem.vue 两个组件封装了通用的插件框和插件
主要细节在于插件组件的使用：
**问题描述一：**
由于我们采用自己封装的通用插件组件，因此在定制特定功能的插件时，我们需要传入相对应的插件对象，这些插件对象为高德地图创建出来的插件实例。**注意：** 在挂载高德地图实例的时候，我们就提到过，外部实例挂载在Vue实例上，需要等待Vue实例完全挂载。在地图实例挂载时，在mounted中回调创建实例的函数是为了防止Vue实例未挂载完全就执行实例的创建，从而地图实例缺少挂载容器而挂在失败的问题。而在插件实例挂载中，我们本意是在创建通用插件时，传入相应的插件实例从而完成插件个性化定制。但是此时插件实例还未被创建(只有在mounted后执行)，因此用同步方法传递实例自然是失败了。
**解决方法：**
用异步方法传递插件实例，插件实例创建后，本文方法将插件实例统一放到对象内并通过`Vue.prototype.$xxx`全局注册，在插件中通过异步方法获取该全局对象，即用`Promise`对象包裹异步操作，当检测到`$plugin`(本文注册的全局对象)时，通过`resolve()`落定Promise状态为fulFilled，这样我们就能获得包含有插件实例的全局对象，然后对通用插件进行个性化配置。

**问题描述二：**
上述代码中，我们要特别注意 this 的指向问题，在 Vue 的 methods 定义中，使用箭头函数，由于 this 会指向最近外层作用域`{}`，因为methods并没有 this，所以值为 undefined。我们需要用 `function() {}` 来定义，这样定义的函数的 this 会指向 Vue 实例。同理，单独封装的 `getPlugin.js` 中的箭头函数 this 经过判断也是 undefined，需要做进一步处理保证其获得的是 Vue 实例。
**解决方法：**
为了避免多次判断 this 指向，我们可以在明确 this 指向的地方，用变量保存 this 值，然后通过参数传递该变量，用该变量来作为 this 使用。
