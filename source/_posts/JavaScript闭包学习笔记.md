---
title: JavaScript 闭包学习笔记
categories: JavaScript
tags:
- 闭包
date: 2017/02/21
---

闭包在 JavaScript 中一直都很"高端"，很多高级用法都会有闭包，但其实闭包就是作用域规则的"副产物"。

<!--more-->

什么是闭包？直接上代码：

```javascript
var name = "global"
function wrapper() {
  var name = "local"
  function echo() {
    4、-------------------------
    console.log(name)
  }
  // 2、-------------------------
  return echo
}
// 1、---------------------------
var foo = wrapper()
// 3、---------------------------
foo() // 打印 local，这就是闭包
```

## 详细解读

之前在学习闭包时，简单记忆为：
> 函数能够 **记住** 定义时作用域中的变量。

比如这里的`name="local"`，但是一直没有深入了解为什么能够访问。在了解了作用域链后，答案很明显，因为函数对象的`[[scope]]`属性。

像之前一样对作用域链的变化进行详细分析。

1、调用 wrapper() 前的作用域链：
```javascript
// 调用 wrapper() 前的作用域链
scopes = {
  0: {
    name: "global",
    wrapper: {
      name: "wrapper",
      [[scope]]: {
        0: {...}
      }
    },
    foo: undefined,
    // 以及全局变量与方法
  }
}
```

2、调用`wrapper`函数时的作用域链，将`warpper`的变量对象加入到自身的`[[scope]]`对应的`socpes`中，就是这样的：

```javascript
// 调用 wrapper 函数时
scopes = {
  1: {
    name: "local",
    echo: {
      name: "echo",
      [[scope]]: {
        1: {...},
        0: {...}
      }
    }
  },
  0: {
    name: "global",
    wrapper: {
      name: "wrapper",
      [[scope]]: {
        0: {...}
      }
    },
    foo: undefined,
    // 以及全局变量与方法
  }
}
```

重点在于`echo`对象的`[[scope]]`保存了两个变量对象，然后将这个`echo`对象返回。

3、调用`foo`函数前的作用域链：

```javascript
scopes = {
  0: {
    name: "global",
    wrapper: {
      name: "wrapper",
      [[scope]]: {
        0: {...}
      }
    },
    foo: {
      name: "foo",
      [[scope]]: {
        1: {...},
        0: {...}
      }
    },
    // 以及全局变量与方法
  }
}
```

这里和 `1、----` 处不同在于`foo`从`undefined`变成了对象，并且这个对象就是`3、----`处的`echo`对象。

4、调用 `foo` 函数时的作用域链：

```javascript
scopes = {
  2: {
    this: Window
  },
  1: {
    name: "local",
    echo: {
      name: "echo",
      [[scope]]: {
        1: {...},
        0: {...}
      }
    }
  },
  0: {
    name: "global",
    wrapper: {
      name: "wrapper",
      [[scope]]: {
        0: {...}
      }
    },
    foo: {
      name: "foo",
      [[scope]]: {
        1: {...},
        0: {...}
      }
    },
    // 以及全局变量与方法
  }
}
```

在`echo`函数内会寻找`name`变量，在作用域链的第一个变量对象中没有找到(`2`对应的对象)就往下继续找，在`1`中找到了，并且是`local`，所以最终打印出`local`。

同样以一张图片来更为直观的了解：

![作用域链变化](http://upload-images.jianshu.io/upload_images/3531509-2a6a9c6c4eb7a13e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 红色箭头指 2 处的作用域链来自 wrapper 的 [[scope]] 属性
- 橘色箭头指 echo 函数的 [[scope]] 属性等于 2 处加入 wrapper 变量对象后的作用域链
- 蓝色箭头指 3 处作用域链中全局变量对象 0 中的 foo 其实就是 echo(因为 wrapper 返回 echo 赋给 foo)
- 黑色箭头指 4 处的作用域链来自 foo(也就是 echo )的 [[scope]] 属性，而这个属性又指向 2 处的作用域链，所以是有两个变量对象

`echo`函数内的`console.log(name)`查找`name`时首先在自身的变量对象中寻找，没有找到后往下继续寻找，找到了`local`后就返回，所以不会再继续往下找到`global`。最终打印`local`。

## 随处都是闭包

> 当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。
—— 《你不知道的 JavaScript 》(上卷)

按照上面的定义，只要声明了函数，函数是能够记住当时的作用域链的，所以只要声明了函数，就产生了闭包？

```javascript
function foo() { 
  var a = 2;
  function bar() { 
    console.log( a ); // 2
  }
  bar(); 
}
foo();
```

 `bar`记住了定义时的作用域链，在这作用域链顶端的变量对象有`a: 2`的键值对，在其他任何地方调用`bar`都能够访问到`a`变量。

但是更严格来说，函数在当前词法作用域之外执行，才算闭包。

## 参考

- [JavaScript作用域学习笔记](http://www.jianshu.com/p/f921b622acfc)

