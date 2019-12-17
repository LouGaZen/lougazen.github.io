---
title: 通过brew cask按照指定版本的应用
date: 2019-12-17 22:58:25
tags:
---

# 序

最近在全新安装DataGrip(2019.3)的时候发现使用以前的洗白工具无法输入序列号，但是从2019.2版本升上来的idea和webstorm却能正常使用。作为一名idea系列的重度使（bai）用（piao）者实在是难受，首先想到的办法就是安装一个旧版本的DataGrip。但是Cask却不像Formulae通过一个@就能指定版本号，需要另辟蹊径。

# 准备工作

先把DataGrip完全卸载
![](https://i.loli.net/2019/12/17/cEWU3ZDBgvsYKHt.png)

然后执行一波`brew update`避免待会配置文件被覆盖

# 寻找旧版应用的版本号及sha256信息

对于大多数开发工具来说官网都会给出旧版的软件版本号或者下载地址等其他信息，如果有sha256值提供就在好不过了，可惜[DataGrip](https://www.jetbrains.com/datagrip/download/other.html)没有给出，只好下载回来自己算sha256

![](https://i.loli.net/2019/12/17/QNCOB5ZTRifz3Mu.png)

![](https://i.loli.net/2019/12/17/gVTJ1B6ewIvcLus.png)

# 替换目标软件的Cask下载源地址

执行`brew cask edit datagrip`就会在默认编辑器打开这个软件的配置文件

![](https://i.loli.net/2019/12/17/UnozwtJDTMFXYh9.png)

我们主要改的是红框中的两个信息，把它替换成指定版本的应用信息，保存退出

![](https://i.loli.net/2019/12/17/r569XZWOVjmcEwo.png)

![](https://i.loli.net/2019/12/17/tLVYFqg8nT2bKGz.png)

很明显这个配置文件所在的目录是个git仓库，所以改错了不用怕，到时候一波`git reset --hard`即可复原

# 执行安装命令

这个时候执行`brew cask install datagrip`，顺利的话就会自动下载旧版本并且顺利安装
![](https://i.loli.net/2019/12/17/oaHz1MYjsRQ9wp6.png)

# 鸣谢

> [Use Homebrew Cask to downgrade or install specific version of package](https://zeckli.github.io/en/2016/11/05/use-homebrew-cask-to-downgrad-or-install-en.html)
