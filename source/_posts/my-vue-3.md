---
title: 数据监听 -3
categories: Vue
tags:
  - vue
---

数据绑定的功能基本实现了，但也很明显存在很多问题，

首先，第一个问题，现在是将属性名作为了事件名来实现数据的监听，假设存在`name`和`person.name`，我们是将`name`传入渲染函数，对节点遍历查找“指令的值”，将其与`name`进行对比，符合就是找到了，但是很明显`'person.name' !== 'name'`，所以无法实现双向绑定；第二个问题当然是数组类型的处理；
参考 Vue 源码（v0.10）来解决这些问题。

<!--more-->

## emitter.js
首先查看 Vue 使用的注册事件及响应事件的类。下面是与我们之前的`dep.js`的 API 进行对比：

- addListeners => on
- notify => emit
- removeListener => off
- listeners => _cbs

多出一个`this._ctx`，执行上下文，会在执行函数时作为第一个参数传入。
> Vue 中的该文件，和通用的自定义事件类库很相似，可能接口名会不同。所以一次学习，终身受益~


使用该文件来替代之前的`dep.js`文件，熟悉用法。
```javascript
// ./scripts/emitter.js
var slice = [].slice

export default class Emitter{
    constructor(ctx) {
        this._ctx = ctx || this
    }
    on(event, fn) {
        this._cbs = this._cbs || {}
        ;(this._cbs[event] = this._cbs[event] || []).push(fn)
        return this
    }
    once(event, fn) {
        this._cbs = this.cbs || {}
        var self = this
        function on() {
            self.off(event, on)
            fn.apply(this. arguments)
        }
        on.fn = fn
        this.on(event, on)
        return this
    }
    off(event, fn) {
        this._cbs = this._cbs || {}
        // all
        if (!arguments.length) {
            this._cbs = {}
            return this
        }
        // specific event
        var callbacks = this._cbs[event]
        if (!callbacks) return this
        // remove all handlers
        if (arguments.length === 1) {
            delete this._cbs[event]
            return this
        }
        // remove specific handler
        var cb
        for (var i = 0; i < callbacks.length; i++) {
            cb = callbacks[i]
            if (cb === fn || cb.fn === fn) {
                callbacks.splice(i, 1)
                break
            }
        }
        return this
    }
    emit(event, a, b, c) {
        this._cbs = this._cbs || {}
        var callbacks = this._cbs[event]
        if (callbacks) {
            callbacks = callbacks.slice(0)
            for (var i = 0, len = callbacks.length; i < len; i++) {
                callbacks[i].call(this._ctx, a, b, c)
            }
        }
        return this
    }
    applyEmit(event, a, b, c) {
        this._cbs = this._cbs || {}
        var callbacks = this._cbs[event], args
        if (callbacks) {
            callbacks = callbacks.slice(0)
            args = slice.call(arguments, 1)
            for (var i = 0, len = callbacks.length; i < len; i++) {
                callbacks[i].apply(this._ctx, args)
            }
        }
        return this
    }
}
```
大概描述一下，`on`传入事件名与处理函数，注册事件；`emit`传入事件名，触发事件（执行处理函数）；`off`参数为空时清空所有事件，传入事件名则只取消该事件；`once`和`on`用法相同，区别在于`once`只响应一次，不同于`on`可以响应多次；

由于接口和我们自己的`dep.js`类似，只需要修改 watch.js 中相应的代码，逻辑完全可以不变。


## observer.js 
作用与之前的 `watch.js`相同，都是对传进来的数据进行处理，添加 get 和 set。但 Vue 在该文件内有对数组类型的处理。大概逻辑如下

![](28179d7a-cf87-48fa-9dfe-a6aa899099e2.png)

`observe`是暴露的接口，可以使用该函数对对象类型（虽然数组也是对象，但这里指狭义的对象）进行处理，添加 set 和 get，注册事件与响应事件。

