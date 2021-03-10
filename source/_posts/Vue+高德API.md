---
title: Vue.js挂载(高德)地图服务API
date: 2021-03-10 20:26
tags: [Vue,API]
categories: Vue
excerpt: "轨迹目的地预测系统"项目开发，基于Vue.js+Egg.js+MySQL进行全栈开发。该篇记录地图服务API挂载到Vue框架的具体操作，并总结归纳外部实例在Vue实例挂载过程。
---

# Vue+高德API
## 高德地图服务JS API异步加载
[高德地图服务JS API官方文档](https://developer.amap.com/api/javascript-api/guide/abc/prepare)
### Step1: 创建挂载容器
官方文档->准备->注册并申请Key值
添加div标签作为地图容器，同时为该div指定id属性：
```
//后续AMap创建的地图实例将挂载在container地图容器上
<div id="container"></div> 
```
为地图容器指定高度、宽度
```
#container {width:300px; height: 180px; } 
```

### Step2: 获取异步加载代码
参考文档：获取异步加载API的相关代码
```
window.onLoad  = function(){
    var map = new AMap.Map('container');
}
var url = 'https://webapi.amap.com/maps?v=1.4.15&key=您申请的key值&callback=onLoad';
var jsapi = doc.createElement('script');
jsapi.charset = 'utf-8';
jsapi.src = url;
document.head.appendChild(jsapi);
```

### Step3: 挂载实例
先挂完整的代码：
```
//TMap.vue

<template>
  //用于挂载实例的容器
  <div id="container"></div>
</template>

<script>
import Vue from "vue";

export default {
  name: "TMap",
  methods: {
    initMap() {
      // 高德异步加载地图相关代码
      window.onLoad = function () {
        // new AMap.Map("container",...) 创建高德实例并挂载在container上
        var map = new AMap.Map("container", {
          zoom: 11, //级别
          center: [116.397428, 39.90923], //中心点坐标
          viewMode: "3D", //使用3D视图
        });
        //将地图实例map注册到全局，方便在其他组件作用域内使用(调用$map即可)
        Vue.prototype.$map = map;

        //高德地图插件注册(可暂时忽略)
        AMap.plugin(
          ["AMap.Scale", "AMap.ToolBar", "AMap.OverView"],
          function () {
            //异步加载插件
            let scale = new AMap.Scale({
              visible: false,
            });
            let toolbar = new AMap.ToolBar({
              visible: false,
            });
            let overview = new AMap.OverView({
              visible: false,
              isOpen: true,
            });
            map.addControl(scale);
            map.addControl(toolbar);
            map.addControl(overview);
            // 全局注册
            let obj = {
              scale,
              toolbar,
              overview,
            };
            Vue.prototype.$plugin = obj;
          }
        );
      };
      var url =
        "https://webapi.amap.com/maps?v=1.4.15&key=此处填申领的Key值&callback=onLoad";
      var jsapi = document.createElement("script");
      jsapi.charset = "utf-8";
      jsapi.src = url;
      document.head.appendChild(jsapi);
    },
  },
  // 最关键的一步：在组件挂载完成后(mounted)挂载高德实例
  mounted() {
    this.initMap();
  },
};
</script>

<style>
#container {
  height: 100%;
}
</style>
```

**重点：**
高德JS API异步加载代码，会通过`new AMap.Map("容器id",...)`创建高德实例并挂载在指定容器上。
值得注意的是，我们需要将整个异步加载的过程放在 Vue 挂载完成后执行。
原因：
在 Vue 实例没有完全挂载完成前，我们无法保证组件内 `<div id="container"></div>` 已经被创建。
若容器未被创建，异步加载代码就已经被执行，地图是无法挂载到 Vue 实例上的。
因此，我们需要借助 Vue 的生命周期 mounted (在Vue实例挂载完成后执行内部函数)。
