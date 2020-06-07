---
title: nodeJs 应用开发学习
categories: JavaScript
tags:
- nodejs
---


之前使用 nodejs 完成了简陋版的静态服务器，了解了服务器的运行机制：
- 1、创建服务器对象并监听用户请求
- 2、设置路由来根据用户请求返回不同的响应结果
<!--more-->

但这里仅仅是返回了静态的结果，而没有动态的数据。如果需要有动态的数据，整个服务器的运行机制是怎么样的？
- 1、同样是创建服务器对象并监听用户请求
- 2、用户请求分为静态文件请求和获取动态页面。比如
`get 127.0.0.1:3000/index.html`和`get 127.0.0.1:3000/index`的结果是不同的。


- 3、当请求`127.0.0.1:3000/index`时，触发`render()`函数，传入`index.html`作为模板，传入`{name: 'ltaoo'}`作为页面数据。`render()`函数获取`index.html`文件内容，遇到`{name}`表示需要替换为`ltaoo`。
> 使用模板引擎来替代自己解析`html`并插入数据。


## 简单实现
尝试实现最简陋版本的服务端程序，只有`index`页面并显示固定数据，不从数据库中获取数据、没有静态服务器（意味着没有样式与js脚本）。

```javascript
var http = require('http')
var url = require('url')
var fs = require('fs')
var path = require('path')

http.createServer(function (req, res){
  //解析请求
  var pathname = url.parse(req.url).pathname
  if(pathname === '/index') {
    //只处理请求 index 这一种情况
    fs.readFile(path.join(__dirname, './index.html'), 'utf-8', function (err, file){
      if(err) console.log(err)
      //假数据
      var data = {
        "title": "MVC project",
        "author": "ltaoo",
        "content": "a project content"
      }
      //获取html 文件并将其中的 {{xx}} 和 data 数据进行匹配。
      var pattern = /({{)[a-z]+(}})/g;
      var ary = file.match(pattern);
      for (var i = ary.length - 1; i >= 0; i--) {
        ary[i] = ary[i].replace(/\W/g, ')
      }
      //console.log(ary)
      // 将 html 文件内的变量替换
      for (var j = ary.length - 1; j >= 0; j--) {
        file = file.replace('{{' + ary[j] + '}}', data[ary[j]])
      }
      // 渲染页面
      res.writeHead(200, {"Content-Type": "text/html"})
      res.write(file)
      res.end()
    }) 
  }
}).listen(3000)

console.log('server is listening at port 3000');

```

对应的模板文件`index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nodejs study</title>
</head>
<body>
  <article>
    <h1>{{title}}</h1>
    <p>{{author}}</p>
    <p>{{content}}</p>
  </article>
</body>
</html>
```

运行后页面成功显示预期的数据。


---


如何从该程序拓展成一个“比较”通用的服务器程序？



## 拓展

服务器的工作原理即**接收请求，返回数据**。查看express官方文档：
```javascript
app.get('/', function (req, res) {
    res.render('index', {title: "Express", content: "this is a express project"});
});
```
这里等同于
```javascript
if(pathname === '/index') {
    //coding
}
```
可以从上面的简陋版程序中看到，这一段代码属于响应请求的核心部分，代码量比较多且逻辑相同。所以抽象为函数。即 render() 函数。
该函数接收模板、数据作为参数，并在函数内渲染页面。

### render 函数
根据 pathname 来调用 template。比如
- get index, index.html
- get articles, articles.html
- get article/2016/07/26/first, 调用article.html

```javascript
function render(template, data) {
    // 这个地方的代码和简陋版的原理差不多，同样是先获取到模板文件，可以是 jade，然后将数据替换掉模板中的预留位置。最后渲染页面。这个地方是请求的终点。
}
```
这里涉及到模板引擎，如果使用 express 内置引擎，需要现在项目开始位置配置 views 的路径，则 render 函数将会在 views 目录下查找模板。

### MVC 架构
OK，上面讲到了渲染页面，但是问题在于数据是怎么得到的，又是怎么传递给 render 函数的。如果 render 函数属于 V，则 M 和 C 都暂时还没有出现。

> MVC 模式是软件工程的一种软件架构模式。...专业人员可以通过自身的专长分组：
- 控制器（Controller）-负责转发请求，对请求进行处理。
- 视图（View）-界面设计人员进行图形界面设计。
- 模型（Model）-程序员编写程序应有功能（实现算法等）、数据库专家进行数据管理和数据库设计（可以实现具体的功能）。






