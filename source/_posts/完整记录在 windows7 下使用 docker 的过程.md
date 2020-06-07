---
title: 完整记录在 windows7 下使用 docker 的过程
categories: 工具
tags:
- docker
date: 2017/01/13
---

借助 docker 可以不在开发电脑中安装环境，比如 nodejs，记录下如何实现。
<!--more-->

## 下载安装

根据自己的电脑系统，在 [install-docker-for-mac-windows](https://get.daocloud.io/#install-docker-for-mac-windows) 下载最新安装包并安装。

选择好目录后下一步，提示需要安装什么组件：
- Docker Compose for Window 不知道什么用，勾选
- VirtualBox 虚拟机，如果电脑之前安装过 Oracle VM VirtualBox 可以不勾选
- Kitematic for Windows(Alpha) 使用图形界面来使用 docker，建议勾选
- Git for Windows 一个版本控制 + bash 命令终端，如果没有安装建议勾选


![安装选项-1](snipaste20170112_111810.png)


继续下一步
- Create a desktop shortcut
- Add docker binaries to PATH
- Upgrade Boot2Dcoker VM

![安装选项-2](snipaste20170112_134831.png)

三个选项全部勾选，下一步就开始安装软件到系统中，等待一会即可完成安装。

## 初始化虚拟机

打开安装目录，不出意外目录结构是这样的：
```bash
├──kitematic
├──boot2docker.iso
├──docker.exe
├──docker-compose.exe
├──docker-machine.exe
├──docker-quickstart-terminal.ico
├──start.sh
├──unins000.dat
└──unins000.exe
```

将`boot2docker.iso`拷贝至`C:\Users\用户名\.docker\machine\cache`目录下，双击打开`start.sh`文件。

![初始化虚拟机](snipaste20170112_135118.png)

当看到该画面时表示虚拟机已经安装在`Virtual Box`里面，可以打开`Oracle VM VirtualBox`查看：
![虚拟机初始化完成](snipaste20170112_135227.png)

这一步完成后，我们需要了解一个概念，就是现在我们有了两个系统，一个 windows 系统即我们直接操作的**图形界面系统**，我们称为**主机**，在主机上安装了`VirtualBox`，该软件内有 linux 虚拟机，称为**docker主机**，在 docker 主机中我们之后还会创建 linux 系统，称为**容器**。

可以通过终端显示的用户名@计算机名来区分
![三种系统](snipaste20170113_095830.png)


下面我们实现主机与 docker 主机共享文件夹。

## 主机与 docker 主机共享文件夹
假设我需要将`docker_study`作为共享文件夹，该文件目录如下：
```base
D:/docker_study
        └──koa-template
```
> `koa-template`是我已经写好的项目，可以在`docker_study`文件夹内执行`git clone https://github.com/ltaoo/koa-template.git`，会在`docker_study`文件夹生成`koa-template`文件夹。

打开`Oracle VM VirtualBox`，选中“正在运行”状态的 default 虚拟机，进入 设置-> 共享文件夹，添加共享文件夹，选中`docker_study`文件夹，勾选“自动挂载”、“固定分配”，确定。

![设置共享文件夹-1](snipaste20170112_135354.png)
![设置共享文件夹-2](snipaste20170112_135913.png)

在`default`上右键，重启该虚拟机。

重启完成后，使用`git`作为终端来连接我们的 docker 主机
```bash
docker-machine ssh default
```
> 如果提示该命令不存在，需要将 docker 的安装目录加入到环境变量中

进入到 docker 主机中，也就是终端显示`docker@default:~$`的情况，输入命令
```bash
mount
```
可以看到`docker_study`在虚拟机内的路径，进入该路径，查看是否读取到我们主机的文件与文件夹
![共享文件夹](snipaste20170112_151603.png)

```bash
# 进入 docker_study 目录
cd /docker_study
# 查看该目录下的文件与文件夹
ls
```
如果能够显示我们主机中的`koa-template`就表示配置共享文件夹成功。


## 创建容器
确保终端是处于“docker 主机”内，先下载镜像：
```bash
docker pull node
```
表示下载安装最新版本 node 的 linux 系统。
> 如果提示 Cannot connect to the Docker daemon at unix:///var/run/docker.sock，使用`sudo nohup docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock &`命令，再按一下回车即可。这一命令建议先[配置加速器](https://yq.aliyun.com/articles/29941)

然后就可以使用该镜像生成容器了：
```bash
# 查看镜像是否下载成功，看到 Repository 为 node 且 Tag 为 latest 表示成功，Created 不是下载镜像时间所以可能是几天前
docker images
# 生成容器
docker run --name koa -v /docker_study/koa-template:/app -p 3000:3000 -i -t node /bin/bash
```
这句命令，表示我们会生成一个名字为`koa`的容器，并且将 docker 主机内的 /docker_study/koa-tempate 文件夹映射(挂载)到容器中的 app 文件夹，该容器是使用刚刚下载的有最新版本 node 的镜像生成的，并且使用 /bin/bash 命令来访问该容器。

执行完上面的命令，终端将会进入容器中，即终端显示`root@xxxxxxx:/#`这样的。
容器承载着我们的应用，我们先将应用运行起来：
进入项目目录安装依赖
```bash
cd /app
npm install --registry=https://registry.npm.taobao.org --no-bin-links
```
> 如果没有`--no-bin-links`会报错，参考[unknown symlink '../express/bin/express](https://github.com/npm/npm/issues/2380)

依赖安装完成后，就可以开启我们的服务
```bash
node start.js
```

在主机浏览器打开网址`http://192.168.99.100:3000/`即可访问，并且修改主机内`docker_study/koa-template/views/base.html`的内容，刷新网页能够看到改变！

## 总结
作为一个个人开发者，使用 docker 最大的优点就是不用考虑配置环境了，想要学习新的语言直接运行一个容器就可以。

## 参考

- [Windows下运用Docker部署Node.js开发环境](https://segmentfault.com/a/1190000007955073)
- [unknown symlink '../express/bin/express](https://github.com/npm/npm/issues/2380)
- [Docker 镜像加速器](https://yq.aliyun.com/articles/29941)
