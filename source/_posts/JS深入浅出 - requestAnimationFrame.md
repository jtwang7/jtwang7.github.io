---
title: JS深入浅出 - requestAnimationFrame
date: 2021-05-08 11:53
tags: 
- JavaScript
- 面试
categories: JavaScript
---

# JS深入浅出 - requestAnimationFrame

参考文章:
[探讨requestAnimationFrame](https://github.com/sisterAn/blog/issues/30)
[深入理解HTML5定时器requestAnimationFrame](https://segmentfault.com/a/1190000017332455)
**JavaScript高级程序设计(第4版) P549-551**

## 1. JS 动画
早期 JS 定时动画：主要通过 setTimeout 和 setIntarval 实现。
HTML5 出现后：又出现了两种实现动画的方式，1. CSS 动画（transition、animation）2. H5的 canvas 实现。
与此同时，HTML5 还提供了一个专门用于请求动画的 API `requesetAniamtionFrame()`，统一了 DOM 动画、canvas动画、svg动画、webGL动画等的**刷新机制**。

## 2. requestAnimationFrame(callback)
### 2.1 定义
告知浏览器在**下一次重绘前**，调用其回调函数来更新动画。
```
window.requestAnimationFrame(callback)
```
* callback：下一次重绘之前更新动画帧所调用的函数。callback仅接收一个固定参数，为DOMHighResTimeStamp参数，表示`requestAnimationFrame()`开始执行回调函数的时刻。
* 返回值：一个 long 类型整数，标记本次回调任务，可将该值传给 cancelAnimationFrame() 以取消本次回调对应的重绘任务。
### 2.2 内部执行机制
* 首先判断 `document.hidden` 属性是否可见（true），可见状态下才能继续执行以下步骤。
* 浏览器**清空**回调队列中的动画函数。
* `requestAnimationFrame()` 将回调函数追加到**动画帧请求回调函数列表**的末尾。
> 注意：执行 `requestAnimationFrame(callback)` 不会立即调用 callback 回调函数，只是将其放入**动画帧请求回调函数队列**(专门存放动画帧的回调队列，与其他回调队列独立)而已，同时注意，每个 callback回调函数都有一个 cancelled 标志符，初始值为 false，并对外不可见。
* 当页面可见并且动画帧请求callback回调函数列表不为空时，浏览器会定期将这些回调函数加入到浏览器 UI 线程的队列中（由系统来决定回调函数的执行时机）。
> 当浏览器执行这些 callback 回调函数的时候，会判断每个元组的 callback 的cancelled标志符，只有 cancelled 为 false 时，才执行callback回调函数(若被 `cancelAnimationFrame()` 取消了，对应 callback 的 cancelled 标识符会被置为 true)。
### 2.3 总结
* callback 实际上就是一帧动画的回调实现，`requestAnimationFrame()` 只会执行一次， 一次只能向回调队列中推入一个回调函数，因此实现动画需要通过递归调用`requestAnimationFrame()`实现。
```
function myCallback(time) {
  console.log(time); // requestAnimation() 的唯一传入参数
  // 动画没有执行完，则递归渲染
  if (count < 5) {
    count++;
    // 渲染下一帧
    rafId = window.requestAnimationFrame(requestAnimation);
  }
}
// 渲染第一帧
window.requestAnimationFrame(myCallback);
```
* `requestAnimationFrame()` 的回调函数触发时间是在浏览器下一次重绘之前，而浏览器大约每秒重绘60次，因此动画帧会在大约每16.6ms后执行一次。
> 大多数电脑显示器的刷新频率是60Hz，大概相当于每秒钟重绘60次。大多数浏览器都会对重绘操作加以限制，不超过显示器的重绘频率，因为即使超过那个频率用户体验也不会有提升。因此，最平滑动画的最佳循环间隔是1000ms/60，约等于16.6ms。
* `cancelAnimationFrame()` 只取消对应请求 ID 的重绘任务，内部实现是将请求 ID 标记的回调函数的 cancelled 标识符置为 true，以此让浏览器忽略并跳过该回调函数的执行。
## 3. 特点
### 3.1 定时动画存在的问题
* setTimeout / setInterval 不能保证回调的运行时刻：计时器只能保证何时将回调添加至浏览器的回调队列(宏任务)，不能保证回调队列的运行时间，假设主线程被其他任务占用，那么回调队列中的动画任务就会被阻塞，而不会按照原定的时间间隔刷新绘制。
* setTimeout / setInterval 计时不精确：不同浏览器的计时器精度都存在误差，此外浏览器会对切换到后台或不活跃标签页中的计时器进行限流，导致计时器计时误差。
* setTimeout / setInterval 在后台运行增大 CPU 开销：当标签页处于非活跃状态，计时器仍在执行计时工作，同时刷新动画效果，增大了 CPU 开销。(现阶段浏览器对此做了优化，如 FireFox/Chrome 浏览器对定时器做了优化：页面闲置时，如果时间间隔小于 1000ms，则停止定时器，与requestAnimationFrame行为类似。如果时间间隔>=1000ms，定时器依然在后台执行)
### 3.2 requestAnimationFrame 动画刷新机制的特点
1. requestAnimationFrame 采用**系统时间间隔**来执行回调函数，保持最佳绘制效率，不会因为间隔时间的过短，造成过度绘制，增加页面开销，也不会因为间隔时间过长，造成动画卡顿，不流程，影响页面美观。
> requestAnimationFrame的基本思想：让页面重绘的频率和刷新频率保持同步，即每 1000ms / 60 = 16.7ms执行一次。由于每次执行动画帧回调是由浏览器重回频率决定的，因此不需要像 setTimeout 那样传递时间间隔，而是浏览器通过系统获取并使用显示器刷新频率。
2. requestAnimationFrame 自带节流功能，例如在某些高频事件（resize，scroll 等）中，requestAnimationFrame 依据系统时间间隔来调用回调，可以防止在一个刷新间隔内发生多次函数执行。
3. requestAnimationFrame 延时效果是精确的，即在每次页面重绘前必会清空一次动画帧回调队列。(setTimeout 任务被放进异步队列中，只有当主线程上的任务执行完以后，才会去检查该队列的任务是否需要开始执行，造成时间延时)。
4. requestAnimationFrame 会把每一帧中的所有DOM操作集中起来，在一次重绘或回流中完成。
5. setTimeout 的执行只是在内存中对图像属性进行改变，这个改变必须要等到下次浏览器重绘时才会被更新到屏幕上。如果和屏幕刷新步调不一致，就可能导致中间某些帧的操作被跨越过去，直接更新下下一帧的图像，即**掉帧**。使用 requestAnimationFrame 执行动画，最大优势是能保证动画帧回调队列中的回调函数在屏幕每一次刷新前都被执行一次，然后将结果一起重绘到浏览器页面，这样就不会引起丢帧，动画也就不会卡顿。
6. `requestAnimationFrame()` 只有当标签页处于活跃状态是才会执行，当页面隐藏或最小化时，会被暂停，页面显示，会继续执行，节省了 CPU 开销。早期浏览器会对切换至后台或不活跃的标签页中的计时器执行限流，导致计时器时间不精确，此外计时器在后台仍会进行计时工作，执行动画任务，此时刷新动画是完全没有意义的。

