---
title: react + webpack 配置
categories: 工具
tags:
- webpack
- react
---

官方脚手架能够满足基本需求，但通过自己配置，可以学习到 webpack 的用法，毕竟 webpack 这么热门。

这篇笔记最后能够配置出支持 es2015、jsx 和 hot-reload 的 react 开发环境，是的，只是开发环境，没有打包的配置。

<!--more-->

## 初始化
从一个基本功能的 webpack 配置，一步步升级到适合 react 的配置。
```bash
npm install --save-dev style-loader css-loader webpack
```
先下载 webpack 和 两个样式“加载器”，然后配置 webpack.config.js 文件。
```javascript
// webpack.config.js
var path = require('path')// 只用到了 join 函数用来拼接路径

module.exports = {
  entry: './src/index.js',// 项目入口文件

  output: {
    path: path.join(__dirname, './dist'), // 打包文件输出路径
    filename: 'bundle.js'  // 打包文件名
  },

  module: {
    loaders: [
      {test: /\.css$/, loader: 'style!css'} // 样式加载器
    ]
  }
}
```
```json
// package.json
"devDependencies": {
    "webpack": "^1.13.2",
    "css-loader": "^0.24.0",
    "style-loader": "^0.13.1"
}
```


### 测试
然后写一些代码测试配置文件是否有效。
```javascript
// ./src/index.js
var app = require('./components/App.js')
require('./styles/main.css')

alert(app.name)
```
```javascript
// ./src/components/App.js
module.exports = {
  name: 'ltaoo'
}
```
```css
<!-- ./src/styles/main.css -->
body {
  background: #eee;
}
```
由于 webpack 是打包工具，所以肯定是支持JavaScript打包的，`require`js 文件并没有问题，但是这里还`require`了`main.css`，这就是 webpack 的一个特色，所有的资源都视为模块，都可以`require`。但是为了支持这种特殊的模块，需要额外下载依赖，就是我们之前现在的`style-loader`和`css-loader`。

使用`webpack`命令编译文件，可以得到`./dist/bundle.js`。在`index.html`引入该文件，浏览器访问`index.html`，背景色为灰色、弹出`alert`框，表示成功。

> `webpack`命令有两种使用方式，一是全局安装后可以直接在命令行使用该命令。二是只在项目中安装，就需要使用`npm scripts`来使用了。
```json
"scripts": {
    "build": "node ./node_modules/webpack/bin/webpack.js"
}
```
使用`npm run build`等同于全局安装下的`webpack`。不过全局安装更方便。

## 支持 es6
```bash
npm install --save-dev babel-core babel-loader babel-preset-es2015
```
虽然 webpack 支持`require`js文件，但es6 语法就无法识别了，所以我们还要指定一个`loader`来处理我们的 js 文件，该 loader 能够将 es6 语法转换为 es5。
```json
"devDependencies": {
    "webpack": "^1.13.2",
    "babel-core": "^6.13.2",
    "babel-loader": "^6.2.5",
    "babel-preset-es2015": "^6.14.0",
    "css-loader": "^0.24.0",
    "style-loader": "^0.13.1"
}
```
```javascript
// webpack.config.js
loaders: [
  {
    test: /\.js$/, 
    loader: 'babel',
    query: {
      presets: [
        require.resolve('babel-preset-es2015')
      ],
    }
  },
  {test: /\.css$/, loader: 'style!css'}
]
```

### 测试
修改之前的 js 代码为 es6 语法。
```javascript
// index.js
import app from './components/App.js'
import './styles/main.css'

let name = app.name

var alertName = (name)=>{
  alert(name)
}

alertName(name)
```
```javascript
// ./components/App.js
export default {
  name: 'ltaoo'
}
```

再次使用`webpack`命令编译，也能够成功弹出 alert 框。

## jsx 语法支持
```bash
npm install --save-dev babel-preset-react
```
由于 react 有自己特殊的语法，也需要特定的 loader 来识别。类似这样
```javascript
// index.js
import React from 'react'
import ReactDOM from 'react-dom'
import App from './components/App.js'
import './styles/main.css'

ReactDOM.render(
  <App />,
  document.getElementById('app')
)
```
```javascript
// ./components/App.js
import React, { Component } from 'react' 

export default class App extends Component {
  render() {
    return (
      <h2>Hello React</h2>
    )
  }
}
```
可以看到，在`javascript`中写`html`标签（我知道本质上并不是），使用`webpack`命令会提示`<`语法错误，因为这在 JavaScript 是比较符号？

所以需要使用`babel-preset-react`来进行处理。
```javascript
// webpack.config.js
loaders: [
  {
    test: /\.js$/, 
    loader: 'babel',
    query: {
      presets: [
        require.resolve('babel-preset-es2015'),
        require.resolve('babel-preset-react')
      ],
    }
  },
  {test: /\.css$/, loader: 'style!css'}
]
```
```json
// package.json
"devDependencies": {
    "webpack": "^1.13.2",
    "babel-core": "^6.13.2",
    "babel-loader": "^6.2.5",
    "babel-preset-es2015": "^6.14.0",
    "babel-preset-react": "^6.11.1",
    "css-loader": "^0.24.0",
    "style-loader": "^0.13.1",
    "react": "^15.3.1",
    "react-dom": "^15.3.1"
}
```

依赖都安装好后，再次`webpack`编译。没有报错，页面也正确显示`Hello React`！

## sass
预处理器可以让开发更简单，所以也加上。
```bash
npm install --save-dev sass-loader node-sass
```

