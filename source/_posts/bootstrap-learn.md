---
title: bootstrap 的正确打开方式
categories: CSS
tags:
- sass
- bootstrap
---
这篇笔记的来源是和一个正在学习前端的人聊天衍生出来的。他说他并没有将太多时间花在 CSS 的学习上，理由是有 bootstrap 就够了。

但是我个人看法却很不一样，也不能说对错，只是一个看法而已。

首先要了解 bootstrap 适合在什么场景使用。
> Bootstrap 让前端开发更快速、简单。所有开发者都能快速上手、所有设备都可以适配、所有项目都适用。


这是 bootstrap 的官网对 bootstrap 的描述。“所有开发者”意味着包括非前端，既然非前端都能够快速上手，那前端开发者使用 bootstrap 的优势在哪里？答案是 bootstrap 并不是给前端开发者使用的，或者说，打开方式错误。

<!--more-->
## bootstrap 的适用场景


bootstrap 的适用场景是项目没有 UI 设计的情况，没有 UI 设计，就意味着需要开发者自己去写页面样式，但是大部分的后端开发并不**擅长** CSS，即使他们都说 CSS 只是一门标记语言。后端开发使用 bootstrap 可以快速简单搭建出一个“能看”的页面，且相比自己写要好看太多，所以 bootstrap 几乎成为项目标配。当然，还有很多原因。。。响应式设计、完善的组件等等都是初级前端无法做到，借助 bootstrap 却很容易就能做到而且又是项目需要的功能。


但是，如果项目中有 UI 设计师，不用 bootstrap 最大的理由是，UI 设计师不懂 bootstrap，不会按照 bootstrap 的样式来设计。


- 有能力的设计师都会有自己的设计风格，所以不会按照 bootstrap 的样式来设计
- 没有能力的设计师也不会按照 bootstrap 的样式来设计，因为会觉得自己的设计完爆 bootstrap（其实并不）。而且没有能力的设计往往不会考虑响应式设计、不同状态的样式、栅格系统下的排版等等。而设计稿没有这些，那 bootstrap 的作用就几乎没有了，能想到的就是 bootstrap 的样式重置，其实也是`normalize.css`的功能而不是 bootstrap 的了。


总之，使用 bootstrap 前应该考虑清楚是否真的需要，或者只是需要 bootstrap 的一部分功能？


## 正确打开方式


说了这么多，其实是想说，前端开发者在使用 bootstrap 时，绝不是学习类名、记忆类名。而是学习 bootstrap 的代码组织方式，或者说，编写样式的方式。 


什么方式呢，**往大了说，是组件化的开发方式，往小说，是预处理器的使用方式**。所以个人看法，前端开发者在使用 bootstrap 时，应该使用其源码，即 less 文件或者其他版本的预处理器源码。既在项目中使用了 bootstrap，又能够学习到 bootstrap 的核心知识。


## 核心


对于前端开发者而言，bootstrap 的核心应该是 mixins。理由是
- bootstrap 的前身 preboot 就只有变量及 mixins，是一个针对前端开发者的项目
- 相同的 mixins 可以根据项目快速编写不同的样式


举个栗子，按钮基本是每一个项目必备的组件/控件，而且往往会重置原生控件的样式，所以我们会这样写样式：
```css
.button {
      background-color: #fff;
      width: 100px;
      color: red;
      border: 1px solid red;
      border-radius: 15px;
      line-height: 26px;
      cursor: pointer;
    }
    .button:hover {
      background-color: red;
      color: #fff;
    }
    .button--active {
      color: #fff;
      background-color: red;
    }
```
```html
<button class="button button--active">click it</button>
  <button class="button">reset</button>
```
上面的样式会在页面显示这样的按钮

![](3d08bce4-0e5c-4b41-bcb8-0a61117d33b7.png)



OK，在这个项目要求是这样，可能在另一个项目会要求颜色不同，border-radius 的值不同等等，不知道别人是怎么做的，我是重新写一样的代码。但是如果使用预处理器，就可以这样写 mixin:
```scss
// mixins/_button.scss
@mixin button($color, $background,  $border-color, $width, $line-height, $radius) {
    width: $width;

    line-height: $line-height;

    color: $color;

    background-color: $background;
    border: 1px solid $border-color;

    border-radius: $radius;

    corsor: pointer;

    &:hover {
         background-color: red;

         color: #fff;

     }

}
```
然后这样使用
```scss
@import 'mixins/button';
.button {
    @include button(red, #fff, red, 100px, 26px, 15px);

}
```
可以根据项目样式传入不同的值。而且 mixin 的用处远远不止这些。在后处理器出现前，如果用到了需要添加浏览器厂商前缀的属性，一个一个写就很麻烦，也可以借助 mixin 来简化这些工作。




所以，一个前端开发，应该要有自己的 mixins。说这么多，其实是想说，可以直接将 bootstrap 的 mixin 作为自己的初始 mixin 库，开发项目的过程中去增加、完善这个库。所以先查看下每一个 mixin 是如何编写的，解决了什么问题，是否适合自己，而且 bootstrap 作为一个开源项目，代码质量毋庸置疑，所以也可以将 bootstrap 作为你的第一个阅读源代码的开源项目。


##  学习方式


正确的学习方式当然是一个一个文件阅读，scss 版的源码（其他版本当然也有），有一个`bootstrap.scss`文件，属于入口文件，可以从注释中了解每一个文件的作用，并且和 bootstrap 官网是有对照关系，跟着官网，很容易就能从源码中学习到 bootstrap 的核心内容。

学习完 bootstrap 的 mixins，或许可以再看看开源的 mixin 库。

- [Preboot 2](https://github.com/mdo/preboot)
- [SassMagic](https://github.com/W3cplus/SassMagic)