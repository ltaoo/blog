---
title: 数据监听 - 2
categories: Vue
tags:
  - vue
---

之前代码存在很多问题，最大的问题是，实现的并不是一个严格意义上的观察者模式（发布-订阅模式），因为默认订阅了所有数据，且不能取消订阅。
在《JavaScript设计模式与开发实践》中，使用售楼处的例子来类比观察者模式：

售楼处属于发布者，小明（需要购房的人）是订阅者，小明向售楼处订阅了房子开售的信息，售楼处就把小明的信息写入记录表，当房子开售时，售楼处遍历记录表通知到小明（和其他记录表上的人）。

把这个例子和 vue 的实现类比：

创建实例时，发布者（售楼处）会把渲染函数（购房者）添加到发布列表（记录表）中，数据变化（房子开售）时，发布者（售楼处）会调用渲染函数（通知购房者）。
> 而且不同的数据可以类比于不同户型的房子。不同的数据（不同户型的房子）改变（发售）时，可以通知不同的渲染函数（购买者）。

<!--more-->

```javascript
var vm = new Vue({
    data: {
        web: 'my web',
        books: ['first', 'second'],
        person: {
            name: 'ltaoo',
            age: 23
        }
    }
})
```
我们写下这样一段代码后，Vue 做了什么呢？

## 实现 watcher/observer ？
参考文章来对我们之前的代码进行修改，文章中对传进来的数据对象进行修改，而不是之前的复制。我们传入 data，对 data 的属性遍历：
```javascript
// data 是上面的 data，{web: 'my web'....}
Object.keys(data).forEach(function(key) {
    //key = 'web'/'books'/'person'
    Object.defineProperty(data, key, {
        enumerable: true,
        configurable: true,
        get: function () {
            return data[key]
        },
        set: function (newVal) {
            // 如果我们设置的值也是对象，也要对该对象添加get和set，就是调用自身，所以这是一个递归函数
            if(typeof newVal === 'object') {
                // 将传入的值变成新值
                data[key] = newVal
                // 还需要调用自身，call myself
            }
        }
    })
})
```
为了让代码清晰，分为`useForEachAddSetAndGet`函数，用来使用 forEach 循环执行`addSetAndGet`函数，`watch`函数（暴露给外部的函数，可以给对象属性添加 get 和 set 的方法）实例化 Watcher 对象。

```javascript
// ./scripts/watch.js
export default class Watcher {
  constructor (value) {
    // value 即 data
    this.value = value
    this.useForEachAddSetAndGet(value)
  }
  // 遍历对象的属性
  useForEachAddSetAndGet (value) {
    Object.keys(value).forEach(function (key) {
      addSetAndGet(value, key, value[key])
    })
  }
}
// add properties
export function addSetAndGet(obj, key, val) {
  // 这一句就是判断 值 是否也是 对象，如果是就调用自身来添加 get 和 set
  watch(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: ()=>{
      console.log('get value: ', val)
      return val
    },
    set: newVal=>{
      // 如果是赋一样的值，直接退出
      if (newVal === val) {
        return
      }
      // 将新值替代旧值
      val = newVal
      // 这里是对新值做判断，如果新值也是对象，就调用自身来添加 get 和 set
      watch(newVal)
    }
  })
}
// 对传进来的值做判断，如果是对象，就添加 get 和 set
export function watch(value, vm) {
  if(!value || typeof value !== 'object') {
    return
  }
  return new Watcher(value)
}
```
```javascript
// ./srcipts/viewModel.js
init() {
    var watcher = new Watcher(this.data)
}
```

根据实际情况来解读上面的代码。`new Vue`后，将`data`传入 Watcher 来添加 set 和 get：
```javascript
// forEach 读取每个键执行下面的函数。。以person为例
addSetAndGet(data, 'person', {name:'ltaoo'， age: '23'}){
    // 这一次执行是给 person 添加 get 和 set
    // get 简单，返回{name: 'ltaoo', age: 23}即可
    // set，默认接收一个参数，为要赋的新值
    set: function (newVal) {
        val = newVal
    }
    // 好像也OK，但是这样{name: 'ltaoo', age: 23}就没有添加 set 和 get
    // 所以还要对值执行 watch 函数来添加 set 和 get
    watch(val)
    // OK，这样获取值的时候，比如vm.data.person.name 也能触发 console.log 了
    // 并且触发的顺序是，先触发person 上的 get，再触发 name上的get
    //但是如果新值也是一个对象，而我们的 set 并没有对新值做处理，所以还要对新值也执行 watch
    set: function (newVal) {
        val = newVal
        watch(newVal)
    }
    // 这样就OK了！
}
```

