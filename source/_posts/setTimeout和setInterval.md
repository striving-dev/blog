---
title: setTimeout和setInterval
date: 2020-12-07 13:51:41
tags: JS基础
author: 李京阳
---

最近接到一个需求，开发一个倒计时组件，渲染后能够自动倒计时直到到达服务器下发的结束时间。纯展示类组件，暂时没有其他交互。

接到这个需求时，一开始是轻视的，但是在实现和测试阶段发现了很多问题，所以把倒计时相关的知识总结一下。

<!-- more -->
---

以下列出文章所有的知识点，如果对介绍的内容已经掌握了，可以快速跳过

1. [`setTimeout`、`setInterval`基础知识（参数、含义）](#基础介绍)
2. [嵌套调用`setTimeout`以达到setInterval效果](#Nested-setTimeout)
3. [为什么一定要“清除定时器”](#关于垃圾回收)
4. [延伸](#延伸)

## 基础介绍

### setTimeout

`const timerId = setTimeout(fun|code, [delay], [arg1], [arg2], ...)`

- `func|code`

  由于历史原因，传入字符串代码也是允许的，但是不建议这样做

- `delay`

  单位是ms，默认为0

- `arg1, arg2...`

  更多的参数，传递给参数1（参数1是一个函数）中的参数。IE9及以下不支持此参数

```javascript
function sayHi() {
  alert('Hello');
}

setTimeout(sayHi, 1000);

// 参数
function sayHi2(name) {
  alert(`hello, ${name}`);
}

setTimeout(sayHi2, 1000, 'John');

// 字符串参数
setTimeout("alert('Hello')", 1000);  // 不推荐

// 清除timeout
const timerId = setTimeout(sayH1, 1000);
const oldTimerId = timerId;
clearTimeout(timerId);
oldTimerId === timerId // true
```

### setInterval

`const timerId = setInterval(func | code, [delay], [arg1], [arg2], ...)`

使用方式和`setTimeout()`差不多，不同的地方就是`setInterval()`方法创建的异步任务会无休止的执行下去，除非主动调用`clearInterval()`

## Nested setTimeout

综上，如果我们想要实现以一定频率反复执行一段代码，我们可以使用`setInterval`来实现。除此之外，我们还有另一个选择：嵌套执行`setTimeout`：

```javascript
function tick() {
  console.log('tick');
  setTimeout(tick, 1000);  // nested call
}
let timerId = setTimeout(tick, 1000)
```

那么，这两种方式的区别是什么呢？

```javascript
setInterval(func, 100)

setTimeout(function tick() {
  func();
  setTimeout(tick, 100);
}, 100)
```

![setTimeout setInterval](https://cdn.ljybill.com/uPic/setTimeout%20setInterval.png)

这个图能够直观表现两者的差异，**在`setInterval`中，两个函数之间的实际间隔是小于100毫秒的**，这是因为，`func`代码执行的时间“占用”掉了一部分间隔时间。

如果`func`执行时间过长（比如在这个例子中超过了100ms），那么当func执行完毕后，调度器会立即执行下一个`func`代码，这样就会无休止的执行下去。

如上图，**Nested setTimeout就能够保证一个固定的时间**，这是因为我们会在一个`func`的最后去生成一个新的定时器。

### 优化

嵌套调用setTimeout的优势就在于能够灵活地调整延迟时间来达到更“丝滑”的效果，下面就看下升级版的“Nested setTimeout”

```javascript
const delay = 1000;
function run() {
  let count = 0;
  const startTime = Date.now();
  setTimeout(function tick() {
    // do something
    
    count++;
    const offset = Date.now() - (startTime + count * delay)
    setTimeout(tick, delay - offset)
  }, delay)
}
```

## 关于垃圾回收

我们平时在写定时器的时候，一定会听过这样一句话：“一定要记得及时调用`clearTimeout/Interval`清除定时器”。那么为什么要及时清定时器，清的到底是什么？下面讲下关于`setTimeout`和`setInterval`的垃圾回收机制。

当我们给`setTimeout`/`setInterval`传递函数时，引擎会创建一个内部的引用，并在调度器（scheduler）中存起来，以防被垃圾回收所收集，哪怕外部并没有其他的引用。

```javascript
// 这个函数会被保留到内存中，直到函数被调用了，也就是说函数调用之后，垃圾回收就能够回收此函数了
setTimeout(function() {...}, 100);

// 这个函数会被保留到内存中，直到执行了clearInterval
setInterval(function() {...}, 100);
```

这里有个副作用，如果`function`中使用依赖了外部的词法环境（俗称闭包），那么这些外部的变量也会被保留到内存中。**外部变量消耗的内存很可能比`function`本身消耗的内存多**，这也是为什么我们常说清除定时器。

## 为什么我的定时器会很慢

请注意，不管是setTimeout还是setInterval都不保证精确的delay时间，并且在某些情况下，差距会很大，比如：

- CPU超载了
- 浏览器的标签页进入了background mode
- 设备进入节能模式（个人觉得也是限制了CPU的性能）

## 总结

- 方法`setTimeout(func, delay, ...args)`和`setInterval(func, delay, ...args)`能够一次或周期性地执行一个`function`在`delay`毫秒后
- 如果想取消计时器，我们可以调用`clearTimeout`/`clearInterval`，传递`setTimeout`/`setInterval`的返回值作为参数
- 嵌套执行`setTimeout`会比`setInterval`更灵活，能够及时调整`delay`到恰当的时间
- 记得及时清除计时器

## 延伸

学习以下内容，能够更深入了解JS的定时器运行机制。

- 微任务和宏任务

- requestAnimationFrame