---
title: 数据监听 - 1
categories: Vue
tags:
  - vue
---

在Vue.js中，数据的变化会引起 DOM 的改变，是否能理解为 DOM 订阅了数据的改变事件，每当数据改变时，就发布广播，DOM 得以知道数据改变？

具体如何实现呢？尝试以自己对数据绑定的理解，先实现一个“看起来”能够监听数据的实例，而不是一开始就阅读 Vue 的源码（不过如果没先阅读过，也不知道自己看不懂...），再将该实例进行优化。

虽然不直接阅读源码，但首先需要对 Vue 的使用方式了解。
<!--more-->

Vue 实例的 `$data`属性放置了全部的数据，所以对该属性的监听就可以实现我们需要的效果？

那问题在于，如何知道`$data`改变了？或者说如何判断何时发布广播告诉订阅者数据改变了？

## 实例化 Vue 对象
一个 viewModel 其实就是一个 Vue 实例。当初始化 Vue 实例时
```javascript
var vm = new Vue({
    el: '#app',
    data: {
        web: 'my web',
        books: ['first', 'second']
    }
})
```
对`data`属性进行遍历，
```javascript
var keys = Object.keys(data);// ['books', 'web']
```

然后复制到 Vue 实例的`$data`上？ 

```javascript
keys.forEach(function (key) {
    // 表示给 Vue 实例的 $data 属性添加属性
    Object.defineProperty(vm.$data, key, {
        set: function () {
            // 设置值，这里要手动把值进行替换
            data[key] = newValue;
            console.log('I set value')
        },
        get: function () {
            console.log('I get value')
            return options.data[key];
        }
    })
})
```

然后`$data`属性会变成这样：
```javascript
{
    web: 'my web',
    books: ['first', 'second']
}
```

## 测试是否有效

我们似乎已经完成了数据的监听，测试下当赋值时是否会触发相应的 `console.log`。
```javascript
vm.$data.web = 'your web';// console.log('I set value')
var list = vm.$data.books;// console.log('I get value')
```

看起来似乎可以了。

## 实现的代码 

先交代一下现在的目录结构：
```
- index.html
- src
    - index.js
    - scripts
        - viewModel.js
```
使用 gulp 将 src 文件夹内的 js 文件合并打包，index.html 文件将会引用最后的打包文件。scripts 文件夹内存放我们将要实现的 Vue.js。而 index.js 就是我们用来写业务的 js 文件。

上面的代码以es6 语法来写：
```javascript
// ./scripts/viewModel.js
export default class Vue {
  constructor (options) {
    this.data = options.data;
    this.$data = {};
    this.init();
  }
  init() {
    var keys = Object.keys(this.data);
    // 下面使用箭头函数就可以避免写这一句
    var vue = this;
    // copy property
    keys.forEach(function (key) {
      Object.defineProperty(vue.$data, key, {
        set: function () {
          // 设置值，这里要手动把值进行替换
          vue.data[key] = newValue;
          console.log('I set value');
        },
        get: function () {
          console.log('I get value')
          return vue.data[key];
        }
      })
    })
  }
}
// index.js
import Vue from './scripts/viewModel.js';

var vm = new Vue({
  data: {
    web: 'my web',
    books: ['first', 'second']
  }
})

vm.$data.web = 'your web';
console.log(vm.$data.books);
```
可以看到浏览器控制台输出我们预期的结果。我们成功“订阅”了数据变化事件。

### 问题
不过很明显，如果我们传入的 data 是这样的：
```javascript
var vm = new Vue({
  data: {
    web: {
        name: 'my web',
        url: 'localhost'
    },
    books: ['first', 'second']
  }
})

vm.$data.web.name = 'your web';
```
在控制台输出两条`I get value`，可能是因为用`.`来获得了`name`（触发了 web 属性的 get）吧，暂时不清楚。

但是可以肯定的是`name`属性是没有我们自己设置的`set` 和 `get` 的，所以我们需要对对象类型的值做遍历，每一个属性都加上`set`和`get`。

如果是数组或者字符串类型呢，数值类型呢？

数组一般而言会用到数组的方法来改变值，比如`push`、`shift`等，所以可以通过修改这些方法来实现订阅。

## 渲染页面

OK，我们实现了“订阅”后，只在控制台打印一条信息显然对我们没有什么帮助，我们需要能够在数据发生改变后也改变 DOM。所以这就要求数据和 DOM 是有关系的，通过某种手段，将数据和视图建立联系。

所以很明显，我们要把`get`和`set`中的`console.log`替换成有实际意义（渲染dom）的函数。

假设我们的`index.html`是这样的：
```html
<div id="app">
    <h2 v-bind="web"></h2>
    <ul v-bind="books"></ul>
</div>
```

`id = "app"`可以简化程序。。。方便我们查找绑定数据的区域。

