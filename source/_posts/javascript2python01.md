---
title: 写给 JSer 的 python 学习手册 - 基础语法篇
categories: Python
tags:
- python
date: 2018/1/7
---

作为一个标榜“程序员”而非“前端开发”的前端开发，不应该只会一门`JavaScript`，但学习新语言累觉不爱。

能否将`JavaScript`的经验借鉴到其他语言的学习上呢？毕竟它们都是“编程语言”，提取共同点，突出差异点，将新语言学习的成本降到最低。

总体来说，编程语言有几大核心

- 语法
- 数据类型
- 内置库
- 第三方库
- 工程相关（包/依赖、测试、构建）

当然有些语言有更多内容，而有些只有部分，这就要在学习的过程中不断补充完善。

所以，这系列博客，将从这几个核心依次介绍，最终实现能够阅读`python`实际项目代码。

<!--more-->

## 语句

`python`没有命令结束标志，即不用分号结尾；`javascript`以`;`作为语句的结束标志也可以不加。

> 其实`JavaScript`会自动判断语句是否结束，比较经典的例子就是
> 
```javascript
function foo(params) {
    return
    params;
}
```
从语义上来说，是想返回`params`，但其实在`return`这行就结束了，最终函数返回`undefiend`。

## 代码块

`python`使用缩进来控制层级，或者说代码块，不过不同于`JavaScript`可以随意得到一个块，`python`只有三种情况才有代码块

- 在`for...in`循环
- `if`判断
- 函数声明


下面会再介绍。

## 变量声明

```python
# python
foo = 'bar'
```
不需要关键字，对应`JavaScript`代码则是：

```javascript
// javascript
let foo = 'bar';
```

## 运算符

由于运算符很多，这里只挑一些特殊的介绍：

### 比较运算符

不同于`JavaScript`有`===`和`==`的区别，`python`只有`==`，比较两个对象是否相等。

`!=`和`<>`表示相同含义，即不等于。

### 逻辑运算符

表示”并且“关系，使用`and`，表示”或“使用`or`，”非“使用`not`。

对应`JavaScript`则分别是`&&`、`||`和`!`。


## 函数

### 函数声明

声明一个空函数，

```python
# python
def foo():
    pass
```

`python`使用`def`关键字表示声明一个函数，紧接着是函数名、括号、括号中的参数和冒号`:`，下一行缩进，编写函数体。如果是空函数，则需要使用`pass`作为函数体。

```javascript
// javascript
function foo() {}
```

### 函数调用

```python
# python
print('Hello Python');
```

即函数/方法后面接`()`，参数置于括号内。

```javascript
// javascript
console.log('Hello Javascript');
```

### 参数

和`JavaScript`一样，声明函数时指定参数的位置，调用时按照顺序传递实参即可。

```python
# python
def foo (x, n):
    print('first is', x)
    print('second is', n)

foo(5, 2)
```

```javascript
// javascript
function foo(x, n) {
    console.log('first is', x);
    console.log('second is', n);
}
foo(5, 2);
```

### 默认参数

```python
# python
def foo(x, n = 2):
    print('first is', x)
    print('second is', n)

foo(5)
```

```javascript
// javascript
function foo(x, n = 2) {
    console.log('first is', x);
    console.log('second is', n);
}
foo(5);
```

### 可变参数

如果参数个数不确定，怎么办？

```python
# python
def calc(*numbers):
    print(len(numbers))

calc(10, 21) # 2
calc(0) # 1
```

`numbers`是一个列表（数组），保存了所有传入的参数。

```javascript
// javascript
function calc(...args) {
    console.log(args.length);
}
```

只是符号不同，`args`也是数组。

### 关键字参数

```python
# python
def person(name, age, **kw):
    print('name', name, 'age', age, 'other', kw)

person('ltaoo', 18, city='Hangzhou')
# ('name', 'ltaoo', 'age', 18, 'other', {'city': 'Hangzhou'})
```

得到的`kw`值是一个对象，键为参数名，值为对应的值。

`JavaScript`中没有这个概念，也无法模拟，因为`JavaScript`没有严格意义上的”命名参数“。这里是第一个`python`的特殊点 [1]

### 命名参数

前面的例子：

```python
# python
def foo (x, n):
    print('first is', x)
    print('second is', n)

foo(n = 10, x = 4)
# ('first is', 4)
# ('second is', 10)
```

`JavaScript`中虽然没有命名参数，但可以使用解构模拟：

```javascript
// javascript
function foo({ x, n }) {
    console.log('first is', x);
    console.log('second is', n);
}
foo({ n: 10, x: 4 });
```

## 循环

`python`中有两种循环，`for...in`和`while`。

### for...in 循环

```python
# python
for item in ary:
    print(item)
```

对应的`JavaScript`则是

```javascript
// javascript
for(let item of ary) {
    console.log(item);
}
```

### for 循环

但是`for...of`在`JavaScript`中是新特性啊，`python`没有`JavaScript`中常用的`for`循环，不过可以使用`for...in`实现类似的

```python
# python
for i in range(len(ary)):
    num = ary[i]
```

首先使用`len()`函数获取`ary`的长度，再使用`range()`得到从 0 开始，`ary`长度结束的数组，如`range(5)`得到`[0, 1, 2, 3, 4]`，再遍历该数组。

```javascript
// javascript
for (let i = 0, l = ary.length; i < l; i += 1) {
    const num = ary[i];   
}
```

一般用在需要使用下标的情况。

#### 步长

那`python`如何控制步长？

```python
# python
for i in range(0, len(ary), 2):
    num = ary[i]
```

步长则为2，本质上来说并不是步长为2，而是将遍历的数组间隔变为了2，还是以上面为例：`range(0, 5, 2)`得到`[0, 2, 4]`。

对应的`javascript`实现：

```javascript
// javascript
for (let i = 0, len = ary.length; i < len; i += 2) {
    let num = ary[i];
}
```

> 这么一比感觉`for`循环更灵活。


### while 循环

```python
# python
sum = 0
n = 99
while n > 0:
    sum = sum + n
    n = n - 2
print(sum)
```

只要满足条件，就执行循环体。对应`javascript`：

```javascript
// javascript
let sum = 0;
let n = 99;
while (n > 0) {
    sum = sum + n;
    n = n - 2;
}
console.log(sum);
```

## 条件判断

`python`中只有`if..else`，没有`switch`。

```python
# python
if a > b:
    print('yes')
elif a == b:
    print('same')
else:
    print('no')
```

对应的`JavaScript`则是：

```javascript
// javascript
if(a > b) {
    console.log('yes');
} else if (a === b) {
    console.log('same');
} else {
    conosle.log('no');
}
```

## 实践

有 29, 12, 49, 50, 9, 8 六个数字，要求对每个数字进行判断，

- 如果大于等于 50，则打印 max 字符串和该数字
- 如果大于等于 10 且小于 50，则打印 middle 字符串和该数字
- 否则打印 min 字符串和该数字

> 小提示: `python`中数组也可以直接使用字面量声明。


## 总结

这里只是列出了一部分，还有更多内容比如

- python 有变量提升吗？
- 函数参数是按值传递还是按引用传递？


之类，就自己探索吧！


## 延伸阅读

- [冒号是必要的吗？](https://docs.python.org/3/faq/design.html#why-are-colons-required-for-the-if-while-def-class-statements)



