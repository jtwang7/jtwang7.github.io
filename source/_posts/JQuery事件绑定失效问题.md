---
title: 社矫项目：jQuery事件绑定失效原因总结
date: 2020-11-22 19:28
tags: jQuery
categories: jQuery
excerpt: 在社区矫正人员项目中，遇到了用 jQuery 进行事件绑定失效的问题，即在相应事件触发时无法执行内部绑定的函数，后在网上找到了解决方法及原因，特此记录。

---

# jQuery事件绑定失效问题
## 代码
社矫项目中部分 js 代码及 html 代码如下：
**监听汉字输入的 js 代码：**
```
// 输入汉字监听
$(function (){
    $('#nameSearch').on('compositionend', event=>{
        console.log(1);
        let val = $('#nameSearch').val();
        if(isNaN(Number(val))){
            debounce(tooltips, 2000)();
        }
    });
})
// $('#nameSearch').on('compositionend', event=>{
//     console.log(1);
//     let val = $('#nameSearch').val();
//     if(isNaN(Number(val))){
//         debounce(tooltips, 2000)();
//     }
// })
```

**搜索框的 html 代码：**
```
<div style="position: absolute; width: 300px; height: auto; left:25px">
	<form class="navbar-form1 input-group" id="search1" onsubmit="return false" style="position:absolute;">
		<input type="text" id="nameSearch" name="nameSearch" class="form-control" onkeydown="search();" list="nameList" autocomplete="on"
			placeholder="请输入姓名...">
		<datalist id="nameList">
			<option></option>
		</datalist>
	</form>
	<button class="btn btn_form_search" data-toggle="button" id="bottom_search" 
	style="position:absolute; right:48px; z-index:10;outline: none;border-radius: 0 4px 4px 0;background-color: yellowgreen;" 
	onclick="buttonClick()">确定</button>
</div>
```

## 搜索框样式
![](/img/posts_img/20201122193701509_16268.png)

## 实现功能
监听汉字输入，当输入中文时，在输入阶段(即输入法拼音部分)不执行函数，在完成中文拼写并输入到 input 框中后执行绑定函数。
未完成输入时状态如下：
![](/img/posts_img/20201122193911660_25674.png)
完成输入状态如下：
![](/img/posts_img/20201122193934278_3961.png)

## jQuery 事件绑定失效原因
JQuery事件绑定不生效，大概分两种情况。

1. **绑定事件在未加载完成之前：** DOM 元素在未加载完成之前，通过`$("...").on()`方法进行事件绑定，由于此时未加载完成(页面加载的异步性)，实际上`$("...")`是一个空数组，所以最终的结果是未对任何元素进行事件绑定。
2. **绑定事件后移除了元素重新加入：** DOM 元素首先通过JQuery 的方法被创建出来，然后被加入到 body 中，然后绑定事件，之后从 body 中移除，然后在加入 body 中，此时点击也不会生效，这是因为，在从文档中移除的时候，JQuery 会自动把绑定的事件移除掉了，然后在加入的时候，事件绑定已经不存在了。

## 解决方法
针对**在DOM元素未加载完成之前绑定事件导致绑定失效的问题**，通常把事件绑定放在 DOM 元素加载完成之后即可，jQuery的做法是用`$(function(){})`包裹事件绑定操作，例如上例：
```
$(function (){
    $('#nameSearch').on('compositionend', event=>{
        console.log(1);
        let val = $('#nameSearch').val();
        if(isNaN(Number(val))){
            debounce(tooltips, 2000)();
        }
    });
})
```
针对**绑定事件后移除元素后重新加入导致绑定失效的问题**，一般来说考虑 delegate 的方式，这通常针对某些元素存在频繁的添加、移除、再添加的操作。
```
$(document.body).delegate('#button2','click',function(){
    alert("button2 clicked");
});
```

---
# 补充
## 关于 jQuery 的 $(function() {})
`$(function() {})` 是`$(document).ready(function(){})`的简写。
该函数在 DOM 元素加载完毕之后执行。

**什么是 DOM ？**
DOM 就是一个 html 页面的标签树:
![](/img/posts_img/20201122195517212_4963.png)
当页面所有的 html 标签（包括图片等）都加载完了，即浏览器已经响应完了，DOM 即完成了加载操作。DOM 在第一次页面加载完毕后，就在内存里了，无论后面怎么局部修改 html 页面，都只是对内存中的 DOM 树进行修改，而 DOM在第一次页面加载后就已经加载完毕了。所以后面 js文件（动态加载或者 head 中加载）再使用到 `$(function() {})` 函数肯定会执行的。