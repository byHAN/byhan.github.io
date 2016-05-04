---
layout: post
title: enventlet backdoor
category: 技术
tags: openstack
keywords: 
description: 
---

# 背景知识 #

今天在看进场的后台分析，看到一个eventlet的blackdoor蛮有趣，记录如下
后续可以用来分析线程栈，类似于java中的stackdump.
可以用来分析死锁及其性能问题。

# 如何使用 #

在配置文件中配置端口信息，这里以nova.conf为例
配置的时候可以配置为0，或者指定特定端口，或者指定一个段

![](http://i.imgur.com/oa8QXj6.png)

另开一个终端，telnet到对应端口即可

![](http://i.imgur.com/dA6zllg.png)

自己也可以写个小例子，例子可以[参考这里](http://eventlet.net/doc/modules/backdoor.html)

# 源码分析 #

我们都知道一个进程服务启动的是时候是调用的服务的service的serve方法

![](http://i.imgur.com/O5kS915.png)

然封装调用的oslo的

![](http://i.imgur.com/4G6sk8r.png)

也就是调用的这里,根据配置文件构造出launcher对象

![](http://i.imgur.com/NIfJDkH.png)

在ServiceLauncher的基类中Launcher中，构造对象的时候会调用eventlet的backdoor方法，然后返回一个绑定的端口

![](http://i.imgur.com/27wdN18.png)

具体的可以参考eventlet_backdoor这个文件，主要是孵化了一个线程，线程调用了eventlet的backdoor_server，具体[参考这里](http://eventlet.net/doc/modules/backdoor.html)

![](http://i.imgur.com/Wr8MGAL.png)
