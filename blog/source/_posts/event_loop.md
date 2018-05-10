---
title: 浏览器和Node事件循环的区别
---

> **事件循环**，是 js 中老生常谈的一个话题了，而在浏览器和 Node 中的事件循环执行机制也不相同，浏览器的事件循环是在 HTML5 中定义的规范，而 Node 中则是由 libuv 库实现，不可以混为一谈。

先看一个简单的事件循环笔试题：
```javascript
function sleep(time) {
    let startTime = new Date();
    while (new Date() - startTime < time) {}
    console.log('<--Next Loop-->');
}

setTimeout(() => {
    console.log('timeout1');
    setTimeout(() => {
        console.log('timeout3');
        sleep(1000);
    });
    new Promise((resolve) => {
        console.log('timeout1_promise');
        resolve();
    }).then(() => {
        console.log('timeout1_then');
    });
    sleep(1000);
});
     
setTimeout(() => {
    console.log('timeout2');
    setTimeout(() => {
        console.log('timeout4');
        sleep(1000);
    });
    new Promise((resolve) => {
        console.log('timeout2_promise');
        resolve();
    }).then(() => {
        console.log('timeout2_then');
    });
    sleep(1000);
});
```
在不同的环境中，输出的结果也是不同的：
- 浏览器中的输出：
```
timeout1
timeout1_promise
<--Next Loop-->
timeout1_then
timeout2
timeout2_promise
<--Next Loop-->
timeout2_then
timeout3
<--Next Loop-->
timeout4
<--Next Loop-->
```
- Node 环境中的输出：
```
timeout1
timeout1_promise
<--Next Loop-->
timeout2
timeout2_promise
<--Next Loop-->
timeout1_then
timeout2_then
timeout3
<--Next Loop-->
timeout4
<--Next Loop-->
```

接下来我们就看看浏览器和 Node 中时间循环的区别是什么。

## 1. 任务队列
### 浏览器环境
浏览器环境下的 **异步任务** 分为 **宏任务(macroTask)** 和 **微任务(microTask)**：
- **宏任务(macroTask)**：script 中代码、setTimeout、setInterval、I/O、UI render；
- **微任务(microTask)**： Promise、Object.observe、MutationObserver。

当满足执行条件时，**宏任务(macroTask)** 和 **微任务(microTask)** 会各自被放入对应的队列：**宏队列(Macrotask Queue)** 和 **微队列(Microtask Queue)** 中等待执行。

### Node 环境
在 Node 环境中 **任务类型** 相对就比浏览器环境下要复杂一些：
- **microTask**：微任务；
- **nextTick**：`process.nextTick`；
- **timers**：执行满足条件的 setTimeout 、setInterval 回调；
- **I/O callbacks**：是否有已完成的 I/O 操作的回调函数，来自上一轮的 poll 残留；
- **poll**：等待还没完成的 I/O 事件，会因 **timers** 和超时时间等结束等待；
- **check**：执行 setImmediate 的回调；
- **close callbacks**：关闭所有的 closing handles ，一些 onclose 事件；
- idle/prepare 等等：可忽略。

因此，也就产生了执行事件循环相应的任务队列 **Timers Queue**、**I/O Queue**、**Check Queue** 和 **Close Queue**。

## 2.执行过程
### 浏览器环境
先执行`<script>`中的同步任务，然后所有微任务，一个宏任务，所有微任务，一个宏任务......
- 1. 执行完主执行线程中的任务；
- 2. 取出 **Microtask Queue** 中任务执行直到清空；
- 3. 取出 **Macrotask Queue** 中一个任务执行；
- 4. 重复 2 和 3 。

需要 **注意** 的是：
- 在浏览器页面中可以认为初始执行线程中没有代码，每一个`<script>`中的代码是一个独立的 **task** ，即会执行完前面的`<script>`中创建的 **microTask** 再执行后面的`<script>`中的同步代码；
- 如果 **microTask** 一直被添加，则会继续执行 **microTask** ，“卡死” **macroTask**；
- 部分版本浏览器有执行顺序与上述不符的情况，可能是不符合标准或 js 与 html 部分标准冲突；
- Promise 的`then`和`catch`才是 **microTask** ，本身的内部代码不是；
- 个别浏览器独有API未列出。
 
### Node 环境
#### 循环之前
在进入第一次循环之前，会先进行如下操作：
- 同步任务；
- 发出异步请求；
- 规划定时器生效的时间；
- 执行`process.nextTick()`。

#### 开始循环
循环中进行的操作：
- 清空当前循环内的 **Timers Queue**，清空 **NextTick Queue**，清空 **Microtask Queue**；
- 清空当前循环内的 **I/O Queue**，清空 **NextTick Queue**，清空 **Microtask Queue**；
- 清空当前循环内的 **Check Queue**，清空 **NextTick Queue**，清空 **Microtask Queue**；
- 清空当前循环内的 **Close Queue**，清空 **NextTick Queue**，清空 **Microtask Queue**；
- 进入下轮循环。

可以看出，**nextTick** 优先级比 Promise 等 **microTask** 高，`setTimeout`和`setInterval`优先级比`setImmediate`高。

#### 注意
在整个过程中，需要 **注意** 的是：
- 如果在 **timers** 阶段执行时创建了`setImmediate` 则会在此轮循环的 **check** 阶段执行，如果在 **timers** 阶段创建了`setTimeout`，由于 **timers** 已取出完毕，则会进入下轮循环，**check** 阶段创建 **timers** 任务同理；
- `setTimeout`优先级比`setImmediate`高，但是由于`setTimeout(fn,0)`的真正延迟不可能完全为 0 秒，可能出现先创建的`setTimeout(fn,0)`而比`setImmediate`的回调后执行的情况。

## 总结
事件循环在 **浏览器** 和 **Node** 中的区别很容易被人忽视，执行顺序整理如下：

浏览器环境下：
```javascript
while (true) {
    宏任务队列.shift();
    微任务队列全部任务();
}
```
Node 环境下：
```javascript
while (true) {
    loop.forEach((阶段) => {
        阶段全部任务();
        nextTick全部任务();
        microTask全部任务();
    });
    loop = loop.next;
}
```
个人觉得比较清晰了，有什么问题可以私信讨论。

> 文章部分理论参考自：https://segmentfault.com/a/1190000013660033?utm_source=channel-hottest 