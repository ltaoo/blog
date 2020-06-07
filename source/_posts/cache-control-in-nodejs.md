---
title: nodejs 中的 Cache-Control
date: 2020/01/05
categories:
    - node
tags:
    - http
    - cache-control
    - nodejs
---

我一直对前端需要「熟悉」浏览器缓存是持疑问态度的，因为前端无法控制缓存，即无法通过代码或者某种方式来指定某些资源是否需要缓存。
举个例子，我们的 `index.html` 文件中引入了 `bundle.js` 这个文件，我们希望在每次刷新页面时，都不要使用缓存，每次都去服务器获取最新的 `bundle.js` 文件。
要实现这个需求，只从前端的角度来思考，怎么做？
或许有人会说 `meta` 标签支持指定 `Cache-Control`，但这是针对全站资源，如果只是针对特定的资源如 `bundle.js` 获取最新的，其他资源照样使用缓存。
似乎是无法实现的。那什么角色应该了解这些内容呢？
<!--more-->

应该是后端，或者说写 `node` 应用的开发，因为是否缓存是服务端来决定的。
前端只需要知道这么一个东西，能在需要时查询到解决方案就行了，并不需要特意去学习这些知识，原因前面也说了，前端的实际业务场景中根本用不上。

## 服务端处理请求响应资源
浏览器，或者说客户端吧，所有的请求都是到达服务端，服务端再返回资源，再返回资源时，可以额外再返回一些信息用来告诉浏览器这个资源要不要缓存、缓存多久。

### nodejs 实现服务器

https://codesandbox.io/s/wizardly-mahavira-og20m
```js
const http = require('http');
const fs = require('fs');
const path = require('path');
const server = http.createServer((req, res) => {
    const { method, url } = req;
    if (method === 'GET') {
        if (url === '/') {
            res.writeHead(200, {
                'Content-Type': 'text/html'
            });
            res.write(`
<img src="/example.png" />
            `);
            res.end();
            return;
        }
        if (url === '/example.png') {
            const filepath = path.resolve(__dirname, url.slice(1));
            try {
                const imgContent = fs.readFileSync(filepath);
                res.writeHead(200, {
                    'Content-Type': 'image/png'
                });
                res.write(imgContent);
            } catch (err) {
                res.writeHead(404);
            }
            res.end();
            return;
        }
    }
    res.writeHead(404);
    res.end();
});
const PORT = 8080;
server.listen(PORT, () => {
    console.log(`Server is listening at port ${PORT}`);
});
```

打开浏览器访问 https://og20m.sse.codesandbox.io/ 并打开控制台，会看到加载了一张 `example.png` 图片，并且无论怎么刷新，每次请求都会从服务端请求最新的图片，而不会出现 `200(from memory cache)` 或者 `304`。

> 这里不能直接在浏览器里访问图片，即不能直接访问 https://og20m.sse.codesandbox.io/example.png 来测试是否使用缓存，因为每次都是新请求，具体为什么可能要看浏览器怎么处理这种直接访问图片的情况了。

### 使用 cache-control 或 expires 实现强缓存

强缓存就是客户端在发出请求前，自己就能判断这次请求是否使用缓存，而不会向服务器发出实际请求。
实现强缓存可以用 `cache-control` 或者 `expires`，但大部分情况都是两者一起使用，因为 `cache-control` 是 `http1.1` 规范内的，如果浏览器无法识别 `cache-control` 头，还可以用 `expires`。
要使用 `cache-control` 也很简单，在响应头内增加该字段即可

```js
// 省略重复内容
                res.writeHead(200, {
                    'Content-Type': 'image/png',
                    'Cache-Control': 'max-age=30',
                });
```
这样写表示缓存时间为 30 秒，30 秒内的重复请求都不会处理，让浏览器用缓存。再来看看增加该字段后请求有什么变化。

![增加 cache-control 后的请求](增加 cache-control 后的请求.png)

可以看到 `Size` 显示 `memory cache`，`Time` 更是变成了 0。超过 30 秒后刷新，会重新请求服务器，再看看这次的请求

![正常请求](正常请求.png)

`expires` 也是类似，把 `cahce-control` 删除，增加 `Expires` 字段

