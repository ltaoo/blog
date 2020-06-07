---
title: callback、promise、generator和await
categories: JavaScript
tags:
- promise
- generator
---

学习 promise、generator和最新的 await。

假设现在有这么一个需求：
> 用户输入关键词查询记录，先按照书籍ISBN码进行搜索，如果没有搜索到，就按照书籍名称搜索，如果没有搜索到，就按照借阅会员学号搜索，如果没有搜索到，就按照借阅会员姓名搜索。

<!--more-->
数据是这样的：

```json
{
  "record": [
    { "id": 1, "title": "小王子", "author": "圣埃克絮佩里",  "isbn": "9787511305206", "number": "学号一", "name": "ltaoo"},
    { "id": 2, "title": "草房子", "author": "曹文轩",  "isbn": "9787020078400", "number": "学号二", "name": "hello"},
    { "id": 3, "title": "青铜葵花", "author": "曹文轩",  "isbn": "9787534633362", "number": "学号三", "name": "world"},
    { "id": 4, "title": "城南旧事", "author": "林海音",  "isbn": "9787561344972", "number": "学号四", "name": "and"},
    { "id": 5, "title": "HTTP权威指南", "author": "吉尔利 (David Gourley)",  "isbn": "9787115281487", "number": "学号五", "name": "some"},
    { "id": 6, "title": "JavaScript权威指南", "author": "弗兰纳根(David Flanagan) ",  "isbn": "9787111376613", "number": "学号六", "name": "thing"},
    { "id": 7, "title": "JavaScript设计模式与开发实践", "author": "曾探",  "isbn": "9787115388889", "number": "学号七", "name": "about"},
    { "id": 8, "title": "JavaScript高级程序设计", "author": "泽卡斯 (Zakas. Nicholas C.)",  "isbn": "9787115275790", "number": "学号八", "name": "you"}
  ]
}
```

为了方便使用了`json-server`作为接口返回这些数据。

- http://127.0.0.1:3000/record?author=林海音 会返回第四条数据
- http://127.0.0.1:3000/record?title=JavaScript权威指南 会返回第六条数据

详细使用可以查看 json-server 文档。


## callback
由于每一次的搜索，都依赖于上一次的结果，所以必须在上一次查询的结果中进行下一次的查询。
这是我之前的写法：

```javascript
var req = require('request');

var searchByAuthor = 'http://127.0.0.1:3000/record?author=';
var searchByBookName = 'http://127.0.0.1:3000/record?title=';
var searchByNumber = 'http://127.0.0.1:3000/record?number=';
var searchByName = 'http://127.0.0.1:3000/record?name=';

function search(params, cb) {
  req(searchByAuthor+params, function (err, response, body) {
    console.log('按照作者名查询');
    if(JSON.parse(body).length !== 0) {
      cb(body)
    }else {
      req(searchByBookName+params, function (err, response, body) {
        console.log('按照书名查询');
        if(JSON.parse(body).length !== 0) {
          cb(body)
        }else {
          req(searchByNumber+params, function (err, response, body) {
            console.log('按照学号查询');
            if(JSON.parse(body).length !== 0) {
              cb(body)
            }else {
              req(searchByName+params, function(err, response, body) {
                console.log('按照姓名查询');
                if(JSON.parse(body).length !== 0) {
                  cb(body)
                }else {
                  cb('empty')
                }
              })
            }
          })
        }
      })
    }
  })
}
// 进行查询
var query = encodeURI('hello');
search(query, function (result) {
  console.log(result);
})

```
如果条件再多几个，嵌套更深，整个代码会往右边偏。。。。为了解决这个问题，出现了 Promise。

## Promise

Promise 是对象，有 then 和 catch 方法，可以获取到成功和失败各自的返回结果。

```javascript
function temp(params) {
    return new Promise(function (resolve, reject) {
      req(params, function (err, response, body) {
        if(err) {
          reject(err);
        }
        if(JSON.parse(body).length !== 0) {
          resolve(body);
        }else {
          reject('empty');
        }
      })
    })
  }
```

定义了一个`temp`函数，调用该函数后会返回一个 promise 对象，调用该对象的 then 方法接收成功的返回结果，调用 catch 方法接收失败的返回结果。

所以上面的例子这么改写：

