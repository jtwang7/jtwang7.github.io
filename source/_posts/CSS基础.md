---
title: CSS基础
date: 2021-03-06 11:14
tags: CSS
categories: CSS
excerpt: 系统性学习CSS(层叠样式表)，学习资料来源 W3School 。
---

# CSS基础
参考资料：[W3School--CSS](https://www.w3school.com.cn/css/index.asp)

---
## CSS语法
**CSS规则集(rule-set)**：选择器 + 声明块
![](/img/posts_img/20210306111858285_13158.png)
实例：
```
p {
  color: red;
  font-size: 14px;
}
```
**重点：**
1. 选择器概念
2. 声明块内可包含多条声明语句，用 `;` 分隔。(区别于对象)
3. 声明语句以键值对(key-value)形式书写，值不需要指定为字符串形式。(区别于对象)

### CSS选择器
概念：指定设置样式的 HTML 元素
CSS选择器类型包括：
1. 简单选择器（根据名称、id、类来选取元素）
2. 组合器选择器（根据它们之间的特定关系来选取元素）
3. 伪类选择器（根据特定状态选取元素）
4. 伪元素选择器（选取元素的一部分并设置其样式）
5. 属性选择器（根据属性或属性值来选取元素）

#### 简单选择器
##### css元素选择器
作用：根据元素名称(HTML标签)选择 HTML 元素，所有指定的标签样式均会被更改。
实例：
```
img {
  display: flex;
}
```
##### css id选择器
作用：根据 HTML 元素的 id 属性来选择特定元素。元素的 id 在页面中是唯一的，因此 id 选择器用于选择一个唯一的元素。
语法：`#idname {css style;...}`
实例：(id 为 myidname 的标签被选中并修改样式)
```
#myidname {
  color: red;
}
```
##### css类选择器
作用：根据 HTML 元素的 class 属性来选择特定元素。
语法： `.classname {css style;...}`
实例1：(所有class名为myclassname的标签均被选中并修改样式)
```
.myclassname {
  color: red;
}
```
实例2：(h2标签中class名为myclassname的标签被选中并修改样式)
```
h2.myclassname {
  color: red;
}
```
##### css通用选择器
作用：选择页面上的所有的 HTML 元素。
语法：`* {css style;...}`
实例：
```
* {
  color: red;
}
```
##### css分组选择器
作用：将样式相同的 HTML 元素写在一起，缩减代码量。
语法：`name1, name2, ... {css style;...}`
实例：
```
h1, #idname, .classname {
  text-align: center;
  color: red;
}
```

#### 后续补充，敬请期待


## CSS使用
### 外部CSS
外部样式表可以在任何文本编辑器中编写，并且必须以 `.css` 扩展名保存。
可在所引入的页面下使用表内样式。
引用：
1. HTML 页面：在 `<head>` 部分的 `<link>` 元素内包含对外部样式表文件的引用。
2. 框架内：在 `<script>` 部分通过模块化导入 `.css` 文件

实例：
```
HTML：
<link rel="stylesheet" type="text/css" href="mystyle.css">

框架：
import "./xxx/xx/mystle.css";
```

**注意**：
1. 外部 `.css` 文件不应包含任何 HTML 标签。
2. 请勿在属性值和单位之间添加空格（例如 `margin-left: 20 px;`）。正确的写法是：`margin-left: 20px;`

### 内部CSS
仅在某一页面内使用定义的样式。
引用：
1. HTML 页面：在 `<head>` 部分的 `<style>` 元素中进行定义。
2. 框架内(以Vue为例)：在组件的 `<style>` 部分定义样式。

实例：
```
<style>
body {
  background-color: linen;
}

h1 {
  color: maroon;
  margin-left: 40px;
} 
</style>
```

### 行内CSS(内联样式)
为单个元素应用唯一的样式。
引用：行内样式在相关元素的 `style` 属性中定义
实例：
```
<h1 style="color:blue;text-align:center;"></h1>
<p style="color:red;"></p>
```
提示：
1. 行内样式失去了样式表的许多优点（通过将内容与呈现混合在一起）。请谨慎使用此方法。
2. 标签内属性值均用 `""` 包裹，内联样式传递给 `style` 属性时不需要 `{}` 包裹。

### CSS样式层叠优先级
优先级从高到低依次为：
1. 行内样式（在 HTML 元素中）
2. 外部和内部样式表（在 head 部分）
3. 浏览器默认样式

行内样式具有最高优先级，并且将覆盖外部和内部样式以及浏览器默认样式。


## CSS颜色
## CSS背景
## CSS边框

## CSS边距
### 外边距(margin)
CSS margin 属性用于在任何定义的边框(包括没有定义边框)之外，为元素周围创建空间。
**margin 属性**：
1. margin-top
2. margin-right
3. margin-bottom
4. margin-left

**margin 属性值(允许负值)**：
1. `auto` - 浏览器来计算外边距
2. `length` - 以 px、pt、cm 等单位指定外边距
3. `%` - 指定以包含元素宽度的百分比计的外边距
4. `inherit` - 指定应从父元素继承外边距

margin 属性简写：
```
margin: 上 右 下 左
margin: 上 右左 下
margin: 上下 右左
margin: 上下左右
```

auto 属性值的特殊用途(水平居中)：
`margin: auto`
元素将占据指定的宽度，并且剩余空间将在左右边界之间平均分配。

#### 外边距合并
当两个垂直外边距相遇时，它们将形成一个外边距。合并后的外边距的高度等于两个发生合并的外边距的高度中的较大者。
具体参照：[外边距合并](https://www.w3school.com.cn/css/css_margin_collapse.asp) ，此处不做细致讲解

### 内边距(padding)
CSS padding 属性用于在任何定义的边界内的元素内容周围生成空间。即向内填充。
**padding 属性**：
1. padding-top
2. padding-right
3. padding-bottom
4. padding-left

**padding 属性值(允许负值 | 没有 auto 属性值)**：
1. `length` - 以 px、pt、cm 等单位指定外边距
2. `%` - 指定以包含元素宽度的百分比计的外边距
3. `inherit` - 指定应从父元素继承外边距

padding 属性简写：
```
padding: 上 右 下 左
padding: 上 右左 下
padding: 上下 右左
padding: 上下左右
```

## CSS高度/宽度
height 和 width 属性用于设置元素的高度和宽度。height 和 width 属性不包括内边距、边框或外边距。
标准盒子模型：
![](/img/posts_img/20210306140343369_9779.png)

IE盒子模型：
![](/img/posts_img/20210306140404202_28129.png)

**height | width 属性**：
1. `auto` - 默认。浏览器计算高度和宽度。
2. `length` - 以 px、cm 等定义高度/宽度。
3. `%` - 以包含块的百分比定义高度/宽度。
4. `initial` - 将高度/宽度设置为默认值。
5. `inherit` - 从其父值继承高度/宽度

**注意**：
在标准盒子模型中，height 和 width 属性不包括内边距、边框或外边距！它们设置的是元素的内边距、边框和外边距内的区域的高度/宽度！




