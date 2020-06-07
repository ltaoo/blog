---
title: 自执行函数(IIFE)
categories: JavaScript
---

JavaScript中存在一种写法：
```javascript
(function () {
    console.log('hello')
}())
//or
(function () {
    console.log('hello')
})()
```
可以看到在`()`内定义函数，然后又使用`()`来执行该函数。执行函数的`()`有两种位置，包裹函数的`()`内或者外。
<!-- more -->

## 作用
了解了写法后，这种函数有什么意义或者作用？

### 避免污染全局变量
我们知道使用`var`关键字声明的变量在当前作用域是全局的：
```javascript
var a = 'global';

function fnc() {
    console.log(a);//'global'
}
```
如果项目需要多人合作，假设A实现一个功能，比如计算当天是本月第几周并打印：
```javascript
var time = new Date();
//some code
console.log('当天是本月第' + time + '周');
```
A 将代码写在 A.js 文件中，index.html 中引入该文件。OK，一切正常。此时B也需要实现一个功能，打印当前时间：
```javascript
var time = new Date();
console.log(time);
```
B 觉得很简单啊，新建了B.js 文件，在 index.html 中 A.js 文件后面的位置引入自己的 B.js，打开网页一看，打印出了自己需要的数据，OK，完美。C 也有工作要做，根据当前是第几周来决定是否要给页面加一些特效，A 告诉他直接获取 `time`变量就可以，不用自己去实现计算第几周。C 于是就直接拿来用，结果肯定不对，因为 `time`被 B 给覆盖掉了。

这里是 B 覆盖掉了 A 的值，如果 B 采用自执行函数的写法：
```javascript
(function () {
var time = new Date();
console.log(time);
}());
```
仍然可以实现“打印当前时间”的功能，同时 C 还是可以获取到 `time`这个变量的值是第几周而非当前时间。

> 事实上，我们无法预知是否会覆盖掉别人的变量或者被别人覆盖。如果需要被别人使用，向外暴露最少的变量，以免被覆盖；如果不需要被别人使用，直接使用自执行函数将会更好。

OK，了解了用法，接下来了解下为什么可以实现不污染全局变量。
> 而对于在函数内用 var 关键字声明的局部变量来说，当退出函数时，这些局部变量即失去了他们的价值，他们都会随着函数调用的结束而被销毁。

对于上面的代码而言，定义了函数并立即执行内部的代码，执行完毕后，`time`就被销毁了。
看到这里，那如果先定义函数，再执行这个函数不是一样的吗？
```javascript
function fnc() {
    var time = new Date()
    console.log(time)
}
fnc();
```
的确没有污染`time`这个变量了，但是很明显相比自执行函数，多出了`fnc`变量。而这个变量又有覆盖其他变量的可能，所以最好变量越少越好。**定义一个对象，在对象中定义函数进行计算，最后返回需要的值，这样似乎是“最佳实践”？**


### 封装变量
其实封装变量的目的就是为了减少全局变量，避免污染全局变量。不过封装变量往往配合闭包使用。
```javascript
var cache = {};

var mult = function () {
    var args = Array.prototype.join.call(arguments, ',');
    if(cache[args]) {
        return cache[args];
    }
    var a = 1;
    for(var i = 0, l = arguments.length; i< l; i++) {
        a = a + arguments[i];
    }
    
    return cache[args] = a;
}
```
书上的例子，`cache`变量只在函数`mult`中使用，但是又不能将其放在函数内部。针对这种其他，就需要使用到封装变量，将`cache`变量封装，外部无法获取变量，只有`mult`函数可以访问到`cache`。
```javascript
var mult = (function () {
    var cache = {};
    return function mult() {
        var args = Array.prototype.join.call(arguments, ',');
        if(cache[args]) {
            return cache[args];
        }
        var a = 1;
        for(var i = 0, l = arguments.length; i< l; i++) {
            a = a + arguments[i];
        }
        
        return cache[args] = a;
    }
}());

console.log(mult(1, 2));
console.log(mult(1, 2));
console.log(mult(1, 3));
```

很容易就想到`for`循环中的变量`i`刚好符合这种情况，如果同一作用域有多个`for`循环，我们就要用到变量`j`、`k`...很显然这样不方便，所以可以将变量`i`封装，只有`for`循环可以拿到变量`i`。

```javascript
(function () {
    for(var i = 0; i < 10; i ++) {
        console.log(i);
    }
}())
console.log(i);//i is not defined
```

### 惰性加载

在查找相关说明时，很多人提到可以实现惰性加载。惰性加载是指在某些需要很多if判断的情况但永远只匹配一种情况，在判断条件的同时，改写了函数本身为符合条件的情况。
```javascript
var addEvent = (function(el,type,handler){ 
    if(el.addEventListener) {
        addEvent = function (el, type, handler) {
            el.addEventListener(type, handler, false);
        }
    }else{
        addEvent = function (el, type, handler) {
            el.attachEvent('on' + type, handler);
        }
    }
    return addEvent;
})();
```
根据浏览器的兼容性来判断使用什么方法来添加监听器。如果是 IE 就用`attachEvent`，如果是其他浏览器就用`addEventListener`。如果每次需要添加监听器都这么判断肯定很麻烦，所以只做一次判断，判断的同时改写了函数本身。

- 如果是IE 浏览器，addEvent = function () {...};
- 如果是其他浏览器，addEvent = function () {...};

这里用自执行函数的目的是什么？其实不用也可以啊，只要执行一次`addEvent`函数，该函数就被改写了。

比如某网站，在浏览每个页面时会根据用户的类别来给用户打招呼。
- 如果是会员，就说“欢迎”
- 如果不是会员，就说“来买会员吧”

```javascript
var person = {
    type: 'member'
};
var say = (function (person) {
    if(person.type == "member") {
        console.log('如果是会员');
        say = function () {
          console.log('欢迎');
        }
    }else {
        say = function () {
          console.log('来买会员吧');
        }
    }
    return say;
})(person);

say();
say();
```

可以看到只输出了一次"如果是会员"，这一次在立即执行时输出的，而不是在第一次调用`say()`函数时输出的。第二次`say()`函数的调用就直接打印了“欢迎”，而没有打印“如果是会员”表示没有经过判断的步骤。

> 会员的类型是确定的，不会访问这个网页时是会员，访问另一个网页又不是会员了。记住，虽然有很多判断，但是无论经过多少次判断，都是一种情况，所以干脆把这种情况保存下来，以后再用就不判断了，反正知道是一样的。

其实这里不用自执行函数也可以，只是会多出一个变量：
```javascript
var person = {
    type: 'member'
};
var say;
var temp = function (person) {
    if(person.type == "member") {
        console.log('如果是会员');
        say = function () {
          console.log('欢迎');
        }
    }else {
        say = function () {
          console.log('来买会员吧');
        }
    }
};
temp(person);

say();
say();
```

可以看到和上面的写法输出是一样的。

### JQ 插件/命名冲突
看到很多jquery插件是这么写的：
```javascript
(function ($) {
    // start coding
}(jQuery))
```
还有这么写的
```javascript
(function (window) {
    // start coding
    window.console.log('hello');
}(Window))
```
有一个优点，即如果存在另一个第三方类库，同样是使用`$`作为简写形式，就导致了冲突，而采用上面的写法，由于将`jQuery`作为参数传递给了自执行函数，所以该函数内部的`$`变量就肯定是`jQuery`了。




##总结
IIFE 其实就是为了减少变量，避免污染全局空间。无论是封装变量还是惰性函数，其实不用 IIFE 也是可以实现的。


##　参考
- 《javascript设计模式与开发实践》



