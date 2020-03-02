---
title: iPhone 5s降级iOS 10.3.3简要备忘
date: 2020-03-02 13:04:07
tags:
---

## 序

好像好久都没有更新博客了，还是要养成做记录的习惯。

偶然看到5s可以降级iOS 10.3.3的帖子，二话不说GKD。中途遇到一些问题在此做个备忘

## 准备

 1.一台正常运行的iPhone 5s
 2.macOS with python3 environment
 3.科学上网

## 正式开始

 1.到[https://github.com/MatthewPierson/Vieux](https://github.com/MatthewPierson/Vieux)把降级脚本克隆下来，然后去[https://ipsw.me](https://ipsw.me)下载对应型号的iOS 10.3.3固件到脚本所在目录
 2.执行`sudo pip3 install -r requirement.txt`安装所需包（可能需要科学上网）
 3.手机连上电脑进入DFU模式（如何进入DFU可以看[这里](https://zh.ifixit.com/Guide/iPhone+4+-+4S+-+5+-+5S+-+5c+-+6+-+6S+-+%E5%A6%82%E4%BD%95%E5%9C%A8DFU%E6%A8%A1%E5%BC%8F%E5%88%B7%E6%9C%BA%E3%80%82%E5%AF%86%E7%A0%81%E6%81%A2%E5%A4%8D%E3%80%82/28229)）
 3.执行`sudo python3 vieux -i ➕上固件所在路径`
 4.等待脚本撞bug。按照漏洞发现者的说法这个漏洞并不是100%成功，没有撞成功重试即可，成功的标志是手机闪一下绿屏。绿屏过后就会自己恢复10.3.3固件

 ![降级成功](https://i.loli.net/2020/03/02/tKhObEuIjQLPqM1.png)

## 奇难杂症

### No Module Found

 这个我是感觉最坑爹的地方，pip3安装说成功，`pip3 list`也有list出来，但是执行的时候却报错，一看`pip3 --version`下巴都掉了

 ![pip3 --version](https://i.loli.net/2020/03/02/TraXm5hLEo34j89.png)

 ![坑](https://i.loli.net/2020/03/02/p5olFN8YBrPxDZ9.png)

 爬了一下google，发现pip的命令可以用`python3 -m pip ...`代替，所以上述第二步的安装环境可以执行`sudo python3 -m pip install -r requirements.txt`

### No Backend Available

 这个一开始以为是包的问题，用pip install补上后发现问题依旧，然后去脚本仓库找有没有相关的issue，嚯还真有，解决办法是通过brew安装，即`brew install libusb`

 ![No Backend Available](https://i.loli.net/2020/03/02/YzLmIZRJtuWOHi5.png)

## 鸣谢

> [5s降级10.3.3不是很难，就是需要仔细，你就可以完成降级](https://www.feng.com/post/12857994)
> [Vieux - A tool for 32/64 Bit iOS downgrades using OTA Blobs](https://github.com/MatthewPierson/Vieux)
> [Why pip3 install in python2 sitepackages](https://stackoverflow.com/questions/51055429/why-pip3-install-in-python2-sitepackages)
> [使用 Vieux 将 iphone5s 降级到 10.3.3](https://www.nazhuo.work/index.php/archives/11/)
> [iPhone 4 / 4S / 5 / 5S / 5c / 6 / 6S - 如何在DFU模式刷机。密码恢复。](https://zh.ifixit.com/Guide/iPhone+4+-+4S+-+5+-+5S+-+5c+-+6+-+6S+-+%E5%A6%82%E4%BD%95%E5%9C%A8DFU%E6%A8%A1%E5%BC%8F%E5%88%B7%E6%9C%BA%E3%80%82%E5%AF%86%E7%A0%81%E6%81%A2%E5%A4%8D%E3%80%82/28229)
