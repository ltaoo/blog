---
title: React 源码中的位运算
date: 2020/02/08
categories:
    - React
tags:
    - JavaScript
---

看 `react` 源码过程中，发现这样的代码

```js
const NoContext = /*                    */ 0b000000;
const BatchedContext = /*               */ 0b000001;
const EventContext = /*                 */ 0b000010;
const DiscreteEventContext = /*         */ 0b000100;
const LegacyUnbatchedContext = /*       */ 0b001000;
const RenderContext = /*                */ 0b010000;
const CommitContext = /*                */ 0b100000;

let executionContext = NoContext;
executionContext &= ~BatchedContext;
executionContext |= LegacyUnbatchedContext;

if ((executionContext & (RenderContext | CommitContext)) !== NoContext) {
    // We're inside React, so it's fine to read the actual time.
    return msToExpirationTime(now());
}
```

`~`、`|` 和 `&` 都是「位运算操作符」，平常写代码很少会用到，导致完全不理解这些运算符有什么用，`react` 源码中这些代码到底是什么意思呢？
<!--more-->

## ~ 运算符
首先来看第一个运算符 `~`，它是所谓的「否定号」或者说「逻辑否」。
> 与之相对的就是「逻辑与」`&` 和「逻辑或」 `|`。

它的计算过程很简单，就是「二进制取反」，即写出数字的二进制表示，每位都和之前相反，1 变成 0，0 变成 1。

```js
~1 // ~000001 === 111110
```

它实质上是对数字求负，然后减1。
所以 `~1 === -2`、`~0 === -1` 这个很好理解，并且我们能得出 -2 的二进制表示正是 111110

```js
const num = -2;
// 1、先写出 2 的二进制表示
// 000010
// 2、取反
// 111101
// 3、加1
// 111110
```

## & 运算符
这个相比 `~` 运算符在计算过程上很好理解，但没有 `~` 运算符好计算。

```js
25 & 6; // 0
25 & 7; // 1
25 & 8; // 8
```

上面代码的计算过程是这样的，把数字写成二进制，上下比较，如果都是 1 那么结果的相同位也是 1 否则就是 0。

```js
0000 0000 0000 0000 0000 0000 0001 1001  // 25
0000 0000 0000 0000 0000 0000 0000 0110  // 6
// ---------------------------------------
0000 0000 0000 0000 0000 0000 0000 0000  // 0

0000 0000 0000 0000 0000 0000 0001 1001  // 25
0000 0000 0000 0000 0000 0000 0000 0111  // 7
// ---------------------------------------
0000 0000 0000 0000 0000 0000 0000 0001  // 1

0000 0000 0000 0000 0000 0000 0001 1001  // 25
0000 0000 0000 0000 0000 0000 0000 1000  // 8
// ---------------------------------------
0000 0000 0000 0000 0000 0000 0000 1000  // 8
```

## `|` 运算符
与 `&` 刚好相反，二进制表示时，上下其中任意一个是 1 那么结果的相同位上也是 1。所以之前的例子

```js
0000 0000 0000 0000 0000 0000 0001 1001  // 25
0000 0000 0000 0000 0000 0000 0000 0110  // 6
// ---------------------------------------
0000 0000 0000 0000 0000 0000 0001 1111  // 31

0000 0000 0000 0000 0000 0000 0001 1001  // 25
0000 0000 0000 0000 0000 0000 0000 0111  // 7
// ---------------------------------------
0000 0000 0000 0000 0000 0000 0001 1111  // 31

0000 0000 0000 0000 0000 0000 0001 1001  // 25
0000 0000 0000 0000 0000 0000 0000 1000  // 8
// ---------------------------------------
0000 0000 0000 0000 0000 0000 0001 1001  // 25
```
和 `&` 一样无法直观地看出结果。

## react 中的位运算
在了解三个运算符后，我们回过头看看 `react` 源码，它有多种用法

```js
let executionContext = NoContext;

executionContext &= ~BatchedContext;
executionContext &= BatchedContext;
executionContext |= BatchedContext;
(executionContext & (RenderContext | CommitContext) === NoContext;
```

