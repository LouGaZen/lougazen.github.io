---
title: 记一次kubernetes节点内存占用过高问题排查
date: 2021-03-08 16:44:42
tags:
---

## 序

公司的应用服务通过kubernetes集群部署的，集群包含5个节点。最近其中一节点占用一直维持95%，遂进行问题排查。

## 问题排查

### 容器的问题？

首先先把该机器设置为"不可调度"（防止后续操作死灰复燃），然后将部分内存占用高的调度到其他机器上（如nginx-ingress）

![收效甚微](https://cdn.jsdelivr.net/gh/LouGaZen/image-hosting@upic/uPic/YU8FVn.png)

内存确实降下来了（一点点😅），起码先不用频繁触发报警

### kubernetes的问题？

因为节点是购买自阿里云的ECS，且这些节点无法远程登录，排查工作一度暂停。后来换用了[Lens](https://k8slens.dev)对集群进行管理后发现可以轻松登录节点进行操作了。

![Lens - Nodes](https://cdn.jsdelivr.net/gh/LouGaZen/image-hosting@upic/uPic/dZ69hU.png)

进入shell之后执行`top`命令然后按大写`M`查看内存占用最高的服务，看到了`rsyslogd`（当时忘了截图了）

Google了一番后得知rsyslog是日志收集服务，主要是通过以下文件收集

|  路径  |  描述  |
|  ----  | ----  |
| /var/log/messages | 服务信息日志(记录linux操作系统常见的服务信息和错误信息) |
| /var/log/secure | 系统的登陆日志(记录用户和工作组的变化情况,是系统安全日志，用户的认证登陆情况 |
| /var/log/maillog | 邮件日志 |
| /var/log/cron | 定时任务 |
| /var/log/boot.log | 系统启动日志 |

然后`tail -f /var/log/messages`一下，好家伙，日志一直刷刷刷的跑

```text
kubelet_volumes.go:128] Orphaned pod "***" found, but volume paths are still present on disk. : There were a total of 1 errors similar to this.  Turn up verbosity to see them.
...
systemd[1]: Scope libcontainer-***-systemd-test-default-dependencies.scope has no PIDs. Refusing.
systemd[1]: Scope libcontainer-***-systemd-test-default-dependencies.scope has no PIDs. Refusing.
systemd[1]: Created slice libcontainer_***_systemd_test_default.slice.
systemd[1]: Removed slice libcontainer_***_systemd_test_default.slice.
systemd[1]: Created slice libcontainer_***_systemd_test_default.slice.
systemd[1]: Removed slice libcontainer_***_systemd_test_default.slice.
```

（大致恢复了事故现场，截图丢了）

再Google一番后里面这里有两个问题待解决

#### 僵尸Pod

出现`Orphaned pod "***" found`就是僵尸Pod的问题了，一般是因为删除容器的时候没删干净，导致配置文件残留在节点机器上，k8s就会不断尝试给它复活但是基本上会复活失败

进入`/var/lib/kubelet/***`，然后`cat etc-host`看一下该容器的名字，然后去`kubectl get pods | grep （容器名字）`看是否还在（一般都是不在的了）

然后对文件夹进行删除（如有mount volume之类的要先unmount，我这里没有mount所以可以直接删掉）

好了，现在看日志就没有kubelet_volumes.go的报错了

#### 内置container组件问题

看这里的描述[k8s 问题处理](https://latiaoder.com/post/kubernetes/%E9%97%AE%E9%A2%98%E5%A4%84%E7%90%86.html)似乎是container版本过旧或者不兼容，按他的说法去更新一下就好了

```shell
rpm -Uvh https://download.docker.com/linux/centos/7/x86_64/edge/Packages/containerd.io-1.2.10-3.2.el7.x86_64.rpm
```

如果是用Lens去登录节点的话更新过程会断开连接，不要慌，过一会而就能重连了。

### 执下手尾

处理完后看看`tail -f /var/log/messages`，此时没有疯狂刷日志了，但`rsyslogd`的内存占用还没下来，遂进行网管必备的重启大法

```shell
systemd restart rsyslog.service
```

此时再看top，内存占用最高的不是它了

再去阿里云的监控看看，内存占用暴跌至正常水平😄

![会心一击](https://cdn.jsdelivr.net/gh/LouGaZen/image-hosting@upic/uPic/4k4dP0.png)

## 鸣谢

> [rsyslogd、systemd-journald内存占用高解决方案](https://sunsea.im/rsyslogd-systemd-journald-high-memory-solution.html)
> [挂载失败-日志中显示僵尸pod的问题](https://developer.aliyun.com/article/688719)
> [k8s关于Orphaned pod found - but volume paths are still present on disk的处理](https://cloud.tencent.com/developer/article/1385911)
> [k8s 问题处理](https://latiaoder.com/post/kubernetes/%E9%97%AE%E9%A2%98%E5%A4%84%E7%90%86.html)