## 广播

但是页面上没有数据了，因为我们并没有将 render 函数放到 set 中。如果还是直接放在 set 中，万一用户不想监听数据了，不可能修改源代码，所以我们需要拿出来，通过其他方式来实现相同的效果。

不过即使是用其他方式，也还是需要放一个函数到 set 中，利用这个函数来告诉订阅者（现在还没有）数据改变了。命名为 `notify()` 函数。

回想一下售楼处的例子，当房子开售时，会读取记录表来发送信息。所以在这里，是当数据改变时，`notify()`会读取订阅者数组来执行对应的函数。先大概根据这个逻辑来写这个函数：
```javascript
//
function notify() {
    // 获取订阅者列表
    listeners.forEach(function (fn) {
        fn.call()
    })
}
```

`listener`是一个全局的数组，放着订阅者。问题是怎么将订阅者放到这个数组中？订阅者又是什么？这样吗？

```javascript
addToListeners('web', function () {
    console.log('I watch "web" change')
})
```

可以实现不同属性的变化触发不同的处理函数（不像之前都是触发`render`）。

```javascript
function addToListeners(name, cb) {
    listeners.push(cb)
    // 感觉不对啊，使用数组的话，只能保存一个值，那即使'web'改变，会触发所有的回调函数，所以需要使用对象
    listeners[name] = cb
}

// notify() 也需要改变
function notify(key) {
    // 可以获取到 key 参数，得知这是什么属性改变
    listeners[key].call()
}
```

Dep 就是售楼处（发布对象），因为是售楼处将购房人的信息写入记录表，也是由售楼处来通知购房人
整理一下,
```javascript
// ./scripts/dep.js
export default class Dep {
  constructor () {
    // 这个是售楼处的记录表
    this.listeners = {}
  }
  // 这里就是写入记录表
  addToListeners (name, cb) {
    this.listeners[name] = cb
  }
  // 购房者不想买房了，该方法可以删掉购房者
  removeListener (name) {
    delete this.listeners[name]
  }
  // 该方法就是通知购房者
  notify(name) {
    // console.log(this.listeners)
    Object.keys(this.listeners).forEach((key)=>{
      if(key && key === name) {
        this.listeners[key].call()
      }
    })
  }
}
```

现在问题是，在什么地方使用`Dep`，由于`this.listeners`需要全局唯一，就只能实例化一次`Dep`，而在`watch.js`中，会调用多次`Watcher`，所以就只能在`viewModel.js`中实例化`Dep`了。
```javascript
// ./scripts/watch.js
export default class Watcher {
  constructor (value, dep) {
    // 将传入options 的 data
    this.value = value
    this.dep = dep
    // console.log(this.dep)
    this.useForEachAddSetAndGet(this.value)
    // 查看下什么属性被加入到订阅列表中了
    console.log(this.dep.listeners)
  }
  // 遍历对象的属性
  useForEachAddSetAndGet (value) {
    var watcher = this
    //console.log(this.dep)
    Object.keys(value).forEach(function (key) {
      // 添加 set 和 get
      watcher.addSetAndGet(value, key, value[key])
      // console.log('add ', key)
      // 添加了 get 和 set ，再把这个属性写到 listeners 中，知道是什么属性被订阅了
      watcher.dep.addToListeners(key, function () {
        // 这里是回调函数，即 key 改变的时候会执行这里
        console.log('set ', key)
      })
    })
  }
  // 给 key 加 set 和 get
  addSetAndGet(obj, key, val) {
    if(val.constructor == Object) {
      // 如果 val 也是对象，就也要调用一次 useForEachAddSetAndGet()
      this.useForEachAddSetAndGet(val)
    }
    Object.defineProperty(obj, key, {
      enumerable: true,
      configurable: true,
      get: ()=>{
        return val
      },
      set: newVal=>{
        // 如果是赋一样的值，直接退出
        if (newVal === val) {
          return
        }
        // 将新值替代旧值
        val = newVal
        if(val.constructor == Object) {
          // 如果 newVal 也是对象，就也要调用一次 useForEachAddSetAndGet()
          this.useForEachAddSetAndGet(newVal)
        }
        // 广播
        this.dep.notify(key)
      }
    })
  }
}
// ./scripts/viewModel.js 只列出修改的部分
import Dep from './dep.js'
constructor (options) {
    this.data = options.data;
    this.$data = {};
    this.dep = new Dep()
    this.watcher = new Watcher(this.data, this.dep)
    // this.render();
}
```
### 测试

