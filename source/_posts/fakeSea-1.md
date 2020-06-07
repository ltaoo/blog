---
title: 模仿 sea.js 实现 CMD 模块加载器（一）
categories: JavaScript
tags:
- 模块加载器
- 轮子
date: 2017/04/16
---

模块化是现在编写 JavaScript 的必然选择，而模块规范和我们如何写模块化的代码有很大关系，比如`AMD`规范与`CMD`规范，而这些规范具体是指什么呢，下面以仿照`sea.js`的源码自己实现一个简单的模块加载器来具体了解`CMD`规范。

<!--more-->

还是先来实现一个最简单的模块加载器。

## 现代模块机制

下面以一个简单的例子来介绍模块机制。
`html`文件内加载了三个`js`文件，`fakeSea`为核心库，`say.js`声明了一个方法`say()`可以打印`hello`，`main.js`引入`say.js`并使用该方法。

所以下面的代码可以在浏览器中打印出`hello`。

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>实现简单的 CMD 模块加载器</title>
</head>
<body>
    <script src="./fakeSea.js"></script>
    <script src="./say.js"></script>
    <script src="./main.js"></script>
</body>
</html>
```
```javascript
// fakeSea.js
var module = {}
;(function (global) {
    // 内部保存模块的对象
    var providedMods = {}
    // 使用模块
    function load (ids, callback) {
        var deps = []
        for(var i = 0, len = ids.length; i < len; i++) {
            // 获取保存在 module 内的模块，放入 deps 数组中
            deps[i] = providedMods[ids[i]]
        }

        callback && callback.apply(null, deps)
    }
    // 声明模块
    function declare (name, mod) {
        providedMods[name] = mod
    }

    module.load = load
    module.declare = declare
})(this)
```
```javascript
// say.js
module.declare('say', function () {
    alert('hello')
})
```
```javascript
// main.js
module.load(['say'], function (say) {
    say()
})
```

### 为什么使用模块化

当然在一个`js`文件中声明方法，另一个文件中使用，不用这种所谓的**模块化**也可以实现，但是存在一个问题就是存在太多全局变量。而使用模块化，全局只存在`module`一个变量。

而且，现在虽然需要手动在`html`内引入`say.js`和`main.js`，但完善后的`fakeSea.js`，仅仅只需要引入`fakeSea.js`一个文件。

同时，需要注意到先引入了`say.js`再引入`main.js`，因为`main.js`需要用到`say.js`内的函数，即`main.js`依赖`say.js`，使用模块化后就不需要担心先后顺序，都在内部进行了处理。

### 原理

很明显能够看出这是利用了“闭包”，`providedMods`变量存在于自执行函数的作用域内，按理说外部无法访问到，但是同时存在于该函数作用域内还有`load`以及`declare`函数，这两个函数能够访问到，所以`declare`用来修改`providedMods`变量，`load`用来获取`providedMods`变量。

最后将这两个函数暴露至全局，外部就能够借助着两个函数访问作用域内的变量了。

## 开始

`sea.js`最初版本代码量虽然不多只有七百余行，但为了降低难度还是从”简陋版“过渡到”完善版“，一个一个功能点进行实现。

### 自动处理依赖

只需要指定入口文件，不需要关心先引入哪个模块，我们现在来实现该功能点。

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>实现简单的 CMD 模块加载器</title>
</head>
<body>
    <script src="./fakeSea.js"></script>
    <script src="./main.js"></script>
</body>
</html>
```

```javascript
// say.js
module.declare(function (exports) {
    exports.say = function () {
    	alert('hello')
    }
})
```

```javascript
// main.js
module.load(['./say.js'], function (module) {
    module.say()
})
```

可以看到`main.js`中指定了`say`模块的地址为`./say.js`，声明模块的方式也改变为`declare`方法接收函数作为参数，在函数内使用`exports`导出模块。

#### 生成 script 标签

当`load`模块时，如果模块存在，就可以像上面一样直接使用，不存在则需要借助`script`标签加载该文件。

