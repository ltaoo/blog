---
title: setState 是同步还是异步
date: 2020/04/13
categories:
    - React
---



> 以下内容均基于 16.8.6 版本。

`setState` 是同步还是异步，这个问题很多人讨论过，各种说法都有，面试时也经常会问到，那它到底是同步还是异步呢？

我认为，它既是同步的，也是异步的。

<!--more-->



首先来说说我们是怎么判断它是同步还是异步，往往是在 `setState` 后访问 `state` 来确定，所以下面这个例子很常见

```js
class App extends React.Component {
  constructor(props) {
    super(props);
    
    this.state = {
      count: 0,
    };
  }
  handleClick = () => {
    // 1、直接调用
    this.update();
    // 2、包裹在 setTimeout 内调用
    setTimeout(() => {
      this.update();
    }, 0);
  }
  
  update = () => {
    this.setState({
      count: 1,
    });
    console.log(this.state.count);
    this.setState({
      count: 2,
    });
    console.log(this.state.count);
  }
  render() {
    const { count } = this.state;
    
    return <p onClick={this.handleClick}>COUNT: {count}</p>;
  }
}
```



第一种情况两次打印结果都是 0；第二种分别打印 1 和 2；



## 批量更新所以看起来像异步实际还是同步

大部分博客就会提到「批量更新」这个概念，可以理解为在一次同步任务内，`setState` 会「合并」到一起进行更新，但是 `setTimeout` 不在同步任务内，所以不进行「合并」，而是直接依次执行（或者说依次更新）。

用伪代码来说明是这样的



```js
let isBatchingUpdates = false;
let innerState = {};

// 模拟 this.state
let state = {};

// 模拟 setState 方法
function setState(nextState) {
  // 把 nextState 保存起来，实际源码是保存在了 fiber 上
  innerState = nextState;
  // 真实代码还有很多方法，下面都是省略了
  requestWork();
}
// 更新的入口
function requestWork() {
  // 延迟执行真正的更新
  if (isBatchingUpdates) {
    return;
  }
  // 真正的更新
  performWork();
}
function performWork() {
  // 假装这是在进行更新，用之前保存的 nextState 替换掉当前的 state
  state = innerState;
}

// 用来执行 handleClick 方法的方法，fn 就是 handleClick
function batchedUpdates(fn) {
  isBatchingUpdates = true;
  fn();
  
  isBatchingUpdates = false;
  performWork();
}

// 测试用例
function handleClick() {
  // 1、
  setState(1);
  console.log(state);
  setState(2);
  console.log(state);
  
  // 2、
  setTimeout(() => {
    setState(1);
    console.log(state);
    setState(2);
    console.log(state);
  }, 0);
}
batchedUpdates(handleClick);
```



可以发现效果和最开始的例子是相同的。

单纯从代码来说，这应该是同步的，因为没有 `setTimeout` 或者其他什么异步方法介入。那么是否表示，`setState` 就是同步的呢？

并不是，代码可以有多个条件分支，那为什么不能在「某种情况」下使用 `setTimeout` 呢？



## 使用 ConcurrentMode 时 setState 是异步

如果把 `App` 组件用 `React.unstable_ConcurrentMode` 组件包裹，即使在 `setTimeout` 内调用，两次打印的结果还是 0 和 0，即和第一种情况的结果一致。

[实际演示代码](https://codesandbox.io/s/setstate-is-sync-or-asnyc-01c4i?file=/src/App.js)

这是因为触发了异步逻辑，继续用上面伪代码来说明会是这样的



```js
// 省略相同的代码

let isConcurrentMode = true;
function requestWork() {
  // 延迟执行真正的更新
  if (isBatchingUpdates) {
    return;
  }
  // 实际代码完全不是这样的！！！这里只是为了说明
  if (isConcurrentMode) {
    scheduleCallback(performWork);
    return;
  }
  // 真正的更新
  performWork();
}

function scheduleCallback(fn) {
  setTimeout(fn, 0);
}
```



当然实际代码要复杂得多，并且 `scheduleCallback` 的实现其实是在 `scheduler` 包内的。

https://github.com/facebook/react/blob/master/packages/scheduler/src/Scheduler.js#L295



## 结论

所以，`setState` 是同步还是异步的呢？我认为

它既是同步也是异步，视情况而定。