可以在`index.js`中测试是否成功：
```javascript
var vm = new Vue({
  data: {
    web: 'my web',
    books: ['first', 'second'],
    person: {
      name: 'ltaoo',
      age: 23
    }
  }
})
var person = vm.data.person
person.name = 'ltooo'
// 移除对 name 的订阅
vm.watcher.dep.removeListener('name')
person.name = 'loooo' // 这里不会触发console.log
vm.watcher.dep.addToListeners('name', function () {
  console.log('这是很特殊的处理')
})

person.name = 'ooooo' // 这里触发console.log('这是很特殊的处理')
```
浏览器控制台只打印一次 `set name`，打印一次`这是很特殊的处理`，表示成功！

## render 模块
为了能让`render()`函数可以被其他模块调用，将 render()写成一个单独的模块，所以代码是这样的：
```javascript
// ./script/viewModel.js
import { render } from './render.js'
export default class Vue {
  constructor (options) {
    this.data = options.data;
    this.$data = {};
    this.dep = new Dep()
    this.watcher = new Watcher(this.data, this.dep)
    render(this.data);
  }
}
// ./scripts/render.js
export function render(data) {
  var app = document.getElementById('app');
  if(app.hasChildNodes()) {
    // 如果 app 有子元素
    app.childNodes.forEach(function (node) {
      // console.log(node);
      var key = node.getAttribute('v-bind');
      if(key && typeof data[key] === 'object') {
        // 这里只考虑数组和字符串两种，如果是数组
        var content = '';
        data[key].forEach(function (value) {
          content += '<li>' + value + '</li>';
        })
        node.innerHTML = content;
      }else if(key && typeof data[key] === 'string') {
        // 如果是字符串
        // console.log(vue.$data[key]);
        node.innerHTML = data[key];
      }
    })
  }
}
```

查看一下页面，OK，能够显示数据。

## 不同的处理函数

OK，现在再把`render`加入到订阅列表中：
```javascript
watcher.dep.addToListeners(key, function () {
    // 这里是回调函数，即 key 改变的时候会执行这里
    console.log('set ', key)
    render()
})
```
`render()`是要接收一个参数，函数根据这个参数才能够渲染出页面，不过我们其实也不想任何数据改变都重新渲染整个页面，而是什么属性改变就渲染对应的地方，比如：
```html
<h2 v-bind="web"></h2>
```
当`web`字段改变时，就重新渲染这一部分，其他的不会改变。所以我们需要一个新的函数，暂时命名为`renderSingle()`：
```javascript
// ./scripts/render.js
export function renderSingle(key, value) {
  var app = document.getElementById('app');
  if(app.hasChildNodes()) {
    // 如果 app 有子元素
    app.childNodes.forEach(function (node) {
      // console.log(node);
      var name = node.getAttribute('v-bind');
      // 这里的 value 是我们的属性名比如 web、books 这种
      // console.log(value, key)
      if(name === key) {
        // 如果是，OK，节点找到，然后更新值
        node.innerHTML = value[key];
      }
    })
  }
}
```
这个函数放在了数据变化的回调函数中。
```javascript
watcher.dep.addToListeners(key, function () {
    // 这里是回调函数，即 key 改变的时候会执行这里
    console.log('set ', key)
    renderSingle(key, value)
})
```

OK，终于实现了之前就实现的效果，不过增加了很多东西，方便之后的拓展。

## render.js 的拓展

这里就要提到“指令”了，我们只能处理有指令的节点。现在只有`v-bind`，表示会往这个节点里面添加数据。我们还需要`v-model`、`v-for`等，实现方式是，获取节点，读取节点上的属性，如果有`v`，就是我们的指令了，就执行这个指令对应的函数。

```javascript
// 我们有指令对应的函数
operation = {
  'v-bind': function (node, data, value) {
    console.log(node)
    node.innerHTML = data[value]
  },
  'v-model': function (node, data, val) {
    node.value = data[val]
    node.oninput = function () {
      // console.log('input')
      data[val] = node.value
    }
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

render(data) {
    var app = document.getElementById('app');
    if(app.hasChildNodes()) {
      var render = this
      // 如果 app 有子元素
      app.childNodes.forEach(function (node) {
        if(node.nodeType === 1 && node.hasAttributes()) {
          // 获取属性
          var atr = node.attributes
          Object.keys(atr).forEach(function (name) {
            var key = atr[name].name
            var value = atr[name].value

            if(key.indexOf('v') > -1) {
              //根据指令来执行不同的代码
              // console.log(key, value)
              render.operation[key].call(null, node, render.vm.data, value)
            }
          })
        }
      })
    }
  }
  
```