OK，按照维基百科的说明，MVC架构能够让专业人员分组进行工作，各自负责自己擅长的方面。是否能理解成这是属于三个方面，通过函数调用来实现联系。



比如说，用户请求 /index ，这个请求将交给 Controller 来处理，Controller 知道了用户在请求 /index，就让 Model 从数据库中调取数据，调取完成后将数据交给 Controller，Controller拿到这个数据后，把数据和 View 进行组合，最后返回页面给用户。即

V -> C -> M -> C -> V


所以 View 应该仅仅是模板，没有任何的逻辑。上面代码中，  render 函数应该是属于 Controller，因为它接受到了请求，并且处理 data 和 template，现在缺的是 Model。 路由是属于 Controller ，即 if(pathname === '/html') 开始，因为这里接收到请求并根据请求来处理数据，即逻辑部分。




#### Model

Model 属于和数据库交互的部分，在这里定义了方法来实现增删改查，并封装成模块将方法暴露给外部使用。假设：

```javascript

// 连接数据库

function fetch() {

    //从数据库中获取全部数据


}

function fetchItem(id) {

    // 根据id 从数据库中获取到指定数据


}

exports.fetch = fetch

exports.fetchItem = fetchItem

```




然后就可以在 Controller 内调用方法获取数据，这样 Controller 就将 Model 和 View 联系在一起了。


```javascript
var http = require('http'),
    url = require('url'),
    fs = require('fs'),
    path = require('path')

http.createServer(function (req, res){
  //解析请求
  var pathname = url.parse(req.url).pathname
  if(pathname === '/index') {
    //只处理这一种情况
    fs.readFile(path.join(__dirname, './index.html'), 'utf-8', function (err, file){
      if(err) console.log(err)
      /*var data = {
        "title": "MVC project",
        "author": "litao",
        "content": "a project content"
      }*/

      //将 Model 加载进来
      var model = require('model.js');
      // 获取数据
      var data = model.fetch();
      //parse html code and insert data
      var pattern = /({{)[a-z]+(}})/g;
      var ary = file.match(pattern);
      for (var i = ary.length - 1; i >= 0; i--) {
        ary[i] = ary[i].replace(/\W/g, ')
      }
      //console.log(ary)
      // 将 html 文件内的变量替换
      for (var j = ary.length - 1; j >= 0; j--) {
        file = file.replace('{{' + ary[j] + '}}', data[ary[j]])
      }
      // output 
      res.writeHead(200, {"Content-Type": "text/html"})
      res.write(file)
      res.end()
    }) 
  }
}).listen(3000)

console.log('server is listening at port 3000');

```




#### 代码整理

我们有了 M ，有了 V，也有 C，但 C 没有分离出来。

```javacript
var http = require('http'),
    url = require('url'),
    fs = require('fs'),
    path = require('path')

http.createServer(function (req, res){
  //解析请求
  var pathname = url.parse(req.url).pathname
  if(pathname === '/index') {
    //只处理这一种情况
    var index = require('index');

    index.index();


  }
}).listen(3000)

console.log('server is listening at port 3000');

```


将业务逻辑拿出来单独作为一个文件


```javascript

exports.index = function () {    

    fs.readFile(path.join(__dirname, './index.html'), 'utf-8', function (err, file){
      if(err) console.log(err)

      //表示将 Model 加载进来
      var model = require('model.js');
      // 加载数据
      var data = model.fetch();
      //parse html code and insert data
      var pattern = /({{)[a-z]+(}})/g;
      var ary = file.match(pattern);
      for (var i = ary.length - 1; i >= 0; i--) {
        ary[i] = ary[i].replace(/\W/g, ')
      }
      //console.log(ary)
      // 将 html 文件内的变量替换
      for (var j = ary.length - 1; j >= 0; j--) {
        file = file.replace('{{' + ary[j] + '}}', data[ary[j]])
      }
      // output 
      res.writeHead(200, {"Content-Type": "text/html"})
      res.write(file)
      res.end()
    }) 

}

```


这样是否实现了一个 MVC 架构的服务端程序？




## 总结

对 nodeJs 的 http 模块有了一个基本的了解，对之后学习 koa 应该有很大的帮助。






## 参考
- [Node.js：用JavaScript写服务器端程序-介绍并写个MVC框架](http://www.cnblogs.com/QLeelulu/archive/2011/01/28/nodejs_into_and_n2mvc.html)
- 《Node.Js入门》






