---
title: Vue侦听功能(watch)
date: 2021-03-22 14:11
tags: Vue
categories: Vue
excerpt: 项目实战中用到了 watch 侦听属性，简单记录。
---

# Vue侦听(watch)
[vue(watch)官方文档](https://cn.vuejs.org/v2/api/#watch)
```
<script>
export default {
  name: "LayoutFrame",
  data() {
    return {
      isCollapsed: false,
    };
  },
  watch: {
    "$route.path": function (newValue, oldValue) {
      console.log(newValue);
      // js 中数组取末尾不能用索引-1，因为不支持该写法。
      this.curActiveName = newValue;
    },
  },
};
</script>
```
重点：

1. watch 接收对象作为属性值
2. 对象内类型：`{ [key: string]: string | Function | Object | Array }` 即 key 传入的是字符串，属性值有四种(字符串，函数，对象，数组)
3. watch 侦听的目标(target)作为 key 值，后面的回调函数在目标发生状态改变时触发，接收 `(newValue, oldValue)`，其中 newValue 为新状态值， oldValue 为旧状态值。
4. watch 可以侦听 store ，router，route 等全局实例，和监听普通变量一样，同样不需要加 this。

# JS 数组拼接技巧
通常会遇到以下应用场景：
对某一数组内的值进行处理，并返回新的数组。此时我们可以用 `arr.map(()=>{})` 来实现，
但当`arr.map(()=>{})` 内部返回数组时，我们希望将数组整合成一个数组，可采用如下方式：
```
[].concat.apply([],arr.map(()=>{
   ...
   return xxx
}))
```
`[].concat` 为数组拼接，`.apply()`表示对其回调函数返回值都应用`.concat()`方法，`.apply([],callback())`第一个参数为this指向，此处指向`[]`空数组，即定义初始值。

# html a标签禁用跳转功能
`<a>`标签除了作为跳转标签外，还可以包裹文本等元素，使之具有点击的功能(即鼠标放在文本上，显示点击图标)。
但包裹的文本，我们通常不希望其具备跳转的功能，因此，我们可以通过：
```
<a href="javascript:void(0)">
  ...
</a>
```
取消其跳转的功能