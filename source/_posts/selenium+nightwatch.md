---
title: selenium + nightwatch 进行前端测试
categories: 工具
tags:
- selenium
---


第一次写测试是在看《Python Web开发:测试驱动方法》的样章的时候，当通过一行命令就可以打开浏览器并完成指定操作时，感觉非常有趣。在无聊的时候，想到是否可以利用这种方式来简化每日写日报的工作，于是找到了`selenium + nightwatch`搭建的前端测试环境，在实现了这个小目标后，想着将学习到的东西用在项目中。个人觉得该测试环境最大的作用应该是可以简化填写表单的操作，如果项目存在很多表单，在进行业务测试的时候，如果每次都要填写一遍表单，将会是一个非常枯燥无聊的工作。正在进行的项目刚好有非常多的表单，所以尝试利用该工具方便之后的业务测试。

网上已经有很多关于这些的文章，不过对于测试零基础的人来说还是会有一些吃力，所以这篇笔记会提供最简单最简单的实现方式。
<!--more-->

## 环境准备

`selenium`是一个`.jar`后缀的文件，需要有`java`的运行环境，在命令行输入`java -version`查看是否安装好了。nodejs 当然也是必须的。

## 概述

总的来说，核心是使用`selenium`和`driver`操作浏览器，通过 `nightwatch`做断言。

## 依赖
需要在项目中安装两个依赖：
```bash
npm install --save-dev nightwatch selenium-standalone
```

> selenium-standalone 是 selenium 的下载和管理工具。使用该工具，可以方便下载不同浏览器的`driver`。不过不知道是个人原因还是资源被墙了，无法正常下载，所以实际上并没有真正用到`selenium-standalone`。


## 配置

只需要两个文件即可运行搭建好测试环境。

### nightwatch.json

```json
{
  "src_folders": ["tests"],
  "output_folder": "reports",
  "custom_commands_path": "",
  "custom_assertions_path": "",
  "page_objects_path": "",
  "globals_path": "",
  "selenium": {
    "start_process": true,
    "server_path": "node_modules/selenium-standalone/.selenium/selenium-server/2.53.1-server.jar",
    "log_path": "",
    "host": "127.0.0.1",
    "port": 4444,
    "cli_args": {
      "webdriver.chrome.driver": "node_modules/selenium-standalone/.selenium/chromedriver/2.22--chromedriver"
    }
  },
  "test_settings": {
    "default": {
      "launch_url": "http://localhost",
      "selenium_port": 4444,
      "selenium_host": "localhost",
      "silent": true,
      "screenshots": {
        "enabled": false,
        "path": ""
      },
      "desiredCapabilities": {
        "browserName": "firefox",
        "javascriptEnabled": true,
        "acceptSslCerts": true
      }
    },
    "chrome" : {
      "desiredCapabilities": {
        "browserName": "chrome",
        "javascriptEnabled": true,
        "acceptSslCerts": true
      }
    }
  }
}
```
重点是`selenium.server_path`和`selenium.cli_args.webdriver.chrome.driver`，指定了`selenium`和`driver`的路径，上面提到，这两个是用来操作浏览器的。

### start.js
```javascript
require('nightwatch/bin/runner.js')
```


其实现在就已经配置好了，安装好`selenium`和`driver`后就可以进行测试了。

> [点击下载](http://pan.baidu.com/s/1bYaZxW)。或者参考“更多”尝试自行下载。

## 测试代码
```js
// tests/demo.js
module.exports = {
  'open browser': function (client) {
    client.url('http://www.baidu.com')
  }
}
```
运行`node start.js -t tests/demo.js -e chrome --verbose`，如果弹出 chrome 浏览器并且打开百度，表示成功。

## 更多

不同的浏览器需要有不同的`driver`，并且`driver`还会有版本，`selenium-standalone`就是为这种情况准备的。下面的内容网上都能找到。
首先是写好配置文件，要下载什么`driver`，版本是什么：
```javascript
// selenium-conf.js
const process = require('process')
module.exports = {
    // Selenium 的版本配置信息。请在下方链接查询最新版本。升级版本只需修改版本号即可。
    // https://selenium-release.storage.googleapis.com/index.html
    selenium: {
        version: '2.53.1',
        baseURL: 'https://selenium-release.storage.googleapis.com'
    },
    // Driver 用来启动系统中安装的浏览器，Selenium 默认使用 Firefox，如果不需要使用其他浏览器，则不需要额外安装 Driver。
    // 在此我们安装 Chrome 的 driver 以便使用 Chrome 进行测试。
    driver: {
        chrome: {
            // Chrome 浏览器启动 Driver，请在下方链接查询最新版本。
            // https://chromedriver.storage.googleapis.com/index.html
            version: '2.22',
            arch: process.arch,
            baseURL: 'https://chromedriver.storage.googleapis.com'
        }
    }
} 
```
然后是一个脚本，用来从上面指定的网址下载对应的文件到项目依赖中：
```javascript
// setup.js
// 载入安装工具
const selenium = require('selenium-standalone')
// 载入配置，要安装什么驱动，什么版本的 selenium
const seleniumConfig = require('./selenium-conf.js')
// 调用 install 方法从网络下载真正的 selenium
selenium.install({
    version: seleniumConfig.selenium.version,
    baseURL: seleniumConfig.selenium.baseURL,
    drivers: seleniumConfig.driver,
    logger: function (message) { console.log(message) },
    progressCb: function (totalLength, progressLength, chunkLength) {}
}, function (err) {
    if (err) throw new Error(`Selenium 安装错误: ${err}`)
    console.log('Selenium 安装完成.')
})
```

正常情况下，执行上面的脚本就可以成功下载对应的文件到`node_modules/selenium-standalone/.selenium`文件夹下。

再回过头看看`nightwatch.json`文件，是不是`selenium.server_path`和`selenium.cli_args.webdriver.chrome.driver`都写明了版本号？如果之后有新版本了，那不仅要修改`selenium-conf.js`文件，还要修改`nightwatch.json`肯定很麻烦，所以就需要再增加一个文件，这个文件的作用是从`selenium-conf.js`中读取字段，并替换`nightwatch.json`对应内容。

```javascript
// nightwatch.conf.js
const seleniumConfig = require('./selenium-conf')
const path = require('path')
module.exports = (function (settings) {
  // 告诉 Nightwatch 我的 Selenium 在哪里。
  settings.selenium.server_path = `${path.resolve()}/node_modules/selenium-standalone/.selenium/selenium-server/${seleniumConfig.selenium.version}-server.jar`
  // 设置 Chrome Driver, 让 Selenium 有打开 Chrome 浏览器的能力。
  settings.selenium.cli_args['webdriver.chrome.driver'] = `${path.resolve()}/node_modules/selenium-standalone/.selenium/chromedriver/${seleniumConfig.driver.chrome.version}-${seleniumConfig.driver.chrome.arch}-chromedriver`
  // 打开 ie
  settings.selenium.cli_args['webdriver.ie.driver'] = `${path.resolve()}/node_modules/selenium-standalone/.selenium/iedriver/IEDriverServer.exe`
  return settings;
})(require('./nightwatch.json'))
```

需要注意的就是，要使用不同浏览器，需要`nightwatch.json`中"test_settings"字段有不同浏览器的配置，上面默认是`firefox`，有`chrome`。然后也要修改`node start.js -t tests/demo.js -e chrome --verbose`对应参数，表示要使用其他浏览器进行测试。