```js
// 省略重复内容
                res.writeHead(200, {
                    'Content-Type': 'image/png',
                    'Expires': new Date(Date.now() + 30000).toGMTString(),
                });
```

结果和上面一样，只是 `response headers` 有点不同

![使用 Expires 后的响应头](expires-request.png)

### 协商缓存
与强缓存相对的就是所谓的协商缓存了，从名字也能看出来，「协商」，表示要和服务器通信后才能决定是否使用缓存。最容易理解的协商缓存实现方式是增加 `ETag` 字段，该字段简单来说是一个（文件路径 + 最后修改时间）再经过处理后得到的字符串。

这样生成的 `ETag` 就能表达唯一性了
- 2020-01-05 22:00:00 修改的 bundle.js
- 2020-01-05 23:00:00 修改的 bundle.js

这两个文件的 `ETag` 是不同的，浏览器第一次请求 `bundle.js` 时假设返回的 `ETag` 是 `bundle220000`，在 22:00:00 至 22:59:59 这段时间内每次浏览器请求，都会重新计算 `bundle.js` 的 `ETag` 一直等于 `bundle220000`，并和请求时传过来的 `ETag` 判断是否一致，如果一致就只返回响应头，不返回响应体。
用代码来简单说明

```js
// 在响应时增加 ETag 字段
                res.writeHead(200, {
                    'Content-Type': 'image/png',
                    'ETag': computeETag(filepath),
                });
```

```js
// 在请求时获取 If-None-Match 字段和要请求的资源计算 ETag 比对并处理
const server = http.createServer((req, res) => {
    const { method, url, headers } = req;
    if (method === 'GET') {
        // 省略
         if (url === '/example.png') {
            const filepath = path.resolve(__dirname, url.slice(1));
            if (computeETag(filepath) === headers['if-none-match']) {
                res.writeHead(304, 'Not Modified', {
                    'Content-Type': 'image/png',
                });
                res.end();
            }
            // 省略
         }
    }
    // 省略
});
function computeETag(filepath) {
    return filepath;
}
```

省去了读取文件，速度会稍微快一些吧。除了 `ETag`，`last-Modified` 也是用作协商缓存的，原理类似，只是不用拿到资源路径和最后修改时间再做处理，直接用最后修改时间就行。

- 1、如果 headers['if-modified-since'] === getLastModifiedTime(url) 跳 2 否则跳 3
- 2、返回 304 状态码和响应头
- 3、读取文件内容并返回 200 状态码和响应头

## cahce-control

前面只提到了 `cache-control` 设置 `max-age` 的场景，但除了 `max-age`，响应头还可以设置其他值，如 `no-cache` 和 `no-store`。

如果想浏览器每次请求都请求最新的内容，那么 `cache-control` 需要设置成 `no-store`；`no-cache` 和 `max-age=0` 意义是相同的，即浏览器会向服务器发送请求，服务器如果认为可以使用缓存，那么浏览器就会「跳过获取响应体」，不知道服务器要如何处理这种情况。

## api 请求的缓存

`api` 请求和响应，本质上和静态资源的请求响应是没有区别的，所以是否可以缓存 `api` 请求呢？之前有考虑过这类场景，将 `api` 请求人为地分类
- 1、永远不会变化的
- 2、变化频率小，可能一天内不会发生变化的
- 3、变化频率大，每次请求都可能不同

第一类就和静态资源比如图片一样，一张图片永远都不会发生变化，所以 `ETag` 是永远不会变的，即使 `cache-control` 设置的时间超过了，也只是走协商缓存，当然大部分情况都是设置一个超长的 `max-age`。
第二类变化频率小，比如搜索组织内成员，如果是公司组织，变化频率是非常小的，也能够容忍不会立即更新，所以可以设置一定时间内走缓存。
第三类就不用说了，完全不能走缓存。

`api` 请求还适用于缓存的场景是数据量非常大的情况，这时即使响应内容会频繁变化，但可以应用协商缓存，即对响应内容计算唯一值，一样走 `ETag` 的逻辑，是可以省去传输大量数据的时间的，虽然服务器一样要从数据库读这个数据，还要做额外的计算，但流量应该是能减少。

## 参考
- [Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
