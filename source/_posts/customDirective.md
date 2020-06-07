---
title: 实现一个简单的 vue 无限加载指令
categories: Vue
tags: 
- vue
---

vue 中的自定义指令是对底层 dom 进行操作，下面以实现滚动到底部加载数据，实现无限加载来介绍如何自定义一个简单的指令。


无限加载的原理是通过对滚动事件对监听，每一次滚动都要获取到已滚动到距离，如果滚动的距离加上浏览器窗口高度，会大于等于内容高度，就触发函数加载数据。

<!--more-->
先介绍不使用 vue 的情况如何实现无限加载。

## 不使用框架

首先是html：

```html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>实现滚动加载</title>
    <style>
        * {
            -webkit-box-sizing: border-box;
            -moz-box-sizing: border-box;
            box-sizing: border-box;
        }
        li, ul {
            list-style: none;
        }
        .container {
            width: 980px;
            margin: 0 auto;
        }
        .news__item {
            height: 80px;
            margin-bottom: 20px;
            border: 1px solid #eee;
        }
    </style>
</head>
<body>
    <div class="container">
        <ul class="news" id="news">
            <li class="news__item">1、hello world</li>
            <li class="news__item">2、hello world</li>
            <li class="news__item">3、hello world</li>
            <li class="news__item">4、hello world</li>
            <li class="news__item">5、hello world</li>
            <li class="news__item">6、hello world</li>
            <li class="news__item">7、hello world</li>
            <li class="news__item">8、hello world</li>
            <li class="news__item">9、hello world</li>
            <li class="news__item">10、hello world</li>
        </ul>
    </div>
    <script>
    </script>
</body>
</html>
```

打开浏览器，调整下浏览器窗口高度，让页面可以滚动。

先了解三个变量

- document.body.scrollTop 滚动条滚动的距离

- window.innerHeight 浏览器窗口高度

- document.body.clientHeight 内容高度



对应上面的原理就是

```javascript

window.addEventListener('scroll', function() {
    var scrollTop = document.body.scrollTop;
    if(scrollTop + window.innerHeight >= document.body.clientHeight) {
        // 触发加载数据
        loadMore();
    }
});
function loadMore() {
    console.log('加载数据')
}
```

loadMore() 函数就是从接口获取到数据，组装 html，插入到原先到节点后面。

```javascript

// 表示列表的序号
var index = 10;
function loadMore() {
    var content = '';
    for(var i=0; i< 10; i++) {
        content += '<li class="news__item">'+(++index)+'、hello world</li>'
    }
    var node = document.getElementById('news');
    // 向节点内插入新生成的数据
    var oldContent = node.innerHTML;
    node.innerHTML = oldContent+content;
}
```

这样就实现了无限加载。



## 在 vue 中使用指令实现

为什么要用指令实现呢？好像只有指令是可以获取到底层 dom 的？而实现无限加载，是需要拿到内容高度的。



首先初始化一个项目，添加一个组件，用来显示列表。

```javascript

// components/Index.vue

<template>
    <div>
        <ul class="news">
          <li class="news__item" v-for="(news, index) in newslist">
            {{index}}-{{news.title}}
          </li>
        </ul>
    </div>
</template>
<style>
  .news__item {
    height: 80px;
    border: 1px solid #ccc;
    margin-bottom: 10px;
  }
</style>
<script>
    export default{
        data(){
            return{
              newslist: [
                {title: 'hello world'},
                {title: 'hello world'},
                {title: 'hello world'},
                {title: 'hello world'},
                {title: 'hello world'},
                {title: 'hello world'},
                {title: 'hello world'},
                {title: 'hello world'},
                {title: 'hello world'},
                {title: 'hello world'}
              ]
            }
        },
        components:{
        }
    }
</script>
```

OK，现在开始编写指令。从传统实现中，我们了解到要注册对滚动事件对监听，同时拿到内容高度。



```javascript

directives: {
  scroll: {
    bind: function (el, binding) {
      window.addEventListener('scroll', ()=> {
        if(document.body.scrollTop + window.innerHeight >= el.clientHeight) {
          console.log('load data');
        }
      })
    }
  }
}
```

首先是在组件内注册了 scroll 指令，然后在指令第一次绑定到组件时，也就是对应着 `bind`钩子，注册滚动监听。



> 钩子函数就是一些生命周期改变时会调用的函数。bind 在第一次绑定到组件时调用、unbind 在指令与组件解绑时调用。



还可以注意到 bind 对应到函数有两个参数，el 和 binding，这是钩子函数参数，比如 el 对应绑定的节点，binding 有很多数据，比如传给指令的参数等。



下面的`el.clientHeight`就是表示获取绑定指令的这个节点的内容高度。



和之前一样，判断滚动的高度加上窗口高度是否大于内容高度。



绑定指令到节点上：

```html

<template>
    <div v-scroll="loadMore">
        <ul class="news">
          <li class="news__item" v-for="(news, index) in newslist">
            {{index}}-{{news.title}}
          </li>
        </ul>
    </div>
</template>
```

可以看到给指令传了一个值，这个值就是加载数据的函数：

```javascript

methods: {
  loadMore() {
    let newAry = [];
    for(let i = 0; i < 10; i++) {
      newAry.push({
        title: 'hello world'
      })
    }
    this.newslist = [...this.newslist, ...newAry];
  }
}
```
当然，现在在滚动到底部时，只会打印`load data`，只要把这里改成调用函数就OK了：
```javascript
window.addEventListener('scroll', ()=> {
  if(document.body.scrollTop + window.innerHeight >= el.clientHeight) {
    let fnc = binding.value;
    fnc();
  }
})
```
`v-scroll="loadMore"`的 `loadMore`可以在钩子函数参数的 binding 上拿到。

至此，一个简单的指令就完成了。

## 优化

上面的例子并没有真正从接口获取数据，所以存在一个隐藏的 bug：当接口响应很慢的情况，滚动到底部正在加载数据时，稍微滚动一下仍会触发获取数据函数，这会造成同时请求多次接口，一次性返回大量数据。

解决办法是添加一个全局变量 scrollDisable，当第一次触发加载数据函数时，将该值设置为 true，根据该值判断是否要执行加载函数。以普通实现为例：
```javascript
var scrollDisable = false;
window.addEventListener('scroll', function() {
    var scrollTop = document.body.scrollTop;
    if(scrollTop + window.innerHeight >= document.body.clientHeight) {
        // 触发加载数据
        if(!scrollDisable) {
            loadMore();
        }
    }
});
// 表示列表的序号
var index = 10;
function loadMore() {
    // 开始加载数据，就不能再次触发这个函数了
    scrollDisable = true;
    var content = '';
    for(var i=0; i< 10; i++) {
        content += '<li class="news__item">'+(++index)+'、hello world</li>'
    }
    var node = document.getElementById('news');
    // 向节点内插入新生成的数据
    var oldContent = node.innerHTML;
    node.innerHTML = oldContent+content;
    // 插入数据完成后
    scrollDisable = false;
}
```


## 参考

- [自定义指令](http://cn.vuejs.org/v2/guide/custom-directive.html)