这样就能够实现页面初始化了。我们将代码进行整理：

```javascript
// ./scripts/render.js
export default class Render {
  constructor(vm) {
    this.vm = vm
    this.operation = {
      'v-bind': function (node, data, value) {
        console.log(node)
        node.innerHTML = data[value]
      },
      'v-model': function (node, data, val) {
        node.value = data[val]
        // 监听 oninput 事件
        node.oninput = function () {
          // console.log('input')
          data[val] = node.value
        }
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
  }
  
  render(data) {
    var app = document.getElementById('app');
    if(app.hasChildNodes()) {
      var render = this
      // 如果 app 有子元素
      app.childNodes.forEach(function (node) {
        if(node.nodeType === 1 && node.hasAttributes()) {
          // 获取属性
          var atr = node.attributes

          Object.keys(atr).forEach(function (name) {
            var key = atr[name].name
            var value = atr[name].value

            if(key.indexOf('v') > -1) {
              //根据指令来执行不同的代码
              // console.log(key, value)
              render.operation[key].call(null, node, render.vm.data, value)
            }
          })
        }
      })
    }
  }

  renderSingle(key, value) {
    var app = document.getElementById('app');
    var render = this
    if(app.hasChildNodes()) {
      // 如果 app 有子元素
      app.childNodes.forEach((node)=>{
        if(node.nodeType === 1 && node.hasAttributes()) {
          // 获取属性
          var atr = node.attributes

          Object.keys(atr).forEach((name)=>{
            //
            var attrName = atr[name].name
            var value = atr[name].value

            if(key === value) {
              render.operation[attrName].call(null, node, render.vm.data, value)
            }
          })
        }
      })
    }
  }
}
```
重点是在于`v-model`的处理函数，可以看到这里对节点的`oninput`进行监听，将每一次的输入都赋值给 Vue 的 data，这样就实现了双向绑定，即使用`v-model`指令替代了我们之前一直使用的在`index.js`中手动监听。

这里涉及到一个 “值的传递”，在 render.js 这个文件中我们要获取到 `data`，而`data`又是挂载在`Vue`这个类上面，所以我们将这个类传递给`Render()`。
```javascript
// ./scripts/viewModel.js
import Watcher from './watch.js'
import Dep from './dep.js'
import Render from './render.js'

export default class Vue {
  constructor (options) {
    this.data = options.data;
    // 订阅列表
    this.dep = new Dep()
    // 渲染页面，将vm实例传进去
    this.render = new Render(this)
    //
    this.watcher = new Watcher(this.data, this.dep, this.render)
    this.render.render(this.data, this);
  }
}
```

## 总结

现在我们有了`viewModel.js`、`render.js`、`watch.js`、`dep.js`，四个类，实现了我们的简单的 Vue 实现。

- dep 是售楼处
- watch 是什么？在 watch 中数据设置了 set 就表示有购房意愿，dep 才可以把数据加入到记录表
- render 是购房者

不知道这种比喻是否恰当，等对 Vue 的源码进行阅读后应该会有更深的认识。

### 对象的传递

由于每个类只能实例化一次，所以在模块间“通信”是使用了将类作为参数传递的方式。尝试画流程图来将整个流程理清楚，数据是从哪到哪。


![](./2272c6cd-d709-4b89-8d02-6e1ad5fbb377.png)

在画图的过程中，意识到如果按照顺序来实例化，将实例化后的对象挂载在起点（viewModel）上，只需要传递 viewModel 一个参数即可。

```javascript
this.data = options.data
this.render = new Render(this)// this 上就有了 render
this.dep = new Dep() // this 上就有了 dep
this.watcher = new Watcher(this) // this 上有 render 和 dep
```
![](./8f552cd7-e849-4142-9ce8-4a90a48c7f2f.png)


这一次学习中，对“观察者模式”有了更深的理解，也对 Node 的相关 API 进行了更多的了解，很多之前没有用过的方法、属性都在这次学习中出现。

一个前端框架，不仅仅要熟悉 JavaScript 语言，还要对浏览器环境下的 JavaScript 有很深的了解。最重要的还是代码的组织，即设计模式（架构？）非常重要。

## 参考

- [vue 源码分析之如何实现 observer 和 watcher](https://segmentfault.com/a/1190000004384515)
