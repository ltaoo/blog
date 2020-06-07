---
title: 一次「错误」的 Promise 实现 - 1
categories: JavaScript
tags:
- JavaScript
- Promise
date: 2018/2/11
---

`Promise`在日常中经常用到，并且也能够熟练使用：

```javascript
new Promise((resolve) => {
  console.log('a')
  resolve('b')
  console.log('c')
}).then((data) => {
  console.log(data)
});
```

上面代码会依次打印`a、c、b`，对此我们都毫无疑义。

但是为什么呢？我们能自己实现一个`Promise`库吗？

<!--more-->

## 「错误」的实现

不参考任何教程、代码，只从上面的结果推理出`Promise`是什么样的，或许正确，或许错误。但这也是最有趣的地方，完成后可以与其他`Promise`库对照，差距究竟差在哪。

> 读者，也就是你，如果真的想对`Promise`深入了解，更正确的做法也是自己手撸一个，而不是从各种「二手信息」（包括此文）中学习。

话不多说，开始探索之旅。

### Promise 类

首先分析上面的代码，很容易看出是先调用`new`关键字生成`Promise`实例，并执行传入的参数，假设形参是`fn`。

然后调用得到的实例上的`then`方法，也传入一个参数，该参数也会被调用，假设该形参为`resolved`。

所以`Promise`类应该是这样的：

```javascript
// 用 FakePromise 避免覆盖原生的 Promise
class FakePromise {
    constructor(fn) {
    }
    
    then(resolved) {  
    }
}
```

### constructor

而我们又知道，在调用`new`关键字时，马上就会打印`a、c`，这表示传入的参数`fn`被立即调用了。

```javascript
constructor(fn) {
    fn();
}
```

并且该函数有形参`resolve`会被调用时使用，所以在调用`fn`时还要传入实参`resolve`。

```javascript
constructor(fn) {
    fn(resolve);
}
```

那么问题来了，这个`resolve`实参是哪里来的？或许是在全局的一个函数？那么试试看好了：

```javascript
function resolve() {
    // ...
}

class FakePromise {
    constructor(fn) {
        fn(resolve);
    }
    
    then(resolved) {
        
    }
}
```

使用开始的实例测试一下：

```javascript
new FakePromise((resolve) => {
    console.log('a');
    resolve('b');
    console.log('c');
})
    .then((res) => {
        console.log(res);
    });
    
// a
// c
```

bingo！没有报错就是好的开始，好的开始就是成功了一半~~~

### then 方法

继续，我们从开始的例子打印的结果`a、c、b`可以知道在调用`then`方法后，也会调用传入的参数`resolved`，并且还接受一个参数，该参数为`constructor`内`resolve`的实参。

```javascript
class FakePromise {
    constructor(fn) {
        fn(resolve);
    }
    
    then(resolved) {
        resolved(params);
    }
}
```

