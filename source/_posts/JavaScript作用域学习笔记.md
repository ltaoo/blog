---
title: JavaScript 作用域学习笔记
categories: JavaScript
date: 2017/02/21
---

总结来说很简单：

- 函数运行在定义时的作用域中
- 变量查找会从当前作用域开始查找，找不到则到下一层作用域查找，直到找到并返回或者返回 undefined

<!--more-->

## 实际例子

```javascript
var name = "global"
function echo() {
  console.log(name)
}

echo()
```

打印`global`毫无疑问，因为在`echo()`函数作用域没有找到，就到外层作用域中寻找，找到了值为`global`的`name`变量。

## 如何查找

在调用某个函数时，会从函数这个对象上拿到`[[scope]]`属性对应的**作用域链**，并且将函数的变量对象也放入到这个作用域链中，而该作用域链是在函数定义时就确定了的。

同样是上面的例子，作用域链的变化是这样的：

```javascript
// 调用 echo() 函数前
scopes = {
  0: {
    name: 'global',
    echo: {
      name: 'echo',
      function: function () {console.log(name)},
      [[scope]]: scopes
    },
    console,
    parseInt: function () {...}
    // 等一些全局变量与方法
  }
}
```

重点在`echo`对象上有`[[scope]]`属性，指向的就是最外面的这个`scopes`，即作用域链。

在调用`echo()`后，会将该函数的变量对象：

```javascript
{
  this: Window
}
```

放到`echo`对象的`[[scope]]`属性对应的`scopes`上，那就变成了这样：

```javascript
// 调用 echo() 时
scopes = {
  1: {
    this: Window
  },
  0: {
    name: 'global',
    echo: {
      name: 'echo',
      function: function () {console.log(name)},
      [[scope]]: scopes
    },
    console,
    parseInt: function () {...}
    // 等一些全局变量与方法
  }
}
```

在`echo`函数内寻找`name`就是指会先在

```javascript
{
  this: Window
}
```

这个变量对象上寻找名为`name`的键，如果没有找到就向下(将 0 视为下)寻找，所以就会在：

```javascript
{
    name: 'global',
    echo: {
      name: 'echo',
      function: function () {console.log(name)},
      [[scope]]: scopes
    },
    console,
    parseInt: function () {...}
    // 等一些全局变量与方法
}
```

这个变量对象上寻找，然后找到了值为`global`的`name`并返回。

## 进阶问题
尝试解答下面代码会打印出什么？

```javascript

var name = "global"

function echo () {
  console.log(name) // 打印什么？
}  

function change() {
  var name = "local"
  echo()
}

change() 
```

### 具体分析

按照上面的分析过程，在调用`change()`之前，作用域链应该是这样的：

```javascript
// 调用 change() 前
scopes = {
  0: {
    name: 'global',
    echo: {
      name: 'echo',
      function: function () {console.log(name)},
      [[scope]]: scopes
    },
    change: {
      name: 'change',
      function: function () {console.log(name)},
      [[scope]]: scopes
    },
    console,
    parseInt: function () {...}
    // 等一些全局变量与方法
  }
}
```

多了`change`这个键值对，其他没什么，然后调用`change`函数：

```javascript
// 调用 change 函数时
scopes = {
  1: {
    name: 'local',
    this: Window
  },
  0: {
    name: 'global',
    echo: {
      name: 'echo',
      function: function () {console.log(name)},
      [[scope]]: scopes
    },
    change: {
      name: 'change',
      function: function () {console.log(name)},
      [[scope]]: scopes
    },
    console,
    parseInt: function () {...}
    // 等一些全局变量与方法
  }
}
```

如果在`change`函数内寻找`name`变量，找到的肯定是`local`毫无疑问。最后是`echo`函数的调用，也是这段代码的意义所在，调用时是这样的：

```javascript
// 调用 echo 函数时
scopes = {
  1: {
    this: Window
  },
  0: {
    name: 'global',
    echo: {
      name: 'echo',
      function: function () {console.log(name)},
      [[scope]]: scopes
    },
    change: {
      name: 'change',
      function: function () {console.log(name)},
      [[scope]]: scopes
    },
    console,
    parseInt: function () {...}
    // 等一些全局变量与方法
  }
}
```

所以在`echo`函数内寻找不到`name`，就到下层变量对象找，找到了`global`。所以这段代码最终是打印出了`global`。

图片可能更为直观，灰色块为作用域链，即`scopes`，绿色块为变量对象，如下所示 ↓

![模拟流程](http://upload-images.jianshu.io/upload_images/3531509-e366cf6f9cac6e13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 浅拷贝？

虽然到目前为止能够解释绝大部分的作用域问题，但还存在疑问：

```javascript
[[scope]]: scopes
```
这里是指
> 在一个函数被定义的时候, 会将它定义时刻的scope chain链接到这个函数对象的[[scope]]属性。

我们都知道，对象是引用类型，那从 2 -> 3 的过程中，尤其是`echo`函数执行时，先从`echo`函数上拿到`[[scope]]`属性，这时候不应该是：

```javascript
scopes = {
  1: {
    name: 'local',
    this: Window
  },
  0: {
    name: 'global',
    echo: {
      name: 'echo',
      function: function () {console.log(name)},
      [[scope]]: scopes
    },
    change: {
      name: 'change',
      function: function () {console.log(name)},
      [[scope]]: scopes
    },
    console,
    parseInt: function () {...}
    // 等一些全局变量与方法
  }
}
```

这样的吗，因为`[[scope]]`保存的是`scopes`的内存地址啊，所以我能够自己解释为
> 在函数定义时，将`scopes`进行浅拷贝并保存到`[[scope]]`属性上。


所以上面的例子，作用域链严格来说应该是这样的：

```javascript
scopes = {
  0: {
    name: 'global',
    echo: {
      name: 'echo',
      function: function () {console.log(name)},
      [[scope]]: {
        0: {...}
      }
    },
    change: {
      name: 'change',
      function: function () {console.log(name)},
      [[scope]]: {
         0: {...}
      }
    },
    console,
    parseInt: function () {...}
    // 等一些全局变量与方法
  }
}
```

但是真的是这样吗？

## 参考

-  [Javascript作用域原理](http://www.laruence.com/2009/05/28/863.html)

