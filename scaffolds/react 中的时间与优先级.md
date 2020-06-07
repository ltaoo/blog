---
title: React 中的时间与优先级
date: 2020/04/13
---



在 `react` 中，每次调用 `setState` 就表示产生了一个更新，同时会为该更新计算一个时间，在源码内用 `expirationTime` 表示，该值的大小同时还表达了「优先级」的含义，该值越大，优先级越高。

这里的优先级指「进行更新」的先后顺序，如果有 `Child1` 和 `Child2` 两个组件，`Child1` 有一个更新，优先级为 10；`Child2` 有一个更新，优先级为 5，那么 `Child1` 会先完成更新，再更新 `Child2`。



## 计算时间的方法



```js
var currentTime = requestCurrentTime();
var expirationTime = computeExpirationForFiber(currentTime, current);
```



可以看到，需要先获取「当前时间」，在根据当前时间与产生更新的组件（current）返回 `expirationTime`。



### requestCurrentTime

```js
function requestCurrentTime() {
  // 调用 performWorkOnRoot 方法
  if (isRendering) {
    return currentSchedulerTime;
  }
  // 获取链结构中优先级最高的 root
  findHighestPriorityRoot();
  if (nextFlushedExpirationTime === NoWork || nextFlushedExpirationTime === Never) {
		// 当前没有任何更新
    recomputeCurrentRendererTime();
    currentSchedulerTime = currentRendererTime;
    return currentSchedulerTime;
  }
  // 如果已经存在更新
  return currentSchedulerTime;
}
```



```js
function recomputeCurrentRendererTime() {
  var currentTimeMs = scheduler.unstable_now() - originalStartTimeMs;
  currentRendererTime = msToExpirationTime(currentTimeMs);
}
```



```js
  if (typeof performance === 'object' && typeof performance.now === 'function') {
    exports.unstable_now = function () {
      return performance.now();
    };
  } else {
    var _initialTime = _Date.now();

    exports.unstable_now = function () {
      // 这里表示，自该文件加载，到 now 方法调用的时间间隔毫秒数
      return _Date.now() - _initialTime;
    };
  }
```

`performance.now` 方法可以查看 https://developer.mozilla.org/zh-CN/docs/Web/API/Performance/now

简单来说就是获取页面初始化到调用该方法时的时间差，以毫秒数表示，并且该值是在不断累加的，刷新页面会重置该值。



```js
var maxSigned31BitInt = 1073741823; // 2 ** 30 - 1

var Sync = maxSigned31BitInt;

var UNIT_SIZE = 10;
var MAGIC_NUMBER_OFFSET = maxSigned31BitInt - 1;

// 1 unit of expiration time represents 10ms.
function msToExpirationTime(ms) {
  // Always add an offset so that we don't clash with the magic number for NoWork.
  return MAGIC_NUMBER_OFFSET - (ms / UNIT_SIZE | 0);
}
```

所以上面 `currentSchedulerTime` 和 `currentRendererTime` 是什么大概心里就有个数了。这个 `current` 表达的就是「当前距打开页面的时间差」，就和我们 2020/04/13 12:00:00 表达的也是距离 1970/01/01 00:00:00 的时间差一样，我们也把这个时间称为「现在时间」。



### computeExpirationForFiber

获取到当前时间后，还需要根据产生更新的组件来计算出最终的这个 `expirationTime`，其实逻辑很简单，在不使用 `ConcurrentMode` 组件包裹的前提下，会返回一个常量 `Sync`，即 1073741823。



```js
function computeExpirationForFiber(currentTime, fiber) {
  var priorityLevel = scheduler.unstable_getCurrentPriorityLevel();

  var expirationTime = void 0;
  if ((fiber.mode & ConcurrentMode) === NoContext) {
    // Outside of concurrent mode, updates are always synchronous.
    expirationTime = Sync;
  }
  // ...
  
  return expirationTime;
}
```



### computeExpirationBucket

其他情况，会使用该方法来计算出 `expirationTime`



所以，`expirationTime` 是一个从 1073741823 不断变小的值，随着应用打开时间的变长，该时间越来越小，打开 10s，该值减小 1000，所以当页面打开时间为 1073742s 即 12.4d 后，该值变为 0。当然绝大部分情况都不会打开这么长时间。