```javascript
var req = require('request');

var searchByAuthor = 'http://127.0.0.1:3000/record?author=';
var searchByBookName = 'http://127.0.0.1:3000/record?title=';
var searchByNumber = 'http://127.0.0.1:3000/record?number=';
var searchByName = 'http://127.0.0.1:3000/record?name=';

function search(param, cb) {

  function temp(params) {
    return new Promise(function (resolve, reject) {
      req(params, function (err, response, body) {
        if(err) {
          reject(err);
        }
        if(JSON.parse(body).length !== 0) {
          resolve(body);
        }else {
          reject('empty');
        }
      })
    })
  }

  temp(searchByAuthor+param)
    .then(function (result){
      console.log('按照作者名查询');
      cb(result);
    })
    .catch(function (err) {
      return temp(searchByBookName+param);
    })
    .then(function(result) {
      console.log('按照书名查询');
      cb(result);
    })
    .catch(function (err) {
      return temp(searchByNumber+param);
    })
    .then(function (result) {
      console.log('按照学号查询');
      console.log(reuslt);
    })
    .catch(function (err) {
      return temp(searchByName+param);
    })
    .then(function (result) {
      console.log('按照姓名查询');
      cb(result);
    })
    .catch(function(err) {
      cb('empty');
    })
}
// 开始查询
var query = encodeURI('hello');
search(query, function (res) {
  console.log(res);
})
```

可以很明显看到不存在嵌套了，而是采用了链式调用。。。重点在于`catch`方法返回一个新的 promise 对象。

## generator

generator 可以将异步函数变成同步的。道理很简单， yield 关键字会暂停函数的运行，直到返回值了才继续往下执行。

参考 - http://www.voidcn.com/blog/zhiweiusetc/article/p-6093190.html

使用 generator 改写上面的例子：

```javascript
var req = require('request');

var searchByAuthor = 'http://127.0.0.1:3000/record?author=';
var searchByBookName = 'http://127.0.0.1:3000/record?title=';
var searchByNumber = 'http://127.0.0.1:3000/record?number=';
var searchByName = 'http://127.0.0.1:3000/record?name=';

var query = encodeURI('学号三');


function run(generatorFunction) {
  var generatorltr = generatorFunction(resume);

  function resume(err, response, body) {
    generatorltr.next(JSON.parse(body));
  }
  // 这只是用来启动函数的。
  generatorltr.next();
}

run(function *search(resume) {
  var result = null;
  
  // 这里每一行返回的值，都是这次调用接口得到的 body。
  result = yield req(searchByAuthor+query, resume);
  // 判断 body 中是否有值，了解这次查询是否成功，如果不成功，就继续查询
  result = result.length === 0 ? yield req(searchByBookName+query, resume) : result;
  result = result.length === 0 ? yield req(searchByNumber+query, resume) : result;
  result = result.length === 0 ? yield req(searchByName+query, resume) : result;

  console.log(result);
})
```

不是嵌套，也不是链式调用，而是按照代码顺序。而且是在开始时定义一个变量，通过接口查询后，就能够直接拿到值，写起来像是同步的。

这个例子重点在 yield 返回 generatorltr.next() 传的参数。

## async of es7

代码比前面都简单，不过需要使用 babel 才能够使用到 es7 的这个新特性。直接上代码：

```javascript
'use strict';
var req = require('request');

var searchByAuthor = 'http://127.0.0.1:3000/record?author=';
var searchByBookName = 'http://127.0.0.1:3000/record?title=';
var searchByNumber = 'http://127.0.0.1:3000/record?number=';
var searchByName = 'http://127.0.0.1:3000/record?name=';

var query = encodeURI('hello');


function temp(params) {
  return new Promise(function (resolve, reject) {
    req(params, function (err, response, body) {
      if(err) {
        reject(err);
      }
      resolve(JSON.parse(body));
    })
  })
}
async function search() {
  var result = null;

  result = await temp(searchByAuthor+query);
  result = result.length === 0 ? await temp(searchByBookName+query) : result;
  result = result.length === 0 ? await temp(searchByNumber+query) : result;
  result = result.length === 0 ? await temp(searchByName+query) : result;

  console.log(result);
}

search();
```
看起来和 generator 一模一样，因为 async 其实就是 generator 的语法糖。
> 一比较就会发现，async 函数就是将 Generator 函数的星号（*）替换成 async，将 yield 替换成 await，仅此而已。

## github repository
https://github.com/ltaoo/learnCallback

## 参考 

基础 

- [ES6 Generator介绍](http://www.alloyteam.com/2015/03/es6-generator-introduction/)

本文参考

- [用ES6 Generator替代回调函数](http://www.html-js.com/article/A-day-to-learn-JavaScript-to-replace-the-callback-function-with-ES6-Generator)
- [创建koa2工程](http://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/001471087582981d6c0ea265bf241b59a04fa6f61d767f6000)
- [async 函数的含义和用法](http://www.ruanyifeng.com/blog/2015/05/async.html)

拓展阅读

- [co最简版实现](https://cnodejs.org/topic/53474cd19e21582e740117df)


