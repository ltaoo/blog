---
title: this 指向问题深究
categories: JavaScript
tags:
- this
date: 2017/04/11
---

这个问题属于老生常谈了，我们都知道作为对象方法调用时，方法内的`this`指向这个对象，用一句话来形容就是

> 谁调用了函数，`this`就指向谁。

具体是为什么呢？
<!--more-->

## this 指向

这里列出`this`指向的 4 种情况

- 作为对象的方法调用
- 作为普通函数调用
- 构造器调用
- call 或 apply

### 作为对象的方法调用

就是一开始提到的，`this`指向该对象。

```javascript
var obj = {
	a: 1,
	getA () {
		console.log(this === obj)
		console.log(this.a)
	}
}


obj.getA() // true 和 1
```

`getA`函数作为`obj`对象的方法调用，函数内的`this`就是指向这个对象`obj`。

有一个很常见的场景就是`dom`绑定事件，比如：

```javascript
document.querySelector('#id').onclick = function () {
	console.log(this.id)
}
```

`this`肯定指向`document.querySelector('#id')`这个对象，因为`onclick`函数是作为该对象的方法调用的。

### 作为普通函数调用

同样是上面的代码，将`getA`单独进行调用

```javascript
var bar = obj.getA

bar() // false 和 undefined
```

当函数不作为对象的属性被调用，函数内的`this`指向全局对象。

> 在 ECMAScript 5 的 `strict`模式下`this`不再指向全局对象，而是`undefined`。这是否意味着之前指向全局是错误的？


### 构造器调用

```javascript
function Person(name) {
	this.name = name
}

var ltaoo = new Person('ltaoo')
console.log(ltaoo.name) // ltaoo
```

在写面向对象的 JavaScript 时用到的构造函数，我们都知道构造函数和原型方法中的`this`是指向返回的对象的。上面使用`new`返回了`ltaoo`这个对象，所以`this.name = name`就是`ltaoo.name = name`。

### call 和 apply

即显式指定`this`值。

```javascript
var obj1 = {
	name: 'hello',
	getName () {
		return this.name
	}
}

var obj2 = {
	name: 'ltaoo'
}

obj1.getName() // hello
obj1.getName.call(obj2) // ltaoo
```

## 为什么

上面的知识点随便都能查到，但也仅限于此，为什么`this`在不同情况下是不同的值呢？

### 作为普通函数调用

```javascript
function foo() {
	console.log(this)
}
foo()
```
如果是在浏览器环境下，并且是非严格模式，肯定打印出`window`，有人说是因为`foo`在全局环境下声明，所以是挂载在全局对象下，所以实际上是这样的`window.foo()`，这和我们之前的知识点也是吻合的：

> 谁调用函数，`this`就指向谁。

```javascript
function foo () {
	function bar () {
		console.log(this)
	}
	bar()
}

foo()
```
答案还是`window`，但此时`window`上有`bar`吗，很明显没有，所以上面的说法不正确。

再来看一个例子：

```javascript
var name = 'hello'
var obj = {
	name: 'ltaoo',
	getName () {
		console.log(this.name)
	}
}
obj.getName() // ltaoo
```
OK，毫无疑问是吧，如果修改成这样呢？

```javascript
;(0 || obj.getName)()
```
分析一下，首先是执行`(0 || obj.getName)`，所以返回`obj.getName`这个函数，再调用这个函数，结果是`hello`。

当然这里似乎是符合了**作为普通函数调用**的规则。

### 引用类型

从这里开始就很虚幻了，引用类型可以表示为拥有两个属性的对象：

```javascript
var referenceType = {
	base: <base object>,
	propertyName: <property name>
}
```

那这个引用类型在哪里出现，从哪里获取值（base 和 propertyName 的值）呢？

有两种情况

* 当处理一个标识符时
* 处理属性访问器时

#### 处理标识符

首先明确标识符是什么，标识符是变量名、函数名、函数参数名和全局对象中未识别的属性名。

```javascript
var foo = 10
```

这样就有了一个`referenceType`：

```javascript
fooReferenceType = {
	base: window,
	propertyName: 'foo'
}
```

当在使用`foo`这个变量时，内部会去查找该变量对应的值，类似这样：

```javascript
function getValue (value) {
	var base = getBase(value)
	return base.[[Get]](getPropertyName(value))
}

foo // getValue(fooReferenceType) => window.foo => 10
```

#### 属性访问器

```javascript
foo.bar()

foobarReferenceType = {
	base: foo,
	propertyName: 'bar'
}

foo.bar // getValue(foobarReferenceType) => foo.bar => function
```

### referenceType 到 this

那到底怎么样从`referenceType`可以推出`this`值呢？

> 在一个函数上下文中，`this`由调用者提供，由调用函数的方式来决定。如果调用括号`()`左边是引用类型的值，`this`将设为引用类型值的`base`对象，在其他情况下，这个值为`null`。当`this`值为`null`时，会被转换为全局对象。

OK，这是整篇笔记的核心，来举个例子：

```javascript
function foo() {
	console.log(this)
}

foo()
```

看到函数名，意识到这是一个标识符，所以有`referenceType`为：

```javascript
fooReferenceType = {
	base: window,
	propertyName: 'foo'
}
```

怎么说明这不是第二种情况，即`()`坐标不是引用类型的值呢？

```javascript
;(0 || foo)()
```

这种情况上面也提到了，先执行`(0 || foo)`，这时已经遇到`foo`标识符，所以先`getValue`了，再`()`调用，就符合了第二种情况，`()`左边不是引用类型值，所以`this`为`null`，但是会被转换为全局对象。

```javascript
function foo () {
	'use strict'
	console.log(this)
}

foo()
;(0 || foo)()
```

两次打印都是`undefined` ？ 为什么，头疼。

## 参考

- [js函数中的this](https://juejin.im/post/58ea15b62f301e006247bdd9)
- [匿名函数的this指向为什么是window?](https://www.zhihu.com/question/21958425)
- [深入理解JavaScript系列（13）：This? Yes,this!](http://www.cnblogs.com/TomXu/archive/2012/01/17/2310479.html)
- [从Ecma规范深入理解this](https://segmentfault.com/a/1190000003906484)