这里每个常量的值都是有目的的，分别为 0、1、2、4、8、16、32，而不是像枚举那样 0、1、2、3，就是为了配合位运算。

```js
const NoContext = /*                    */ 0b000000; // 0
const BatchedContext = /*               */ 0b000001; // 1
const EventContext = /*                 */ 0b000010; // 2
const DiscreteEventContext = /*         */ 0b000100; // 4
const LegacyUnbatchedContext = /*       */ 0b001000; // 8
const RenderContext = /*                */ 0b010000; // 16
const CommitContext = /*                */ 0b100000; // 32
```

那么这些特定的数字是怎么配合位运算做到「有含义」的呢？
**其实就是 `executionContext` 所对应二进制中 1 的位置，记录了它所「包含」的 `xxxContext`**，举例来说 `000011` 表示 `BatchedContext` 和 `EventContext`；`110001` 表示 `CommitContext` 和 `RenderContext` 和 `BatchedContext`；

### 表示状态累加的 `|`

首先来看看 `|` 运算符，由于每个常量二进制的 1 都在不同位置，两个任意常量经过 `|` 运算符后，其实就是增加了 1 的位置

```js
NoContext | BatchedContext;// 000001
BatchedContext | RenderContext; // 010001;
```

所以可以理解 `|` 运算符就是「记录新增 Context」或者说「状态的累加」。如果用传统的写法，可能是这样的

```js
const BatchedContext = 'BatchedContext';
const EventContext = 'EventContext';

const executionContext = [];
executionContext.push(EventContext); // 等同 react 源码中 executionContext |= EventContext;
```

### 判断状态是否存在的 `&`

然后是 `&` 运算符，同样在特定数字下它能表达特殊的含义。首先由于每个常量的 1 都在不同位置，所以两个常量直接 `&` 操作后结果肯定是 000000，但如果是经过 `|` 运算后的 `executionContext` 和常量进行 `&` 运算，结果就有意思了

```js
let executionContext = NoContext;
executionContext |= BatchedContext;

executionContext &= BatchedContext; // 000001 & 000001 === 000001

executionContext |= EventContext; // 000001 | 000010 === 000011
executionContext &= BatchedContext; // 000011 & 000001 === 000001
executionContext &= EventContext; // 000001 & 000010 === 000000
```

可以发现它有「判断是否存在」的含义，但由于使用了 `&=` 所以更确切的含义是「如果存在就覆盖」。

### 移除状态的 `~`

这个符号目前看到的场景是和 `&` 一起使用

```js
let executionContext = NoContext;
executionContext &= ~BatchedContext;
```

前面说过 `&=` 是判断存在就覆盖的含义，但是 `BatchedContext` 多了个 `~` 符号，含义肯定有所改变，那这段代码究竟是什么含义呢？

```js
executionContext = 000000 & ~000001;
executionContext = 000000 & 111110;

executionContext = 000000;
```

换个用例

```js
executionContext = NoContext;
executionContext |= BatchedContext;
executionContext |= EventContext;

executionContext &= ~BatchedContext;
```

等同于

```js
executionContext = 000011 & ~000001;
executionContext = 000011 & 111110;
executionContext = 000010; // 2
```

它表达的是「移除指定状态」，如果用传统的写法

```js
const BatchedContext = 'BatchedContext';
const EventContext = 'EventContext';

const executionContext = [];
executionContext.push(EventContext); // 等同 react 源码中 executionContext |= EventContext;

executionContext.filter((context) => context !== BatchedContext); // 等同 executionContext &= ~BatchedContext
```

### 位运算的优点
为什么 `react` 使用这么难理解的方式去实现这些逻辑呢，大概是因为位运算的性能好很多吧，如果使用数组来保存状态，无论是要占用堆内存，还是在判断时还需要在内存中查询，任何方面，相比位运算都没有一点优势，那么采用位运算看来是必然的选择了。

## 参考
- [ECMAScript 位运算符](https://www.w3school.com.cn/js/pro_js_operators_bitwise.asp)
- [React的第一次渲染过程浅析
javascriptreact.js](https://segmentfault.com/a/1190000020532672?utm_source=tag-newest)
