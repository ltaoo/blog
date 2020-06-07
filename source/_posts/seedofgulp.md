---
title: 基于 gulp 的前端开发环境
categories: 工具
tags:
- gulp
- sass
- es6
---

在平常学习的时候，会需要写一些 demo，想要用到 gulp 来方便开发、调试，但是每次都复制一份`gulpfile.js`很麻烦，所以先写好一个符合自己技术栈的脚手架，放在 github 上，借助 vue-cli 来简化初始化项目的操作。

> “Gulp.js 是一个自动化构建工具，开发者可以使用它在项目开发过程中自动执行常见任务。Gulp.js 是基于 Node.js 构建的，利用 Node.js 流的威力，你可以快速构建项目并减少频繁的 IO 操作。Gulp.js 源文件和你用来定义任务的 Gulp 文件都是通过 JavaScript（或者 CoffeeScript ）源码来实现的。

<!--more-->

## 使用方式

由于是借助 vue-cli 这个脚手架工具来下载放在 github 上的脚手架，所以需要先全局安装`vue-cli`
```
npm install -g vue-cli
```

然后使用`vue init`命令创建项目
```
vue init ltaoo/demofed <projectName>
```

> `ltaoo/demofed`为我的脚手架在 github 上的仓库名。

可以看到会在当前目录创建`projectName`文件夹，进入文件夹安装项目需要的依赖
```
npm install
```

安装好依赖后，就可以开始使用了

```
npm start
```
等待一会可以看到浏览器打开`127.0.0.1:3000`页面，表示成功运行。

## gulpfile.babel.js

项目的核心文件，通过该文件调用我们的所有的功能，包括编译 sass、编译 es6 到 es5，将 es5 打包合并等功能。

gulp 应该是有两个核心的概念，一个是“流”，一个是“任务”。

使用`gulp.src()`可以打开一个文件，文件通过该方法变成了“水流”，水流在`pipe`里面流动，`pipe`上的“插件”可以对“水流”进行改造，比如改变颜色等，最后“水流”到了`dest()`源头后，又变回了文件。

### sass 任务
该任务会将`.scss`后缀的文件转换为`.css`后缀。
```javascript
gulp.task('sass', ()=>{
  return gulp.src(source.css)
    .pipe(sourcemaps.init())
    .pipe(sass({outputStyle: 'compressed'}).on('error', sass.logError))
    .pipe(sourcemaps.write('.'))
    .pipe(gulp.dest(output.css))
    .pipe(browserSync.stream());
});
```

### build-js 任务
该任务将`js`文件转换为能够在浏览器中执行的`js`文件。
```javascript
gulp.task('build-js', () =>{
  return browserify(source.main, {
    debug: true
  }).transform(babel)
    .bundle()
    .pipe(vinyl(output.main))
    .pipe(buffer())
    .pipe(sourcemaps.init({loadMaps: true}))
    .pipe(sourcemaps.write('./'))
    // 输出的就是编译好的
    .pipe(gulp.dest(output.script))
    .pipe(browserSync.stream());
});
```

### dev 任务
开启静态服务器，并且在指定文件改变时刷新页面。
```javascript
gulp.task('dev', ['sass', 'build-js'], () =>{
  browserSync.init({
    server: {
      baseDir: './'
    }
   //或者代理一个服务器
   //proxy: "127.0.0.1:8080" 替代server键值对，不是baseDir!!!!。
  });
  gulp.watch(source.css, ['sass']);
  gulp.watch(source.scripts, ['build-js']);
  gulp.watch(source.index, () =>{
    browserSync.reload();
  });
})
```

### default 任务
前面的任务都可以在全局安装`gulp`后，使用`gulp <taskName>`的形式来调用单个任务。如果不指定任务，即`gulp`命令将会执行`default`任务。
```javascript
gulp.task('default', ['dev']);
```
其实这个任务的目的是为了少打`dev`，使用`gulp dev`可以实现相同的效果。

## 难点
这里面困扰我很久的地方是将 es5 代码使用`browserify`打包成浏览器可用的代码，并且可调试。

打包很简单，有`browserify`和`gulp-browserify`两个插件，后者比前者要简单，就像前面说的可以对“水流”进行改造。但是不知道为什么，无法生成正确的`sourcemaps`。网上的文章大部分都是使用`browserify`，该插件不同于`gulp-browserify`，是可以独立使用的，意味着它是自己接收文件，自己处理文件，自己输出文件。

但是又因为我们要将`browserify`和`gulp`搭配使用，所以就需要将`browserify`产生的“流”变成`gulp`可用的流（两个虽然都是流，但是有区别，就好像一个是水流，一个是岩浆流...），所以用到了`vinyl-source-stream`和`vinyl-buffer`来完成这一目的。

所以`build-js`任务可以看到和其他处理文件的任务不同的是没有使用`gulp.src()`。

## TODO

之所以借助`vue-cli`来从 github 下载脚手架，因为自己实现一个类似功能的工具比较麻烦，所以在没事情做的时候准备参考 vue-cli 尝试实现一个自己的 npm 包。

- 自己实现 cli
- 增加 eshint
- 增加 单元测试
- 美化首页？




