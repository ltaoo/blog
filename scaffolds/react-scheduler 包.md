---
title: React Scheduler 包
date: 2020/04/13
---



`React` 源码中，存在「调度」这一说法，即对「何时进行更新」进行安排，因为有些更新优先级高，需要立刻进行，有些优先级较低，所以可以在优先级高的完成后，再进行更新。

`scheduler` 就是用来完成这项工作的库。

下面以实际例子来查看该库的工作过程。



<!--more-->



```js
class App extends React.Component {
  render() {
    
  }
}

ReactDOM.render(<React.unstable_ConcurrentMode><App /></React.unstable_ConcurrentMode>);
```



在不使用 `ConcurrentMode` 组件时，所有更新都是「同步」的，不存在优先级问题，所以为了产生优先级，就需要将组件使用 `ConcurrentMode` 包裹。

在点击按钮后，会进行更新，首先会为该更新计算一个优先级，假设为 5。由于是在用户交互时触发更新，所以是「批量更新」，调用 `performSyncWork`，该方法调用 `performWork(Sync, false)`，这表示，会先进行一次优先级最高的更新，但是由于产生的更新优先级不是「最高」的，所以结束这次更新，将产生的更新调用 `scheduleCallbackWithExpirationTime` 进行调度。

该方法会调用



```js
scheduler.unstable_scheduleCallback(performAsyncWork, { timeout: timeout });
```



这里就到了 `scheduler` 包中，可以理解为，调用了 



```js
setTimeout(performAsyncWork, 0);
```