先暂时忽略`convert`函数。`watch`函数是对数据类型做判断，并且调用不同的添加 get 和 set 的函数。和我们之前的做对比，由于我们并没有对数组类型做处理，所以是这样的：

- observe => useForEachAddGetAndSet
- convertKey => addGetAndSet


### 重写我们的`watch.js`
```javascript
// ./scripts/watch.js
import Emitter from './emitter'
export function observe(obj) {
  if(isWatchable(obj)) {
    watch(obj)
  }
}
// 判断是否是对象或者数组
export function isWatchable(obj) {
  if(typeof obj === 'object') {
    return true
  }
  return false
}

export function watch(obj) {
  // 判断 obj 类型
  if(Array.isArray(obj)) {
    // 如果是数组
    console.log('数组暂时不处理')
  }else {
    // 对对象遍历，每一个键（属性）添加 set 和 get
    for(var key in obj) {
      convertKey(obj, key)
    }
  }
}
// 该函数是实际添加 get 和 set 的函数
export function convertKey(obj, key) {
  //console.log(key)
  var value = obj[key] // 
  //console.log(obj[key])
  //给 key 加 set 和 get
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function () {
      console.log('get value is', value)
      return value
    },
    set: function (newVal) {
      console.log('set newVal is', newVal)
      value = newVal
      // 如果新值也是对象，也要对新值调用 observe
      init(obj[key])
    } 
  })
  // ok，处理完了键，如果值也是对象，就要对值调用observe
  init(obj[key])
  // 写成函数不用重复写两次
  function init(obj) {
    if(isWatchable(obj)) {
      observe(obj)
    }
  }
}
```

逻辑是这样，如果传进来的 data 的值都是对象，则每个键都会添加上get 和 set。而且还需要加上触发事件，就是我们之前的`notify(key)`

而 Vue 的做法是，数据的改变，将会以类似冒泡的形式将“数据改变”层层向上传递。

假设有一个 data：
```javascript
data: {
    web: {
        title: 'my web',
        categories: {
            books: {
                name: 'nodejs'
            }
        }
    },
    url: 'ltaoo.com'
}
```
Vue 给每个对象添加了属性`__emitter__`，该属性的值是 emitter 对象。所以上面的 data 最后会变成这样：
```javascript
data: {
    web: {
        title: 'my web',
        categories: {
            books: {
                name: 'nodejs',
                __emitter: emitter（booksObserver）// 括号实际并不存在，只是便于说明才加上的
            },
            __emitter__: emitter（categoriesObserver）
        },
        __emitter__: emitter（webObserver）
    },
    url: 'ltaoo.com',
    __emitter__: emitter（dataObserver）
}
```
这些 emitter 会观察数据变化，观察“同级”的属性。

data 的 dataObserver 观察 web、url 变化
web 的 webObserver 观察 title、categories 变化
categories 的 categoriesObserver 观察 books 变化
books 的 booksObserver 观察 name 变化

反过来说，name 变化，会通知 booksObserver ，booksObserver 通知 categoriesObserver，categoriesObserver 通知webObserver，webObserver 通知 dataObserver，dataObserver 通知 DOM 节点数据改变了。

我们给我们的代码加上这一功能，就是`convert`函数：
```javascript
// ./scripts/watch.js
export function convert(obj) {
  if(!obj.__emitter__) {
    var emitter = new Emitter()
    Object.defineProperty(obj, '__emitter__', {
      value: emitter,
      enumerable: true,
      configurable: true
    })
  }
}
```
> 加上这里后，要在 convertKey 中对 key 做判断，如果是 `__emitter__`，则直接跳过，不然会给该值也加上 get 和 set

然后在 get 和 set 中触发
```javascript
// ./scripts/watch.js convertKey()函数
// 这里的 emitter 是与之同级的，比如，如果 key 是 name，则 emitter 就是 booksObserver
var emitter = obj.__emitter__
Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function () {
      //console.log('get value is', value)
      emitter.emit('get', key)
      return value
    },
    set: function (newVal) {
      //console.log('set newVal is', newVal)
      value = newVal
      // name 的变化要通知 bookObserver
      emitter.emit('set', key)
      // 如果新值也是对象，也要对新值调用 observe
      init(obj[key])
    } 
  })
```

