---
layout: post
title: nfs lead to Linux halt
category: 技术
tags: 技术
keywords: 
description: 
---

## 一句话总结 ##

在/etc/fstab中挂载nfs的时候，建议增加bg参数  
否则nfs-server服务不可达的时候，客户端机器Linux系统启动会被卡住  

## 现象 ##

Linux重启后卡住。  
经过分析是/etc/fstab添加了nfs内容导致  
![](http://i.imgur.com/v8crcUM.png)  

挂载文件系统的时候，nfs服务还没有启动，导致挂载失败  
然后启动过程被卡住  

## 规避 ##

在系统启动的时候grub按e  
进入recovery模式  
此时，/etc/fstab是只读模式  

需要执行  

    mount -n -o remount,rw /

然后修改/etc/fstab相关内容

reboot机器


## 修正 ##

需要追加bg参数  
第一次挂载失败后转为后台挂载  
可以启动  
![](http://i.imgur.com/wZJ7b2G.png)