那我们首先需要查找到需要渲染变量的标签：
```javascript
// 硬编码获取根节点。。。之后优化就是和 Vue 一样传入 el ，根据该值来获取根节点
var app = document.getElementById('app');

if(app.hasChildNodes()) {
    // 如果 app 有子元素
    app.childNodes.forEach(function (node) {
        var key = node.getAttribute('v-bind');// 获取到 v-bind 对应的值
        if(key && typeof vm.$data[key] === 'object') {
            // 这里只考虑数组和字符串两种，如果是数组
            var content = '';
            // 从 $data 中取出值，拼装成 html 插入节点
            vm.$data[key].forEach(function (value) {
                content += '<li>' + value + '</li>';
            })
            node.innerHTML = content;
        }else if(key && typeof vm.$data[key] === 'string') {
            // 如果是字符串
            node.innerHTML = vm.$data[key];
        }
    })
}
```

为了一开始页面就显示数据，这部分肯定要在初始化时就执行一次，然后数据改变时也要执行一次。

OK，我们的代码变成了这样：
```javascript
// ./srcipts/viewModel.js
export default class Vue {
  constructor (options) {
    this.data = options.data;
    this.$data = {};
    this.init();
    // 实例化 Vue 的时候就渲染页面
    this.render();
  }
  init() {
    var keys = Object.keys(this.data);
    var vue = this;
    // copy property
    keys.forEach(function (key) {
      Object.defineProperty(vue.$data, key, {
        set: function () {
          // 设置值，这里要手动把值进行替换
          vue.data[key] = newValue;
          console.log('I set value');
        },
        get: function () {
          console.log('I get value')
          return vue.data[key];
        }
      })
    })
  }
  render() {
    var app = document.getElementById('app');
    var vue = this;
    if(app.hasChildNodes()) {
      // 如果 app 有子元素
      app.childNodes.forEach(function (node) {
        // console.log(node);
        var key = node.getAttribute('v-bind');
        if(key && typeof vue.$data[key] === 'object') {
          // 这里只考虑数组和字符串两种，如果是数组
          var content = '';
          vue.$data[key].forEach(function (value) {
            content += '<li>' + value + '</li>';
          })
          node.innerHTML = content;
        }else if(key && typeof vue.$data[key] === 'string') {
          // 如果是字符串
          node.innerHTML = vue.$data[key];
        }
      })
    }
  }
}
```

但是实际运行时，却提示“getAttribute”不存在，将`node`打印出来，发现是一个“文本节点”，也就是我们`index.html`中的换行。。。。所以将`index.html`中的换行去掉：
```html
<div id="app"><h2 v-bind="web"></h2><ul v-bind="books"></ul></div>
```

然后页面就成功显示我们的数据了。

### 数据绑定测试

显然我们需要把`render`函数放到数据的`set`中去，每次修改数据，就重新渲染整个页面。

而怎么样才能触发数据改变呢？当然不能在代码里修改，所以只能通过事件，比如click、input等。这里增加一个输入框与一个按钮，点击按钮将输入框内的数据赋值给`$data.web`属性。

```javascript
// index.js
import Vue from './scripts/viewModel.js';

var vm = new Vue({
  data: {
    web: 'my web',
    books: ['first', 'second']
  }
})

var btn = document.getElementById('btn');

btn.onclick = function () {
  var web = document.getElementById('name').value;
  // 仅仅是赋值操作，并没有改变 dom
  vm.$data.web = web;
}

// ./scripts/viewModle.js
init() {
    var keys = Object.keys(this.data);
    var vue = this;
    // copy property
    keys.forEach(function (key) {
      Object.defineProperty(vue.$data, key, {
        set: function (newValue) {
          // 设置值，这里要手动把值进行替换
          vue.data[key] = newValue;
          // 增加渲染页面
          vue.render();
          // console.log('I set value');
        },
        get: function () {
          // console.log('I get value')
          return vue.data[key];
        }
      })
    })
}
```
```html
<!-- index.html -->
<div id="app"><h2 v-bind="web"></h2><ul v-bind="books"></ul></div>
<input type="text" id="name">
<button id="btn">update</button>
```

### 问题
很明显有一个问题，即当数据改变时，整个页面都会重新渲染（赋值）但是`web`值的改变不应该让`books`也重新渲染，如果页面一旦节点多起来，这应该对性能会影响很大？


> 当然，上面的功能不用绑定也能实现；点击按钮获取值并将查找 dom ，将获取到的新值替代原先的值；而我们现在实现的，是将查找 dom 并赋值的操作先写好，可以多次调用。本质上的确是一样的，但是如果先写好了查找 dom 并赋值的函数，就可以简化我们之后的工作。


## 总结
数据绑定的模式，很明显是将 DOM 方面的工作交给框架（Vue）来处理，我们只需要关心数据的改变，框架会自动去处理 DOM，这将大大简化我们的工作。

因为问题很多，所以接下来将对我们的代码进行优化（重写）。

