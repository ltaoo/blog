---
title: “[1,2,3]”是JSON吗？
categories: JavaScript
tags:
- json
date: 2017/12/30
---

A: “这个接口我传个 JSON 给你，格式是这样的 `'[1, 2, 3]'`”
B: “等下，这不是数组吗，JSON 应该有键啊，类似这样才行`'{ "key": [1, 2, 3] }'`”
A: “不，这就是 JSON 格式的数据”
B: “啊，是吗？”

如果你对上面的对话也心存疑虑，可以继续往下看。

<!-- more -->

## 什么是 JSON

首先应该明确`JSON`是什么，再讨论`[1, 2, 3]`是不是。

> JSON(JavaScript Object Notation) 是一种轻量级的**数据交换格式**。


它仅仅是一种格式，就好比厨师脑中的食谱，这道菜有什么材料。而从原材料变成成品给顾客食用，这个过程食谱并没有实际参与，只是一个指导作用。

所以`JSON`也是不存在于我们的程序中，不存在任何地方，只是一个思维，存在我们脑海里。

这个思维，就是`JSON`这种格式应该包含哪些元素。

## JSON 格式

`JSON`建构于两种结构：

- “名称/值”对的集合
- 值的有序列表

从这里就能看出`[1, 2, 3]`是符合`JSON`格式的。
> 但不能说`[1, 2, 3]`是一个`JSON`，它在`javascript`中可以被转换为数组，也可以在其他语言中被转换为数组（如果有这种类型）。而之所以只有这两种，是因为大部分现代计算机语言都支持。

那`[1, 2, undefined, 3]`符合`JSON`格式吗？

简单一想，
> “既然结构要是计算机语言都支持的，那结构中的值也需要吧，而`undefined`是`javascript`独有的，其他语言并没有，所以不符合`JSON`格式。”

bingo！合法的`JSON`值可以是以下六种：

- string
- number
- boolean
- null
- object
- array

所以下面的数据格式是合法的`JSON`：

```json
{
    "person": {
        "name": "ltaoo",
        "age": 18,
        "skills": ["javascript", "html", "css"]
    },
    "happy": true
}
```

## JavaScript 中的 JSON

### 传递的到底是什么？

在`JavaScript`中，如果在请求接口时要传递数据，我们往往会说“传一个`JSON`”，从上面已经知道`JSON`只是一个格式，那我们传递的到底是什么？

```javascript
// 一个简单的 post 请求
const body = {
    name: 'ltaoo',
    age: 18
};

fetch('https://easy-mock.com/mock/5a1d30028e6ddb24964c2d91/business/api/login', {
    method: 'POST',
    body,
})
    .then((res) => res.json())
    .then((data) => {
        console.log(data);
    });
```

但实际上并没有将参数传递过去，即`Headers`中并不存在`Request Payload`，需要将`body`使用`JSON.stringify()`方法转换为一个字符串后，才能成功传递。

```javascript
const body = {
    name: 'ltaoo',
    age: 18
};

fetch('https://easy-mock.com/mock/5a1d30028e6ddb24964c2d91/business/api/login', {
    method: 'POST',
    body: JSON.stringify(body),
})
    .then((res) => res.json())
    .then((data) => {
        console.log(data);
    });
```

> `JSON.stringify()`方法是将一个JavaScript值(对象或者数组)转换为一个 JSON 字符串。

所以实际传递的是字符串，更严格来说是“JSON 格式的字符串”。

### JSON 对象

`JSON.stringify()`里面的`JSON`是什么？

这是`JavaScript`中的内置对象，是实际存在的，但不同于之前所说的`JSON`。

这是最容易让人混乱的地方了，如果我们的`JavaScript`中有这么一个对象，由于符合`JSON`格式，有些人称为”JSON 对象“，但这只是一个普通的对象。

```javascript
const jsonObj = {
    "name": "ltaoo"
};
```

#### JSON.stringify

上面提到，实际传递的是字符串，所以不使用`JSON.stringify()`而是自己声明一个也是可以的，比如：

```javascript
fetch('https://easy-mock.com/mock/5a1d30028e6ddb24964c2d91/business/api/login', {
    method: 'POST',
    body: '{ "name": "ltaoo" }',
})
    .then((res) => res.json())
    .then((data) => {
        console.log(data);
    });
```

当然这样很蠢。。。而且`stringify()`方法还有一些额外的”特效“：

```javascript
const body = {
    name: 'ltaoo',
    skills: undefined,
};
console.log(JSON.stringify(body));
```

你认为结果会是什么？

1. 报错，因为`undefined`不是合法的值
2. 成功，忽略不合法的值
3. 成功，将`undefined`转换为`null`
4. 其他

答案是2，最终打印的值是`{"name":"ltaoo"}`，如果换个格式，结果是一样的吗？

```javascript
const body = {
    name: 'ltaoo',
    skills: [undefined],
};
console.log(JSON.stringify(body));
// 结果是 {"name":"ltaoo"} 还是 {"name":"ltaoo", "skills": []} ?
```

答案揭晓，是`{"name":"ltaoo","skills":[null]}`。

我们都知道，`JavaScript`的数组不同于其他语言的数组，元素可以是不同类型的如`[1, true, 'name']`是合法的，使用`JSON.stringify()`会得到什么结果呢？

#### JSON.parse

> JSON.parse() 方法用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。

在实际的数据传递中并不常用，因为请求库（axios、isomorphic 等）帮我们做了处理。常见的使用场景是将自己生成的JSON 字符串转换回来，如`localStorage`存储信息。

那可以用来判断某个字符串是否符合`JSON`格式呢？

可以，而且经常会遇到解析失败导致的报错，类似于：

```
Uncaught SyntaxError: Unexpected token x in JSON at position x
```

如果接口请求得到的结果不符合`JSON`格式也会有这种报错。

## 总结

`JSON`是日常开发中最常使用的，但仅限于“会用”，实际上`JSON`的用途已经不局限在“数据交换”，`NoSQL`、配置文件也有`JSON`的身影，深入了解是有必要的，毕竟看起来这么“简单”。




