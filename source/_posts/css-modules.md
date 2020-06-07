---
title: CSS 组件化/模块化？
categories: CSS
tags:
- bootstrap
- sass
---

现在写样式，基本都会用到预处理器，将页面样式按照模块/组件进行划分，比如`header.scss`、`footer.scss`，或者像 bootstrap 每一个组件单独写一个文件，比如`forms.less`、`navs.less`、`navbar.less`等，在一个主文件`bootstrap.less`中将所有的文件引入，最后编译。这应该是大多数人对 CSS 组件化开发的理解，一开始我也是这么想，直到在复习 sass 时看到有“默认变量”这样一个东西存在。

<!--more-->

## 默认变量

> sass 的默认变量一般是用来设置默认值，然后根据需求来覆盖的，覆盖的方式也很简单，只需要在默认变量**之前**重新声明下变量即可。

```sass
$baseLineHeight: 2;
$baseLineHeight: 1.5!default;

body{
    line-height: $baseLineHeight;
}

//css style
body{
    line-height: 2;
}
```


> 默认变量的价值在组件化开发的时候会非常有用。



## 使用默认变量进行开发
然后搜到这篇教程[sass揭秘之变量](http://www.w3cplus.com/preprocessor/sass-basic-variable.html).

这里有一个`_imgstyle.scss`模块负责图片相关样式，比如给图片添加内补

```sass
//_imgstyle.scss
//vari
$imgPadding: 5px !default;
//mixin
@mixin img-padding($imgPadding: $imgPadding) {
    padding: $imgPadding;
}
//style
.img-padding {
    @include img-padding;
}
```
然后我们在主文件中引用这个模块
```scss
// main.scss
@import 'imgstyle';
```

OK，进行编译后，就可以生成css文件了
```css
// compile file
.img-padding {
    padding: 5px;
}
```
至此我们完成了组件化开发，在`index.html`文件中 link 编译后的 css 文件，给需要 padding 的图片标签挂载`img-padding`类就可以了。如果我们需要修改 padding 值，一般的做法当然是找到`_imgstyle.scss`进行修改，毕竟这是组件化开发的一个优势，能够更快定位需要的地方。

或者在`main.scss`文件中覆盖一次：
```css
// main.scss
.img-padding {
  padding: 6px;
}
```

上面的方法简单但是会有代码的冗余（重新声明覆盖原有样式），并且不“组件化”了，之后如果还需要修改，可能由另一个开发人员修改，这个开发人员到`_imgstyle.scss`文件中找到了并修改，但是并没有作用，因为起作用的样式在`main.scss`中。

教程中提到了可以使用默认变量来实现，并且没有代码的冗余：

```scss
// main.scss
@imgPadding: 10px;

@import 'imgstyle';
```

实际上是这样的：

```scss
// main.scss
@imgPadding: 6px;
//vari
$imgPadding: 5px !default;
//mixin
@mixin img-padding($imgPadding: $imgPadding) {
    padding: $imgPadding;
}
//style
.img-padding {
    @include img-padding;
}
```

根据默认变量的定义，生效的是在默认变量声明前的变量声明，即6px。

如果没有默认变量的存在，就只能到模块内修改变量值。

## 默认变量实现模块化开发

有个问题就是，还能够用回5px 的定义吗？如果不能，那我在`_imgstyle.scss`文件中修改更好一些，避免了`@imgPadding`变量同时存在`main.scss`和`_imgstyle.scss`文件中，不方便后续修改。下面是教程作者的原话：
> ...直接在`_imgstyle.scss`文件中，修改`$imgPadding`为6px即可。当然如果你要的是每个项目使用这个样式的时候都拷贝一份这个，然后打开把变量修改成需要的值，那么我只好承认我脑子进水了，不仅脑子浸水，还得吐血了。

“每个项目使用这个样式的时候都拷贝一份这个，然后打开把变量修改成需要的值（会很麻烦）”，这是作者使用默认变量的目的。

重点在于**每个项目**，即使用默认变量方式写成的组件，是可以被多个项目使用且可以很方便修改组件中的值。比如：

项目A
```scss
//main.scss
@imgPadding: 10px;
@import 'imgstyle';
```
项目B
```scss
//main.scss
@imgPadding: 15px;
@import 'imgstyle'
```

预先写好样式，在不同项目中重新声明变量再引入组件可以得到不同的样式。这样可以使用别人写好的组件，只要文档中说明什么值是可以覆盖的即可。和python或者其他可以引入第三方模块的形式很类似。想想似乎可以让前端样式开发变得简单、高效。

想到 bootstrap 其实也提供了类似的功能，即“定制”。在 bootstrap 网站的定制页面，可以选择需要的组件，定义一些变量的值，比如颜色等，最后生成编译后的 css 文件。但是如果可以以命令行的形式下载组件、引用、重新赋值似乎要更加高效。

那我们有两种开发方式，这里的比前面提到的要更近一步，前面提到的都是针对单个项目，而这里的组件化开发，称为模块化开发可能更恰当？则需要考虑通用性。可以参考 bootstrap。


## 不适用？

在了解了这种形式的开发后，想到了能否实现一个 Glyphicons 模块，用户下载了该模块后，这么使用

```scss
// main.scss
@icon-style: style1;
@import 'icon';
```
而该模块提供了多种风格的图标，不像现在 bootstrap 的图标只有一种“外貌”，用户可以通过定义`@icon-style`来加载不同的图标（图标含义一样，只是样式不同）。
```scss
//main.scss
@icon-style: style2;
@import 'icon';
```

但是并不能。。。因为字体文件路径的问题。想过使用条件分支，但是感觉很麻烦。


这篇教程是在2013年发布，并且作者已经开发出`tobe`开源项目，即我们这里提到的模块化开发，但是相比 bootstrap，tobe并没有受到太多的关注。

bootstrap 其实一开始也仅仅是一些 less 文件，提供了一些预定义的变量与 mixin，再慢慢发展成现在这种只要简单的复制类名就能实现一个美观的网站，于是所有人都在用。但是，bootstrap 真的适用于所有项目吗？很显然 bootstrap 只适用于没有设计甚至没有前端和设计的项目，比如公司内部系统。所以需要一种能够适应不同项目的开发方案。

## 思考

那怎么样才能对有设计稿的项目提供一种简单的开发方式？

如果是同一设计师对不同项目做的设计稿可能还有规律可循，但不同设计师由于不同的个人风格做出来的设计稿往往差别很大，很难说有通用的解决方案。

但是“懒”作为第一生产力，比如因为懒所以把项目分给多个人做，这就要求方案可以多人协作，因为懒才需要一种简单、快捷的方式，最好也能像 bootstrap 那样挂载一个类就可以实现。终究会有人实现的吧。

所以我想过一种方案，

首先，拿到设计稿，由技术leader或者团队中擅长写样式的人写一个“典型页面”，这个页面会包含一些重复用到的组件，比如表单、导航等等，可以参考 bootstrap 的组件页。

然后，由“工具”对该页面做分析，将页面中的组件一一拆分出来（这就要求在写页面时是以组件化进行划分开发的，参考 bootstrap），同时从组件样式中计算、分析得到全局样式（参考 bootstrap）。

最后，也是最重要的，会生成一个和 bootstrap 类似的网站（下面称为项目网站），即显示组件与相应的`html`代码。

正式开始开发后，项目网站是放在项目中的，部署在远程服务器，开发时先查看该网站（就好像使用 bootstrap 网站一样）了解组件与相应的类名（甚至可以和 bootstrap 相同组件一样，降低记忆成本）后，如果自己的任务中有对应的组件，直接挂载类名即可，如果没有，就自己进行开发，同时！！也按照组件化的开发方式，开发完成后提交至仓库，仓库会再一次对页面进行计算、分析，更新项目网站。这样如果别人要开发相同的组件了，可以直接挂载类名。

很明显，这种方式是可行的，但是问题在于如何对项目进行分析得到组件。这就要求能够实现对现有的 bootstrap 网站分析后得到 bootstrap 的 less 源文件一样。


## 总结

如果是单个项目使用，没有抽象出模块准备复用的情况，默认变量并没有用处。

希望哪天能出现一个大大简化UI开发的工具/方案。


## 参考
- [sass揭秘之变量](http://www.w3cplus.com/preprocessor/sass-basic-variable.html)