### 测试
```scss
$gray: #eee;

body {
  background: $gray;
  h2 {
    color: orange;
  }
}
```

在`index.js`中引入该文件。编译。页面显示出我们希望的样式。

> 配置该 loader 时遇到很奇怪的问题，明明安装了 node-sass，却提示该模块下缺少一个 文件。而且由于使用`cnpm`命令，模块真实地址在`.npminstall`文件夹下，但是删除该文件夹下的`node-sass`文件夹又提示没权限，直接将`node_modules`删除，使用`npm`安装依赖。


## webpack-dev-server
肯定要配置一个静态服务器方便开发，与 webpack 配套的是第一选择。
```bash
npm install --save-dev webpack-dev-server
```
和`webpack`一样，也有两种使用方式，推荐全局安装。
在项目根目录使用`webpack-dev-server`命令开启静态服务器，等待编译完成生成`bundle.js`后即可在`127.0.0.1:8080`访问。
```bash
webpack-dev-server --color
```


### Automatic Refresh

webpack-dev-server 支持自动刷新，和 gulp-browsersync 一样。有三种实现方式。

- Iframe mode

这一种是直接通过访问`127.0.0.1:8080/webpack-dev-server`来实现。

- Inline mode

只要在`webpack-dev-server`命令加上`--inline`和`--content-base`参数即可。

```json
// package.json
"start": "webpack-dev-server --inline --content-base ."
```

> `--content-base`参数后面写路径，该路径既是静态服务器的根路径，也是 bundle.js 生成的路径（非打包的 bundle.js，而是开发时的“虚拟的”bundle.js），在`index.html`文件中引入的`bundle.js`必须是当前路径下的，和 webpack.config.js 中 output 配置的没关系。

- Hot Module Replacement

上面的方式是监听文件改变就重新编译、刷新浏览器，所以即使是修改一点样式，都要等很久。开发体验不好，所以使用`hot-reload`来替代`inline mode`。

文档中提到，需要在`webpack.config.js`中添加`HotModuleReplacementPlugin()`插件，才可使用`--hot`参数。

```javascript
// webpack.config.js 省略没改变的
var path = require('path')
var webpack = require('webpack')

module.exports = {
  entry: "./src/index.js",

  plugin: [
    new webpack.HotModuleReplacementPlugin()
  ]
}
```
```bash
webpack-dev-server --inline --hot
```

不过很多网上的文章使用了这种形式的`entry`:
```javascript
entry: [
    'webpack-dev-server/client?http://localhost:8080',
    'webpack/hot/only-dev-server',
    './entry.js'
]
```
并且添加`devServer`可以同样实现`hot-reload`
```javascript
devServer: {
    hot: true,
    contentBase: './'
}
```
然后就不用在命令行添加`--inline --hot`参数了。原因是，`--inline`参数会自动插入`webpack-dev-server/client?http://localhost:8080`，`--hot`参数同理。


**实际上开发 react 时还是刷新整页！！！**，需要使用`react-hot-loader`来实现真正的 热替换。

### react-hot-loader
```bash
npm install --save-dev react-hot-loader
```

修改配置
```javascript
// webpack.config.js
{
    test: /\.js?$/, 
    loaders: ['react-hot', 'babel?presets[]=es2015&presets[]=react'],
    include: path.join(__dirname, 'src')
},
```
由于需要多个加载器，所以不能写成之前的格式，但原理是一样的。这里必须要有`include`字段，是我们的 js 文件存放的路径。



## 优化
功能是完善了，不过可以添加一些配置使开发更加方便。
### resolve.extensions
配置后在加载模块时可以省略后缀名
```javascipt
resolve: {
    extensions: ['', '.js', '.scss']
},
```

### devtool
```javascript
devtool: 'source-map'
```
方便调试。

不过查看样式并没有显示对应的文件行数，而是显示`<style>..</style>`，这是因为 webpack 为了实现样式的即时更新，将我们在 css 或 scss 文件中的样式使用`style-loader`提取出来并插入`index.html`的`head`标签内。
> "css" loader resolves paths in CSS and adds assets as dependencies."style" loader turns CSS into JS modules that inject `<style>` tags.In production, we use a plugin to extract that CSS to a file, but in development "style" loader enables hot editing of CSS.

```javascript
{
    test: /\.scss$/, 
    loaders: ['style', 'css?sourceMap', 'sass?sourceMap']
}
```
将处理`scss`文件的处理器修改成这样，就能够正确显示出样式所在文件与行数了。

更多配置项参考官方文档 - [configuration](https://webpack.github.io/docs/configuration.html)

## todo

- 分离公共代码（第三方类库引用？）与业务代码
- 生成打包 css 文件

## github
[react-template](https://github.com/ltaoo/react-template/blob/master/template/webpack.config.js)是实例代码。

## 参考
- [webpack bable-loader升级无法编译jsx  es6 ](https://segmentfault.com/a/1190000004026252)

- [Beginner’s guide to Webpack](https://medium.com/`@`dabit3/beginner-s-guide-to-webpack-b1f1a3638460#.ht65p3zfl)

- [Webpack’s HMR & React-Hot-Loader — The Missing Manual](https://medium.com/`@`rajaraodv/webpacks-hmr-react-hot-loader-the-missing-manual-232336dc0d96#.iaxez67r2)

- [WEBPACK DEV SERVER](http://webpack.github.io/docs/webpack-dev-server.html#hot-module-replacement)

- [React Hot Loader](http://gaearon.github.io/react-hot-loader/)
