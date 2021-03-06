# 也聊聊 tapable

tapable 是什么，以及很多人写博客讲过了，但我认为讲得不够好，都是在作者本人完全「了解」了 tapable 后写出的博客。

这里可能会有疑问，不了解怎么写博客呢？

话虽如此，但在「了解」的前提下写的博客，很难顾及完全不了解 tapable 的人，看的时候就是一头雾水，作者写博客只是想表达「我懂了」，而不是「我写了博客希望你能看懂」。

那要怎么写呢？

很简单，其实就是写心路历程，你是怎么从「不了解 tapable」到「了解」，你是如何学习，如何思考的，读者如果能够代入你当时的状态，那看完后也肯定学到了。

所以，我就希望写这么一篇博客，讲述我了解 tapable 的过程。


## hook?

现在写前端的，应该没有不知道`hook`这个概念了吧，无论是`vue`还是`react`，都有「生命周期函数」的概念，在`vue`中更直接的称为生命周期钩子。

OK，那其实就简单了，`hook`就是「在特定的阶段做特定的事情」，如`componentDidMount`就是在`DOM`插入页面后会触发。

那么`tapable`也是一样的吗？

一样，又不一样。

## Plugin

绝大部分想了解 tapable 的人都是因为 webpack 中的 plugin。并且在 plugin 相关的介绍中是这么说的：


>


这就导致了会有一个先入为主的概念，`tapable`实现了一套插件机制。但这是错误的观点吧，因为前面提到 tapable 是 hook，hook 是什么？简单来说就是观察者默认，就是 EventEmitter。

和插件完全不搭嘎啊。

事实确实如此，plugin 是 webpack 自己的一套东西，plugin 是基于 tapable 实现的。


但还是不对吧，有一些经验的人都知道，plugin 的结果是类似于「流」一样的东西，将输出的结果可以经过多个`plugin`去处理，而 hook 貌似没有这种特性。

这就是 tapable 的 hook 特殊之处了。

## SyncHook

这篇博客就不一一介绍每个 Hook 对象是什么，怎么用了，因为其他博客都是介绍这些内容的。所以我们来讲一讲如何实现自己的 plugin。


首先给出一个实例，现在要处理一个东西，什么呢，todo list。现在假设我们一个非常定制化的 todo list，它可以在用户输入 todo 后，根据用户安装的插件对 todo 内容进行处理，举个例子：

```
学习tapable
```


这么一段内容，现在有 xx plugin，它的作用是自动在英文两边添加空格。

