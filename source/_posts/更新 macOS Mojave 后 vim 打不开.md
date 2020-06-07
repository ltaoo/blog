---
title: 更新 macOS Mojave 后 vim 打不开
categories: 随笔
tags:
- vim
date: 2018/10/22
---

```bash
Vim: Caught deadly signal SEGV
Error detected while processing function youcompleteme#Enable[5]..<SNR>31_SetUpPython:Vim: Finished.

line   39:
Exception MemoryError: MemoryError() in <module 'threading' from '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/threading.pyc'> ignored
```

使用`vim`时报错，从报错信息中可以看出是`youcompleteme`相关的东西有问题，其实就是`Valloric/YouCompleteMe`这个插件有问题。
幸运的是这个问题也有人遇到，并且有人给出了各自的解决方案。

<!--more-->

## 0、禁用插件

将`~/.vimrc`中取消该插件即可。
但如果还是想要使用该插件，要怎么解决呢？

## 1、重新安装

```bash
brew uninstall vim && brew install vim
```

不幸运的是这种方式对我无效。

## 2、指定正确的 python 路径

有人提出这个问题的核心在于 vim 使用了一个错误的 python 路径，所以只需要告诉 vim 正确的 python 路径就可以了。

通过`where python`可以得到正确的`python`路径，然后修改`~/.vimrc`，在最后面加上

```vim
let g:ycm_path_to_python_interpreter="/usr/bin/python"
```

`/usr/bin/python`是我的`python`路径。

但还是无效。

> 最后面有人提到`ycm_path_to_python_interpreter`已经废弃，使用`ycm_server_python_interpreter`替代，但可惜还是不行。

## 3、更新 YouCompleteMe 插件

由于插件是很久前安装的，所以想通过更新插件来尝试能否修复该问题。进入`~/.vim/bundle/YouCompleteMe`路径，通过`git pull`拉取最新代码即可。

```bash
git pull
git submodule update --init --recursive
```

如果没有第二行命令启动后会显示：

```bash
YouCompleteMe unavailable: cannot import name urljoin
```

重新尝试启动`vim`，但打开后的一瞬间又关闭了，并且显示：

```bash
Vim: Caught deadly signal SEGV
Error detected while processing function <SNR>45_PollServerReady[7]..<SNR>45_Pyeval:Vim: Finished.

line    4:
Exception MemoryError: MemoryError() in <module 'threading' from '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/threading.pyc'> ignored
[1]    32074 segmentation fault  vi
```

还好有人遇到同样的问题，通过如下命令重新安装`vim`

```bash
brew uninstall vim
brew install vim --with-lua --with-override-system-vi
```

**完成后重新启动`shell`**，再次尝试启动`vim`，又有了不同的提示：

```bash
The ycmd server SHUT DOWN (restart with ':YcmRestartServer'). YCM core library too old; PLEASE RECOMPILE by running the install.py script. See the documentation for more details.
```

提示非常明确，通过运行`install.py`脚本来重新编译`YCM core library`：

```bash
# ~/.vim/bundle/YouCompleteMe
python install.py
```

这次是真的正常了。

## 补充

直接在命令行使用 vi 调用正常，但在通过其他软件调用时（如 git ci 会调用 vim）还是会出现和
开始完全相同的问题。

**在将 .vimrc 中 YouCompleteMe 注释、取消注释 后，神奇地正常了**

## 参考

- [Error on start: Error detected while processing function youcompleteme](https://github.com/Valloric/YouCompleteMe/issues/1652)
- [YouCompleteMe fails because of an ImportError: cannot import name 'urljoin' ](https://github.com/Valloric/YouCompleteMe/issues/2583)
- [vim segment fault when i upgrade to macos mojave](https://github.com/Valloric/YouCompleteMe/issues/3165)