```javascript
function load (ids, callback) {
    // 筛选出未加载的模块
    var urls = getUnMemoized(ids)
    // 如果未加载的模块数量为 0 ，表示都是已经加载好的模块，就可以直接调用回调函数了
    if (urls.length === 0) {
        var deps = []
        for(var i = 0, len = ids.length; i < len; i++) {
            // 获取保存在 module 内的模块，放入 deps 数组中
            deps[i] = providedMods[ids[i]]
        }

        callback && callback.apply(null, deps)
    } else {
        // 不然的话就要加载该模块
        for(var i = 0, len = urls.length; i < len; i++) {
            // 使用 provide 函数来加载新模块
            provide(urls[i])
        }
    }
}

// 筛选未加载的模块
function getUnMemoized (ids) {
    var unLoadMods = []
    for(var i = 0, len = ids.length; i < len; i++) {
        if (!providedMods[ids[i]]) unLoadMods.push(ids[i])
    }

    return unLoadMods
}
```

重点在于`provide`函数，

```javascript
// 加载新模块
function provide (url) {
    var script = document.createElement('script')
    script.src = url
    script.async = true
    var head = document.getElementsByTagName('head')[0]
    head.appendChild(script)
}
```

可想而知，当`provide('./say.js')`的时候，会生成一个`script`标签插入`head`内，当文件下载成功，就会立即执行其中的代码：

```javascript
module.declare(function (exports) {
    exports = function () {
        alert('hello')
    }
})
```

我们”假设“该文件加载成功后，模块就被定义好了可以使用了，那就应该在文件加载成功的时候，通知`load`函数可以执行回调函数了是吧，所以需要用到多个回调函数：

![回调流程](./46971596-810F-4DC9-9A99-5B7766D40289.png)

增加好回调函数后，刷新浏览器如果报错`TypeError: say in not a function`表示成功，能够执行我们想要执行的函数。

### 保存模块

由于`load`时会从`providedMods`对象中获取需要的模块，所以肯定要在`script`标签下载文件成功后，将该模块加入到该全局对象中。

为什么不在`declare`函数内完成该功能？因为`declare`只接收一个函数，没有`name`字段，不知道该怎么保存。

`sea.js`的实现是，在`declare`内将函数加入到一个数组中，再在能够拿到`name`的地方取出来。

比如`provide`函数内，因为此时有`url`。

```javascript
// 加载新模块
function provide (url, callback) {
    var script = document.createElement('script')
    script.addEventListener('load', function () {
        for(var i = 0, len = pendingMods.length; i < len; i++) {
            var mod = pendingMods[i]
            // 加入全局对象
            mod && memoize(url, mod)
        }
        callback()
    })
    script.src = url
    script.async = true
    var head = document.getElementsByTagName('head')[0]
    head.appendChild(script)
}
```

### 获取模块

OK，我们现在假定已经完成了模块加载，现在是要调用`load`的回调函数了，我们需要将模块作为参数传入。

```javascript
// 使用模块
function load (ids, callback) {
    // 筛选出未加载的模块
    var urls = getUnMemoized(ids)
    // 如果未加载的模块数量为 0 ，表示都是已经加载好的模块，就可以直接调用回调函数了
    if (urls.length === 0) {
        var deps = []
        for(var i = 0, len = ids.length; i < len; i++) {
            // 获取保存在 module 内的模块，放入 deps 数组中
            deps[i] = providedMods[ids[i]]
        }

        callback && callback.apply(null, deps)
    } else {
        // 不然的话就要加载该模块
        for(var i = 0, len = urls.length; i < len; i++) {
            // 使用 provide 函数来加载新模块
            provide(urls[i], function () {
                callback()
            })
        }
    }
}
```

这部分代码肯定是没有用了，因为`providedMods['./say.js']
`对应的值是：

```javascript
function (exports) {
    exports.say = function () {
        alert('hello')
    }
}
```

我们需要的是`exports`这个变量，而不是整个函数。我们声明一个`require`函数用来获取依赖。

