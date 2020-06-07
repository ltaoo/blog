---
title: 在Ubuntu系统上搭建Hexo博客
categories: 随笔
tags:
- ubuntu
- hexo
---

搭建博客并不是很困难，无论是 wordPress 还是别的什么开源博客系统，有很多选择，所以没有技术方面的难度。但是迟迟没有搭建自己的博客，完全是因为觉得要运营博客将会耗费很多时间，比如搭建好博客，就会想用美观的主题；折腾一段时间后，准备自己写主题；发现自己的产出太少，就想着翻译国外的文章等等。总之感觉会耗费很多时间，每天下班后只有三个小时的时间，并不想全部花在博客上。但是，现实还是残酷的，考虑到找工作的时候，如果有一个博客，面试官首先会通过博客来了解个人能力，可以节省面试双方的时间，而且有个人博客很明显可以加分啊。所以还是准备搭建一个个人博客。作为 Web前端，当然要用 Nodejs 相关的博客了，一番搜索后选择了 Hexo。
<!--more-->

## 服务器
首先搜索了一些搭建 Hexo博客的教程，提到了使用 github 来存放博客文件。因为自己有阿里云服务器，所以将博客文件放置在阿里云上。服务器的操作系统为 Ubuntu 14。
### 环境配置
使用 Hexo 需要 Nodejs环境。
```bash
apt-get install nodejs
apt-get install npm
```
> npm 是 nodejs 的一个包管理工具，用来下载包，所以可以看到下面的命令都是 `npm`开头。

如果是在自己的本地 Ubuntu 系统操作，需要在命令前面加上`sudo`，因为阿里云默认是使用`root`用户登录，不需要`sudo`。
通过查看版本来确认是否安装成功。
```bash
node -v
npm -v
```
出现版本号则表示安装成功。

### 工具安装
- hexo-cli，全局安装，用来初始化项目。
```bash
npm install hexo-cli -g
```
同样，我们通过查看版本号来确认是否安装成功：
```bash
hexo -v
```

## 开始

### 初始化
找到一个可以存放项目的合适路径，使用`hexo-cli`初始化项目：
> 有两种方式来初始化项目。1、先新建项目文件夹，在项目文件夹内执行`hexo init`命令。2、使用下面的方式。

```bash
hexo init {projectName}
```
然后可以看到在当前目录生成了{projectName}文件夹，文件夹下有很多文件与文件夹，`node_modules`为项目依赖，不用理会，具体看另外的文件。
- _config.yml
- package.json
- scaffolds
- source
- themes

从名字就能很明显看出各自的作用。

### 安装依赖
安装项目依赖：
```bash
npm install
```

### 开启网站
开启服务：
```bash
hexo server
```
然后会提示：
```bash
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```
服务在4000端口开启，访问`{服务器ip地址}:4000`即可查看网站。
> 我的 ip 是139.129.17.174，那我需要通过 139.129.17.174:4000 即可访问网站。

### 常驻
关闭终端后发现网站也无法访问了，所以我们需要让该服务常驻。
```bash
nohup hexo server &
```
然后会显示：
```bash
appending output to nohup.out
```
表示`hexo server`命令输出将保存至`nohup.out`文件。这时输入任意键，回车后退回到 shell窗口（即可以输入命令的窗口），使用`exit`来退出终端。

## 上传、更新文章
在后台运行网站服务时，修改`_config.yml`文件并不能即时更新，需要断开服务后重启服务。首先查找进程，然后关闭：
```bash
ps -ef
# 找到 hexo 对应的 PID number
kill {PID number}
```
### 上传文章
新建文件有两种方式：
- 1、命令行生成 md 文件，在 md 文件内编写文章
- 2、直接上传已写好的 md 文件

第一种方式会默认在头部添加`yml`信息。但由于我习惯在为知笔记中写笔记，将笔记整理后成为文章发布到博客上。这里采用第二种方式。直接使用 FTP软件将 md 文件上传至`source/_post`文件夹内，使用命令：
```bash
hexo g
```
> g 应该是 generate 的简写。就好像 server 可以简写成 s。

刷新网站后即可看到新文章。教程中提到可以在 md 文件头部加上一些文章信息，比如：
```yml
--- 
title: hello-world //在此处添加你的标题。 
date: 2014-11-7 08:55:29 //在此处输入你编辑这篇文章的时间。 
categories: Exercise //在此处输入这篇文章的分类。 
toc: true //在此处设定是否开启目录，需要主题支持。
---
```
`title`字段自动成为主标题，即和`#hello-world`效果相同并且自动附带该文章链接。所以这篇文章的头部可以改成这样：
```yml
---
title: 开通个人博客
---
搭建博客并不是很困难，无论是 wordPress 还是别的什么开源博客系......
```
很明显这样好很多。

### 更新文章
教程中并没有提到更新文章，所以猜想直接替换同名 md 文件或者直接在 md 文件内修改即可完成更新。

> 的确是这样，但是有一个问题，在服务开启的状态下，`title`字段为中文时导致整片文章内容为空。断开服务再重新开启即可正常显示。

## 总结
博客搭建很顺利，两个小时就折腾完毕（主要是搜索如何让服务常驻后台和如何关闭常驻服务）。但是很明显博客的样式、功能都很少，所以下一步是优化博客（好吧，我就知道会有这一步的）。

上面提到在为知笔记中写，写好后每次都需要使用 FTP 工具上传至服务器。如果先在本地写好，push 到 github 上，再从服务器上更新以获得最新版本，工作量还是比较大，所以考虑使用通用的方式，即使用 github 保存网站文件。可以参考- [史上最详细的Hexo博客搭建图文教程](https://xuanwo.org/2015/03/26/hexo-intor/)完成该方式。

## 参考
- [史上最详细的Hexo博客搭建图文教程](https://xuanwo.org/2015/03/26/hexo-intor/)

- [Hexo](https://github.com/hexojs/hexo)

- [linux的nohup命令的用法。](http://www.cnblogs.com/allenblogs/archive/2011/05/19/2051136.html)

- [How to get the process ID to kill a nohup process?](http://stackoverflow.com/questions/17385794/how-to-get-the-process-id-to-kill-a-nohup-process)

