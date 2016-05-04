---
layout: post
title: 虚拟机可以ping ip 无法ping通域名处理办法
category: 技术
tags: openstack
keywords: 
description: 
---

/etc/resolv.conf


该文件是DNS域名解析的配置文件，它的格式很简单，每行以一个关键字开头，后接配置参数。resolv.conf的关键字主要有四个，分别是：

nameserver   #定义DNS服务器的IP地址
domain       #定义本地域名
search       #定义域名的搜索列表
sortlist     #对返回的域名进行排序

/etc/resolv.conf的一个示例：

domain ringkee.com
search www.ringkee.com ringkee.com
nameserver 202.96.128.86
nameserver 202.96.128.166


正确配置后，可以ping通域名了。