肯定要先有`emit.on()`注册事件，才能够触发事件。有两个地方可以添加，一是 convert 函数，二是 observe 函数。

按照上面说的，要层层向上触发事件，那就要求`emit.on()`可以获取到父级的 emitter （来触发该对象中的事件）。在 convert 函数中，只传入了 obj，并不能实现我们需要的效果。所以在 observe 函数内添加：
```javascript
// ./scripts/watch.js
export function observe(obj, observer) {
  if(isWatchable(obj)) {
    convert(obj)
    watch(obj)
    var emitter = obj.__emitter__
    emitter.on('set', function() {
      observer.emit('set')
    })
  }
}
```

由于 observe 是递归函数，每一个属性都会添加`__emitter__`并注册`set`事件，而该事件的处理函数是触发父级的`__emitter__`，所以会层层向上触发 set 事件。将代码完善：
```javascript
// ./scripts/viewModel.js
this.emitter
  .on('set', function(path) {
    console.log(path + ' is setting')
  })
observe(this.data, this.emitter)
```
```javascript
// ./scripts/watch.js
import Emitter from './emitter'
export function observe(obj, observer) {
  if(isWatchable(obj)) {
    convert(obj)
    watch(obj)
    var emitter = obj.__emitter__
    emitter.on('set', function(path) {
      observer.emit('set', path)
    })
  }
}

export function isWatchable(obj) {
  if(typeof obj === 'object') {
    return true
  }
  return false
}
export function convert(obj) {
  if(!obj.__emitter__) {
    var emitter = new Emitter()
    Object.defineProperty(obj, '__emitter__', {
      value: emitter,
      enumerable: true,
      configurable: true
    })
    emitter.on('set', function (key, val) {
      // console.log('set')
    })
    emitter.on('get', function(key) {
      //console.log('get', key)
    })
  }
}
export function watch(obj) {
  // 判断 obj 类型
  if(Array.isArray(obj)) {
    // 如果是数组
    console.log('数组暂时不处理')
  }else {
    // 对对象遍历，每一个键（属性）添加 set 和 get
    for(var key in obj) {
      convertKey(obj, key)
    }
  }
}
// 该函数是实际添加 get 和 set 的函数
export function convertKey(obj, key) {
  if(key.charAt() === '_') {
    return
  }
  var value = obj[key] // 
  var emitter = obj.__emitter__
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function () {
      //console.log('get value is', value)
      emitter.emit('get', key)
      return value
    },
    set: function (newVal) {
      //console.log('set newVal is', newVal)
      value = newVal
      emitter.emit('set', key)
      // 如果新值也是对象，也要对新值调用 observe
      init(obj[key])
    } 
  })
  // ok，处理完了键，如果值也是对象，就要对值调用observe
  init(obj[key])
  function init(obj) {
    if(isWatchable(obj)) {
      observe(obj, emitter)
    }
  }
}
```
```javascript
// index.js
import Vue from './scripts/viewModel';

var vm = new Vue({
  data: {
    web: 'my web',
    books: ['first', 'second'],
    person: {
      name: 'ltaoo',
      age: 23,
      web: 'person web',
      obj: {
        name: 'laooo'
      }
    }
  }
})

vm.data.person.obj.name = 'loooo'
```

可以在浏览器中看到打印出 `name is setting`，就表示成功。

事件是向上传递了，而且也可以知道是什么属性发生改变，渲染函数根据传过来的属性值了解到是什么属性发生了改变，但是并没有解决我们一开始提出的问题？
```html
<div id="app">
    <h2 v-bind="web"></h2>
    <input type="text" v-model="web" placeholder="input something">
    <input type="text" v-model="person.web"><a href="" v-bind="person.web"></a>
</div>
```
可以看到同时用到了`web`和`person.web`，如果只知道是 web 发生了变化，难道两个都要重新渲染吗？而且还要对所有的指令的值做分析，先判断是否有`.`，再分割成数组后判断数组中是否有 web，毫无疑问这样做是很有问题的。所以 Vue 使用了“路径”来确定一个属性的位置。