![resolved的参数即resolve的实参](http://oyy3cbpm3.bkt.clouddn.com/15182655749667.jpg)

整体观察一下，在「1」调用了「2」，此时「2」是可以拿到`b`这个参数的，是否可以将参数保存起来作为全局变量，在「3」处使用呢？


```javascript
let globalParam = null;
function resolve(param) {
    globalParam = param;
}

class FakePromise {
    constructor(fn) {
        fn(resolve);
    }
    
    then(resolved) {
        resolved(globalParam);
    }
}
```

再实际测试一下，成功打印`a、c、b`！！实际的`Promise`就这么简单吗？

用复杂些的例子测试：

```javascript
new FakePromise((resolve) => {
    console.log('a');
    // resolve('b');
    setTimeout(() => {
        resolve('b');
    }, 1000);
    console.log('c');
})
    .then((res) => {
        console.log(res);
    });
// a
// c
// null
```

结果是`'a'、'c'、null`，此时流程是这样的（按执行顺序）：

![流程图](http://oyy3cbpm3.bkt.clouddn.com/15182662332911.jpg)

`resolved`在`resolve`前执行，导致`globalParam = param`没有执行，所以传入的是`null`。


### then 与 resolve 的顺序

问题出在哪里呢？

可以想到，必须要`resolve('b')`执行完后，才能调用`resolved`。

> resolved: hi，resolve，执行了吗？
> resolve: 还没呢，再等会，要不然我通知你吧，不然你每隔一秒就来问也挺累的。
> resolved: 那成，麻烦你了啊老铁

按照这个思路，修改下代码，如下：

```javascript
let globalResolved = null;
function resolve(param) {
    if (globalResolved) {
        globalResolved(param);
    }
}

class FakePromise {
    constructor(fn) {
        fn(resolve);
    }
    
    then(resolved) {
        // resolved(globalParam);
        // 这里不能立即调用 resolve，必须等 resolve 通知
        // 那就先保存起来
        globalResolved = resolved;
    }
}
```

在`then`方法内，不立即执行传入的`resolved`了，而是保存起来，等待`resolve`执行完成后再调用，就实现了「`resolve`通知`resolved`」。

实际测试发现在打印`'c'`后延迟 1s 后打印了`'b'`。

但是发现使用开始的例子测试又出问题了：

```javascript
new FakePromise((resolve) => {
    console.log('a');
    resolve('b');
    // setTimeout(() => {
    //     resolve('b');
    // }, 1000);
    console.log('c');
})
    .then((res) => {
        console.log(res);
    });

// a
// c
```

只打印了`'a'、'c'`，这是因为先执行`resolve('b')`后才执行`resolved()`。

用上面的例子来说，就是`resolve`想要通知`resolved`时发现`resolved`还没出现。。。


> resolve: 老铁，你要的参数来了，老铁呢？

### 「成功」的实现

如果这样，还要判断`then`是否执行？

捋一捋，首先我们可以认为两个函数是同时执行的（同一个 task）

- constructor
- then

当调用`then`的实参`resolved`时，`constructor`的实参`fn`内可能还有函数`setTimeout`在执行。

所以是全局`resolve`的调用时间不确定，只能在它调用时去通知`resolved`，并且还要判断`resolved`是否存在。

![可能的流程图](http://oyy3cbpm3.bkt.clouddn.com/15182672169883.jpg)

思路没有问题，回顾下之前的实现：

```javascript
// 第一次的尝试，由 then 的参数 resolved 通知
let globalParam = null;
function resolve(param) {
    globalParam = param;
}
// 第二次的尝试，由 resolve 通知
let globalResolved = null;
function resolve(param) {
    if (globalResolved) {
        globalResolved(param);
    }
}
```

如果把这两者结合起来呢？

```javascript
let globalParam = null;
let globalResolved = null;
function resolve(param) {
    // 2
    if (typeof param === 'function') {
        // 3
        if (globalParam) {
            param(globalParam);
            return;
        }
        globalResolved = param;
        return;
    }
    // 4
    if (globalResolved) {
        globalResolved(param);
        return;
    }
    // 1
    globalParam = param;
}
class FakePromise {
    constructor(fn) {
        fn(resolve);
    }
    
    then(resolved) {
        resolve(resolved);
    }
}
```

思路就是分两种情况，

1、假设`constructor`内的`resolve`先执行了，此时还没有`globalResolved`，就保存`param`，即上面代码的「1」。

然后会执行到`then`方法中的`resolve(resolved)`，「2」「3」处的条件为真，成功。


2、假设先执行了`then`的`resolve(resolved)`，能够通过「2」的条件判断，但是无法通过「3」，所以保存了参数到`globalResolved`变量中。然后执行到了`constructor`内的`resolve`，「4」条件为真，也能够成功打印。

总之`resolve`必须要被调用两次。

代码在两种情况下都能够成功打印需要的结果，但仔细思考，如果`constructor`内的`resolve`参数是一个函数呢？

```javascript
new FakePromise((resolve) => {
    console.log('a');
    resolve(function () {
        console.log('i am function');
    });
    // setTimeout(() => {
    //     resolve('b');
    // }, 1000);
    console.log('c');
})
    .then((res) => {
        console.log(res);
    });
```

### 各种错误

脑洞告一段落，虽然「实现」了需要的功能，但实际上并没有按照规范来，比如我们都知道`Promise`在执行过程中是有「状态」的，并且必然是以下其中一种状态

- Pending
- Fulfilled
- Rejected

参考 [Bare bones Promises/A+ implementation](https://github.com/then/promise) 发现核心原理和上面的类似，除了一些潜在的`bug`、没有状态等等，一个最大的问题是代码都是**同步执行**，即在一个 task 中，而规范要求
> 实践中要确保 onFulfilled 和 onRejected 方法异步执行，且应该在 then 方法被调用的那一轮事件循环之后的新执行栈中执行。

举个例子，下面的代码执行结果是什么？

```javascript
console.log('start');
// 上面「错误」的 Promise 实现
new FakePromise((resolve) => {
    console.log('a');
    resolve('b');
    // setTimeout(() => {
    //     resolve('b');
    // }, 1000);
    console.log('c');
})
    .then((res) => {
        console.log(res);
    });
console.log('end');
```

先仔细思考下。

```html


// 想清楚了？







// 确定吗？







// 确定你确定你想清楚了？




// 好吧，来看答案

```

在使用自己实现的`FakePromise`时，输出为：`'start'、'a'、'b'、'c'、'end'`。

将上面代码的`FakePromise`改为原生的`Promise`即可查看到正确的输出为：`'start'、'a'、'c'、'end'、'b'`。

## 总结

其实`Promise`也没有想象的难，花一些时间就能够实现。但相比实现了，更重要的是探索的过程，以及对于「规范」的了解。

下一篇将`FakePromise`改造成「正确」的`Promise`实现，敬请期待~

> 对于上面答案有疑义的可以看 [Tasks, microtasks, queues and schedules(译)](http://ftandy.com/2015/08/23/2015-08-23-tasks-microtasks-queues-and-schedules/) 这篇。

## 参考

- [【翻译】Promises/A+规范](http://malcolmyu.github.io/malnote/2015/06/12/Promises-A-Plus/)
- [Tasks, microtasks, queues and schedules(译)](http://ftandy.com/2015/08/23/2015-08-23-tasks-microtasks-queues-and-schedules/)


