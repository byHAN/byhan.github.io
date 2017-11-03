---
layout: post
title: gdb debug libvirt
category: 技术
tags: 虚拟化层
keywords:
description: 
---

## 一句话总结 ##

最近调试了libvirt，将过程记录如下 

## gdb简介 ##

openstack是用的python，我们调试的时候用的是pdb  
libvirt是c语言的，可使用gdb(或者其他类似工具)进行调试  

### 0.下载源码 ###

先下载libvirt的源码  

    wget http://libvirt.org/sources/libvirt-1.2.7.tar.gz

也可以直接谷歌，有时候这里下载速度较慢  

注意：  
上面取到的是libvirt的原生代码。  

如果需要取相关操作系统下的代码会有不同，如我的libvir是安装到ubuntu上的，则需要到[这里取](https://launchpad.net/~ubuntu-cloud-archive) 
并且，如果是ubuntu下的话，编译请参考本人[这篇博客](http://www.hanbaoying.com/2016/07/28/compile-libvirt-ubuntu.html)  
然后安装gdb样式的deb包就可以调试了  

如果需要取用centos的社区包，则需要到[这里取](http://vault.centos.org/centos/7/updates/Source/SPackages/)  
详细可以参考本人[这篇博客](http://www.hanbaoying.com/2016/07/28/compile-libvirt-red-hat-7.html)

下面还是依照原生代码做通用介绍  

### 1.编译 ###


#### 解压缩 ####

将libvirt的tar包解开  

    tar -zxf libvirt-1.2.12.tar.gz
    
#### 配置 ####

执行configure  
这一步一般用来生成 Makefile，为下一步的编译做准备。  
可以通过在 configure 后加上参数来对安装进行控制  
比如代码:./configure –prefix=/usr 意思是将该软件安装在 /usr 下面，执行文件就会安装在 /usr/bin （而不是默认的 /usr/local/bin)。  
有一些软件还可以加上 –with、–enable、–without、–disable 等等参数对编译加以控制，如下面--with-sanlock  
可以通过允许 ./configure –help 察看详细的说明帮助。

    ./configure --prefix=/usr --localstatedir=/var  --sysconfdir=/etc --with-sanlock --with-secdriver-apparmor=no

开启apparmor会有一系列权限问题，这里设置为关闭  
执行这步可能会有依赖错误，需要根据错误安装相关的依赖包  
如下面的error1 error2等  

编译libvirtd的时候需要将调试打开  
在执行configure的时候带上--enable-debug=yes  
![](http://i.imgur.com/P2vbcxL.png)

#### 编译 ####

通过make命令执行编译  
这里启动多线程编译（10个线程）

    make -j 10

#### 安装 ####


    make install

如果安装后立即使用libvirt这些程序库时，遇到找不到对应库文件的错误提示  
这时可能需要运行 ldconfig 等工具来更新刚才安装的共享库。


## 2.gdb进程 ##

编译安装(具体参见本人相关博文)  
找到libvirtd的进程  
（libvirtd是守卫进程，调试的时候需要调试libvirtd,看有篇博文gdb virsh是不对的）  
![](http://i.imgur.com/PWo6Ain.png)

执行gdb（以上例进程id为25704作为例子）

    gdb libvirt 25704

![](http://i.imgur.com/ehErYQQ.png)

（这里由于把日志打开了，会有一系列的相关输出）

## 3.断点 ##

使用gdb的命令就可以进行调试了  
（具体命令请google）

这里以打断点为例子  
![](http://i.imgur.com/tow5zSf.png)

具体在哪打断点就要靠对源码的理解了。  
或者根据错误信息搜索源码  

## 4.触发 ##

另外开一个shell，执行virsh create local.xml  
![](http://i.imgur.com/dJbBaXQ.png)  
可见，已经进入到断点了  

可以执行bt查看调用栈  
![](http://i.imgur.com/jsq0Kax.png)

在不同的调用层次上9