在`emit('set')`时，不仅传递 key，还传递 path，而 path 是由发生变化的属性与包含这一属性的属性名构成。所以我们可以知道是`person.obj.name`发生变化。

在哪里添加 path 呢？联想到在调用 observe 时，我们将 emitter 作为参数传入，那同样可以将 key 也传入（`__emitter__`和 key 是同级的），同样在`observer.emit('set')`是调用到了上一级的 emitter，那也在这里把 path 传过去。所以最后代码是这样的：
```javascript
export function observe(obj, observer, path) {
  // 
  var rawPath = path === '' ? '' : path + '.'
  if(isWatchable(obj)) {
    convert(obj)
    watch(obj)
    var emitter = obj.__emitter__
    emitter.on('set', function(path) {
      console.log(path)
      observer.emit('set', rawPath+path)
    })
  }
}
```
然后需要将所有调用`observe`函数增加 path 参数。然后同样的数据，浏览器打印出：
```javascript
person.obj.name is setting
```
### render()
事件可以传播了，最后会传播到我们在 `viewModel.js`中实例化的 emitter 监听的 set 事件，所以我们需要在这里调用 render 函数并传入值，告诉 render 函数是什么属性发生了变化。
```javascript
// ./scripts/viewModel.js
export default class Vue {
  constructor (options) {
    var vm = this
    var data = this.data = options.data;
    //console.log(data)
    var render = new Render(vm)
    // 订阅列表
    var emitter = this.emitter = new Emitter()
    //
    //this.watcher = new Watcher(this.data, this.emitter, this.render)
    this.emitter
      .on('set', function(path) {
        render.renderSingle(path, vm)
      })
    observe(this.data, this.emitter, '')
  }
}
```

我们在`set`事件的处理函数中调用`render.renderSingle()`函数并传入`path`和`vm`，`vm`是为了获取到 `data`，所以传`vm.data`也可。

其他都相同，需要将`operation`内指令对应的函数进行修改，增加如果指令对应的值有`.`的情况（person.web）。逻辑也简单，将值进行分割，使用循环来获取到值，并赋值给 DOM 节点。

`v-model`和`v-bind`的逻辑是相同的，只是一个用`innerHTML`赋值，一个用`value`，所以将其写成一个函数，减少代码的重复：
```javascript
export function get(obj, key) {
  if(key.indexOf('.') > -1) {
    // 
    var ary = key.split('.')
    var temp = obj
    for(var i = 0; i < ary.length; i ++) {
      temp = temp[ary[i]]
    }
    return temp
  }else {
    return obj[key]
  }
}
```
所以`operation`的代码是这样的：
```javascript
// ./scripts/render.js
this.operation = {
  'v-bind': function (node, data, key) {
    node.innerHTML = get(data, key)
  },
  'v-model': function (node, data, key) {
    node.value = get(data, key)
  },
  'v-for': function (node, data, value) {
    // console.log('v-for')
    var content = '';
    data[value].forEach(function (value) {
      content += '<li>' + value + '</li>';
    })
    node.innerHTML = content
  }
}
```

OK，可以正常获取值了，接下来处理赋值，即`oninput`事件，将输入框的值赋给 data：
```javascript
'v-model': function (node, data, key) {
    node.value = get(data, key)
    // input event
    node.oninput = function () {
      if(key.indexOf('.') < 0) {
        data[key] = node.value
        return
      }else {
        // 
        var pathAry = key.split('.')
        var temp = data
        for(var i = 0; i < pathAry.length -1; i ++) {
          temp = temp[pathAry[i]]
        }
        // console.log(v)
        temp[pathAry[i]] = node.value
      }
    }
},
```

然后就可以正确实现双向绑定了。至此我们解决了第一个问题。