```javascript
// 获取依赖
function require(id) {
    var factory = providedMods[id]
    var exports = {}
    if (typeof factory === 'function') {
        factory(exports)
    }

    return exports
}
```
通过声明一个空对象，作为参数传入后，经过`factory`的”加工“后，就变成了`say`函数。

不过发现之前`load`代码存在很大的问题，如果

```javascript
load(['a.js', 'b.js'], function (a, b){
	// ...
})
```
`a.js`和`b.js`都不存在需要使用`script`加载，而`provide`函数在`for`循环内，每加载成功一个`js`文件就要调用一次`callback`很明显不对。

```javascript
for(var i = 0, len = urls.length; i < len; i++) {
    // 使用 provide 函数来加载新模块
    provide(urls[i], function () {
    	// 会调用 urls.length 次数 
        callback()
    })
}
```

所以要重写。。。。怎么写呢，将`load`函数名改为`provide`，`provide`改为`fetch`，再新增`load`函数：

```javascript
// 使用模块
function load (ids, callback) {
    provide.call(null, ids, function () {
        var args = []
        for(var i = 0, len = ids.length; i < len; i++) {
            var mod = require(ids[i])
            if (mod) {
                args.push(mod)
            }
        }
        callback && callback.apply(null, args)
    })
}
```

### 完整代码

```javascript
// fakeSea.js
var module = {}
;(function (global) {
    // 内部保存模块的对象
    var providedMods = {}
    // 临时保存声明的模块
    var pendingMods = []
    // 使用模块
    function load (ids, callback) {
        provide.call(null, ids, function () {
            var args = []
            for(var i = 0, len = ids.length; i < len; i++) {
                var mod = require(ids[i])
                if (mod) {
                    args.push(mod)
                }
            }
            callback && callback.apply(null, args)
        })
    }
    // 
    function provide (ids, callback) {
        // 筛选出未加载的模块
        var urls = getUnMemoized(ids)
        // 如果未加载的模块数量为 0 ，表示都是已经加载好的模块，就可以直接调用回调函数了
        if (urls.length === 0) {
            return callback && callback.apply(null, deps)
        } else {
            // 不然的话就要加载该模块
            for(var i = 0, len = urls.length, count = len; i < len; i++) {
                // 使用 fetch 函数来加载新模块
                fetch(urls[i], function () {
                    // 当模块都加载成功了，才调用回调函数
                    --count === 0 && callback()
                })
            }
        }
    }
    // 声明模块
    function declare (factory) {
        pendingMods.push(factory)
    }
    // 将模块加入到全局对象中
    function memoize (url, mod) {
        providedMods[url] = mod
    }
    // 筛选未加载的模块
    function getUnMemoized (ids) {
        var unLoadMods = []
        for(var i = 0, len = ids.length; i < len; i++) {
            if (!providedMods[ids[i]]) unLoadMods.push(ids[i])
        }

        return unLoadMods
    }
    // 加载新模块
    function fetch (url, callback) {
        var script = document.createElement('script')
        script.addEventListener('load', function () {
            for(var i = 0, len = pendingMods.length; i < len; i++) {
                var mod = pendingMods[i]
                // 加入全局对象
                mod && memoize(url, mod)
            }
            callback()
        })
        script.src = url
        script.async = true
        var head = document.getElementsByTagName('head')[0]
        head.appendChild(script)
    }
    // 获取依赖
    function require(id) {
        var factory = providedMods[id]
        var exports = {}
        if (typeof factory === 'function') {
            factory(exports)
        }
        return exports
    }

    module.load = load
    module.declare = declare
})(this)
```

```javascript
// main.js
module.load(['./say.js'], function (module) {
    module.say()
})
```

```javascript
// say.js
module.declare(function (exports) {
    exports.say = function () {
        alert('hello')
    }
})
```

## 总结

虽然仅仅是一个简单的模块加载器，但是也能够大概了解如何获取模块、如何保存模块。


