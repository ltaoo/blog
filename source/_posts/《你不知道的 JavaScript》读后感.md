---
title: 《你不知道的 JavaScript》读后感
categories: 随笔
tags:
- JavaScript
date: 2017/02/07
---

看到图灵社区的[电子书阅读奖励计划](http://www.ituring.com.cn/article/273646)，很赞的促进阅读的方式，刚好也在看一些书，所以就有了这篇读后感。

内容不长，两三天时间就能看完。看完后还是有一些意外，觉得自己对于 js 已经算了解，但还是有很多细节方面的知识盲点。经过这次阅读对于 JavaScript 的理解肯定是增加了一些，但更多的是在“学习”方面有了更深刻的认知，学习到如何“学习”。
<!--more-->

## 举个栗子
在 p27 提到
> (function foo(){ .. }) 作为函数表达式意味着foo 只能在.. 所代表的位置中被访问，外部作用域则不行。foo 变量名被隐藏在自身中意味着不会非必要地污染外部作用域。

看到的时候就觉得很奇怪，函数名只能在自身作用域访问到，那这个函数名还有什么意义，重点是为什么在外部作用域无法被访问到？

于是写了这么一段代码想要测试`()`是否会产生作用域导致无法在外部访问(虽然明知道不可能...)：
```javascript
;(var a = 1)
console.log(a)
```

但是报错很意外：
```bash
;(var a = 1)
  ^^^
SyntaxError: Unexpected token var
```

到这里就发现，自己对于`()`有什么作用的知识点完全不了解。在了解到`()`可以用来**强制表达式执行**后，又产生了对 IIFE 的一个疑问，如果 IIFE 是“声明一个函数并立即调用”，那下面的代码为什么不行？
```javascript
function () {
	console.log('hello iife')
}()
```

为什么在最外层加上`()`就可以了，既然`()`是用来强制表达式执行，表示内部首先是一个表达式，表达式也可以直接运行的吧。

虽然在继续阅读书的过程中解开了这个疑惑，但我的第一篇博客就是写的关于 IIFE，却发现自己对于 IIFE 这么不了解，实在汗颜。

## 舒适区与一万小时定律

在 p77 页关于`this`的知识点时提到了“舒适区”这个概念，字面意思就是让自己感到舒适的区域。长时间待在舒适区，并不会让自己的能力有多大提升。而“一万小时定律”是指：
> 要成为某个领域的专家，需要10000小时，按比例计算就是：如果每天工作八个小时，一周工作五天，那么成为一个领域的专家至少需要五年。这就是一万小时定律。

如果将两者结合起来思考，在舒适区内的 10000 小时，也能够让人成为专家吗？

## 刻意练习

我觉得是不能的，因为经过实践证明。。。每天涂涂画画自己学会的简笔画，并不会让自己成为顶尖的画师。当然这么说很极端，不过跳出舒适区，刻意去让自己处于“头疼”的境地，绝对比待在舒适区享受成就感要更好。

```javascript
var a, b, c;
```

这样声明变量很常见，自己也会写，但是为什么可以这样声明呢？`,`的作用又是什么？

刻意练习，或者说深究原理吧，才能让自己有更强的竞争力，毕竟要花更多时间在探究“为什么”上~

不止这本书，在 CSS 方面有名的张鑫旭也是将简单的知识点挖掘出意想不到的点，所以保持着这种好奇心，一定能让自己有更多收获。



## 参考 
- [为什么有人工作10年仍不是专家，有人2年就足够卓越了？](https://mp.weixin.qq.com/s?__biz=MzA4ODI5NTE4Mg==&mid=2650242539&idx=1&sn=472dd1ead7ec2df5f40d40fbe7002849&chksm=882f8f8abf58069c0bec785bdbebb3ab60bdac716791d5f8c6d109e61725cddbde48d4e26228&mpshare=1&scene=1&srcid=0131ezqxWypbMLoCcIVeWXyg&from=singlemessage&key=5459fe1fff87676fc6c1bc9ddbb3032292476eaad646a5dd2d748a5b768cfaeca3f220fc3de644eaf47584d349575df310955f6aaa1c323fbc1d7ac7ebec3f0cc84bbac85ea67bd05bcc8d91d8c50b7f&ascene=1&uin=MTQ2MTA2MjA2NA%3D%3D&devicetype=android-22&version=26050431&nettype=WIFI&abtest_cookie=AQABAAgAAQBGhh4AAAA%3D&pass_ticket=0W3IVp6m4EQkjoAHtS30%2BlpZS8NDWLh2Kn1jEq1luFGki7SJMwAZynZyGlkmFi9b&wx_header=1)




