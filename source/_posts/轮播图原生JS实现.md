---
title: codeWheels 专栏 - 轮播图
date: 2021-04-17 17:59
tags: 
- JavaScript
- ES6
- codeWheels
categories: codeWheels
excerpt: 基于纯 JavaScript (ES6) 构造 UI 轮播图轮子, 简要介绍了轮播图的一些核心实现步骤, 项目涉及知识点主要包括 JavaScript 操作 DOM 元素, 事件监听以及浏览器重绘回流等;
---

前期资料参考:
* [手把手教你用原生JavaScript造轮子（二）——轮播图](https://segmentfault.com/a/1190000015976690)
* [手把手教你用原生JavaScript造轮子（三）——项目升级&填坑&重写组件](https://segmentfault.com/a/1190000022308884)
* [参考项目github地址](https://github.com/csdoker/csdwheels/blob/master/src/es6/carousel/carousel.js)

> 注: 将 `github.com` 改为 `github1s.com` 有奇效!

# 轮播图
**github地址:** [Carousel](https://github.com/jtwang7/codeWheels/tree/master/Carousel)

## 核心实现
### 检查图片是否加载完全
```
/ 每隔一段时间检查图片是否加载完全
let checkInterval = 50;
let checkTimer = setInterval(() => {
  // 若全部图片加载成功, 则开始构建轮播动画
  if (this.isImagesComplete) {
    // 清除检测定时器
    clearInterval(checkTimer);
    // 初始化轮播
    this.initCarousel();
    // 初始化圆点
    this.initDots();
    // 初识化箭头
    this.initArrows();
  }
}, checkInterval)
```
```
isImagesComplete() {
  let complete = 0;
  // children 属性: "红宝书第四版 P456-457"
  for (let i = 0; i < this.carouselWrap.children.length; i++) {
    // imgObject.complete 属性: <img> 标签会产生一个 Image 对象, 其 complete 属性返回一个布尔值, 表示浏览器是否已完成对图像的加载
    if (this.carouselWrap.children[i].complete) {
      complete++;
    }
  }
  return complete === this.carouselWrap.children.length;
}
```

### 轮播列表添加首尾过渡元素
```
// 获取轮播列表
getCarouselWrap() {
  // 创建文档片段作为临时仓库
  // DocumentFragment 类型: "红宝书第四版 P424-425"
  let fragment = document.createDocumentFragment();
  // Element 类型 (元素创建及属性设置): "红宝书第四版 P417-419"
  let imgTemplate = document.createElement('img');
  this.options.carouselWrapImages.forEach((imgUrl, idx) => {
    let imgElement = imgTemplate.cloneNode(false);
    imgElement.setAttribute('class', Carousel.CLASS.CAROUSEL_IMAGE);
    imgElement.setAttribute('src', imgUrl);
    imgElement.setAttribute('alt', idx + 1);
    fragment.appendChild(imgElement)
  })
  // 轮播列表无缝循环的关键: 首尾过渡元素
  // Element Traversal API: "红宝书第四版 P447"
  let first = fragment.firstElementChild.cloneNode(true);
  let last = fragment.lastElementChild.cloneNode(true);
  fragment.insertBefore(last, fragment.firstElementChild);
  fragment.appendChild(first);

  // 创建轮播列表元素
  let carouselWrap = document.createElement('div');
  carouselWrap.setAttribute('class', Carousel.CLASS.CAROUSEL_WRAP);
  carouselWrap.setAttribute('id', Carousel.ID.CAROUSEL_WRAP.substring(1));

  // 转移临时文档片段内容至轮播列表
  carouselWrap.appendChild(fragment);
  this.setWidth(carouselWrap, carouselWrap.children.length * this.options.carouselWidth);

  return carouselWrap;
}
```

### 动画效果实现
主要依靠 `style.left` 和 `requestAnimationFrame()` 实现; 此外在完成动画后需要处理过渡元素;
```
// 动画效果实现
switchAnimation(targetLeft, offset) {
  // 标记当前处于动画状态
  this.isAnimationShow = true;
  let currentLeft = this.getLeft(this.carouselWrap);
  // requesetAnimationFrame() "红宝书第四版 P550-551"
  this.animationTimer = requestAnimationFrame(() => {
    if ((offset < 0 && currentLeft > targetLeft) || (offset > 0 && currentLeft < targetLeft)) {
      // 动画
      this.frameAnimation(targetLeft, offset)
    } else {
      // 动画完成, 清除动画计时器, 并依据情况重置轮播列表
      clearInterval(this.animationTimer);
      this.resetCarouselWrap(targetLeft, offset)
    }
  })
}

// 帧动画(每帧)
frameAnimation(targetLeft, offset) {
  // 图片偏移
  this.setLeft(this.carouselWrap, this.getLeft(this.carouselWrap) + offset);
  // 动画递归
  this.switchAnimation(targetLeft, offset);
}

resetCarouselWrap(targetLeft, offset) {
  // 处理过渡轮播片段
  if (this.isDotClick) {
    this.resetDotCarouselWrap(offset)
  } else {
    this.resetMoveCarouselWrap(targetLeft);
  }
  // 重置标记状态
  this.isDotClick = false;
  this.isAnimationShow = false;
}

resetDotCarouselWrap(offset) {
  // 涉及了浏览器回流知识点
  // 下述操作在浏览器中统一执行, 只渲染一次
  this.setLeft(this.carouselWrap, -this.options.carouselWidth * this.dotIndex)
  if (offset < 0) {
    this.carouselWrap.removeChild(this.currentNode.nextElementSibling);
  }
  if (offset > 0) {
    this.carouselWrap.removeChild(this.currentNode.previousElementSibling);
  }
}

resetMoveCarouselWrap(targetLeft) {
  if (targetLeft < -this.carouselCount * this.options.carouselWidth) {
    // 表明当前轮播处在最后一张过渡图片上, 需重置回第二张图片
    this.setLeft(this.carouselWrap, -this.options.carouselWidth);
  }
  if (targetLeft > -this.options.carouselWidth) {
    // 表明当前轮播处在第一张过渡图片上, 需重置回倒数第二张图片
    this.setLeft(this.carouselWrap, -this.carouselCount * this.options.carouselWidth);
  }
}
```

### 通过轮播圆点执行的动画
在轮播片段一侧插入目标副本, 实现动画后再重置到真正的目标, 并删除副本
```
dotChangeCarousel() {
  this.currentNode = this.carouselWrap.children[this.carouselIndex];
  this.targetNode = this.carouselWrap.children[this.dotIndex];
  if (this.dotIndex > this.carouselIndex) {
    // 在当前轮播片段右侧插入目标轮播片段的副本, 用于过渡
    this.carouselWrap.insertBefore(this.targetNode.cloneNode(true), this.currentNode.nextElementSibling);
    this.switchAnimation(
      this.getLeft(this.carouselWrap) - this.options.carouselWidth,
      -this.animationOffset
    )
  } 
  if (this.dotIndex < this.carouselIndex) {
    this.carouselWrap.insertBefore(this.targetNode.cloneNode(true), this.currentNode);
    this.setLeft(this.carouselWrap, -this.options.carouselWidth * (this.carouselIndex + 1));
    this.switchAnimation(
      this.getLeft(this.carouselWrap) + this.options.carouselWidth,
      this.animationOffset
    )
  }
}
```

## 存在的问题
1. 频繁的 DOM 操作, 性能低
2. 动画算法不利于维护与功能拓展

## 解决方案
实现轮播的动画效果, 不需要去计算每个子元素的序号以及对应的顺序变化, 因为不管哪个方向, 自始至终我们看到的都是两个子元素的移动效果, 所以只需要给这两个移动中的元素添加对应的样式即可;
具体实现思路: 使用 CSS3 的 transform 属性来控制子元素的位置变化, 配合transition添加过渡动画, 在 JS 代码中只需要在合适的时机添加对应的类名, 然后移出对应的类名;


# 知识点总结 
> 出现的页码对应"JavaScript 高级程序设计(第四版)"

1. 跨浏览器事件处理程序: "红宝书第四版 P498"
2. node.appendChild(): "红宝书第四版 P405-406"
3. document.querySelector(CSSDescription): "红宝书第四版 P445-446"
4. HTMLElement.style 属性及其设置方式(3种): 1. elem.style.width = value + 'px'; 2. elem.setAttribute('style', value + 'px'); 3. elem.style.cssText = \`width: ${value}px\`;
5. parseInt(): "红宝书第四版 P37"
6. DocumentFragment 类型: "红宝书第四版 P424-425"
7. Element 类型 (元素创建及属性设置): "红宝书第四版 P417-419"
8. Element Traversal API: "红宝书第四版 P447"
9. children 属性: "红宝书第四版 P456-457"
10. 检测标签页是否活跃
* visibilitychange 事件类型: https://developer.mozilla.org/zh-CN/docs/Web/API/Document/visibilitychange_event
* 标签页活跃性判断方法:
* document.hidden : https://developer.mozilla.org/zh-CN/docs/Web/API/Document/hidden
* document.visibilityState : https://developer.mozilla.org/zh-CN/docs/Web/API/Document/visibilityState
11. 定时器 setInterval: "红宝书第四版 P368-369"
12. requesetAnimationFrame() "红宝书第四版 P550-551"
13. element.innerHTML 属性: P452-453
14. 转义字符 `'&lt;'` 与 `'&gt;'`: 表示 '<' 和 '>'
15. 浏览器回流