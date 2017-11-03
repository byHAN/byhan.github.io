---
layout: post
title: 基于ubuntu社区源码包编译libvirt
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

## 一句话总结 ##

来来回回，前前后后，左左右右折腾了好久  
真是满屏荒唐言，一把辛酸泪  

老规矩，一句话总结下：
libvirt的包是操作系统ubuntu提供的  
确切的说，ubuntu上是由[ubuntu-cloud-archive-Team](https://launchpad.net/~ubuntu-cloud-archive)这个小组打出来的  
ubuntu是debian系，因此包是根据debian规则打的  
在libvirt官方源码的基础上，ubuntu-cloud-archive-Team基于debian打包规则，做了一系列patch，然后打出了包  

## 背景介绍 ##

需要修改libvirt的源码实现个功能  
libvirt是C编写的，修改了源码要重新编译包吧  
到[官网](http://libvirt.org/sources/libvirt-1.2.12.tar.gz)上下载源码，configure&make&make install（为了纪念，具体流程附到文后）  
遇到问题，解决问题，好不容易把包装上了  


创建个虚拟机试试，结果报错No PCI buses available  
分析日志，查看源码无果  
包是我编的，我开debug调试（分析过程见本人博客：No PCI buses available）  
调试完，发现这个bug是合理的啊  
按照代码就应该报错  

找个新环境apt-get install nova-compute  
确认装出来的libvirt完全正常，不会有上面的bug
一定是哪里配置不对，期间100次查看virsh capabilities  

最终查阅到ubuntu的打包方法以及debian的指导文档  
才发现上述问题的原因：
包不是直接用libvirtd的原生包，而是基于原生包打了一系列补丁。

为了后来人少走弯路
现将相关流程记录如下：

## 打包流程 ##

### 1.下载源码包 ###

去ubuntu的[云小组](https://launchpad.net/~ubuntu-cloud-archive)下载对应的包  
我[这里](https://launchpad.net/~ubuntu-cloud-archive/+archive/ubuntu/kilo-staging/+packages)下载kilo版本对应的包  
![](http://i.imgur.com/N8LjKLW.png)

注意：
虽然可以通过apt-get source命令下载源码包，不知道为什么下载下来是1.2.2  
而1.2.2的补丁没有解决No PCI buses available（亲身验证过）  
且官方支持的是1.2.12，因此这里推荐下载1.2.12的包  

### 2.解压包 ###

将上述下载的包，放置到ubuntu系统的对应目录中  
执行  

    dpkg-source -x libvirt_1.2.12-0ubuntu14.4~cloud1.dsc

可能需要安装dpkg-dev的包  

    apt-get install dpkg-dev
    
可以看到解压的过程中，显示打了一系列的补丁（具体情况make patch for libvirt in ubuntu这篇博文）  
![](http://i.imgur.com/EBkXLCF.png)  

### 3.编包 ###

    cd libvirt-1.2.12  
    dpkg-buildpackage -us -uc

注意1.：
这里可能需要安装一系列的依赖包  
注意2.：dpkg-buildpackage会自动完成所有从源代码包构建二进制包的工作,[具体参看这里](https://www.debian.org/doc/manuals/maint-guide/build.zh-cn.html#completebuild)  
简单说就是，他会读取./debian目录下的control和rules完成构建  

### 4.结果 ###

    cd ../

到上一层目录，也就是tar包那层目录，可以看到生成的deb包  
![](http://i.imgur.com/HdLCpW7.png)


至此，如何在ubuntu下打libvirt的deb包已经完成。


## 参考文档 ##

[https://www.debian.org/doc/manuals/maint-guide/build.zh-cn.html](https://www.debian.org/doc/manuals/maint-guide/build.zh-cn.html)  
[http://wiki.ubuntu.org.cn/PackagingGuide](http://wiki.ubuntu.org.cn/PackagingGuide)  
[http://www.voidcn.com/blog/xiaoliang_199/article/p-1235598.html](http://www.voidcn.com/blog/xiaoliang_199/article/p-1235598.html)  

kilo版本包：  
[https://launchpad.net/~ubuntu-cloud-archive/+archive/ubuntu/kilo-staging/+sourcepub/6586291/+listing-archive-extra](https://launchpad.net/~ubuntu-cloud-archive/+archive/ubuntu/kilo-staging/+sourcepub/6586291/+listing-archive-extra)  
kilo仓库地址：  
[http://ubuntu-cloud.archive.canonical.com/ubuntu/pool/main/libv/libvirt/](https://launchpad.net/~ubuntu-cloud-archive/+archive/ubuntu/kilo-staging/+sourcepub/6586291/+listing-archive-extra)  


