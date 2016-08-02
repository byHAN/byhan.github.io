---
layout: post
title: compile libvirt（red hat 7）
category: 技术
tags: 虚拟化层
keywords: 
description: 
---
## 背景介绍 ##

需要修改libvirt的代码验证个问题  
下面记录编译安装libvirt的步骤  
os:ret hat7
libvirt version:1.2.7

检查当前是否安装过libvirt  
![](http://i.imgur.com/4Glm9CI.png)

## 下载libvirt的源代码 ##

这里安装的是1.2.7

    wget http://libvirt.org/sources/libvirt-1.2.7.tar.gz

## 解压缩 ##

将libvirt的tar包解开  

    tar -zxf libvirt-1.2.7.tar.gz
    
## 配置 ##

执行configure  
这一步一般用来生成 Makefile，为下一步的编译做准备。  
可以通过在 configure 后加上参数来对安装进行控制  
比如代码:./configure –prefix=/usr 意思是将该软件安装在 /usr 下面，执行文件就会安装在 /usr/bin （而不是默认的 /usr/local/bin)。  
有一些软件还可以加上 –with、–enable、–without、–disable 等等参数对编译加以控制，如下面--with-sanlock  
可以通过允许 ./configure –help 察看详细的说明帮助。

    ./configure --prefix=/usr --localstatedir=/var  --sysconfdir=/etc --with-sanlock

执行这步可能会有依赖错误，需要根据错误安装相关的依赖包  
如下面的error1 error2等  

## 编译 ##

通过make命令执行编译  

    make -j 4

## 安装 ##


    make install

如果安装后立即使用libvirt这些程序库时，遇到找不到对应库文件的错误提示  
这时可能需要运行 ldconfig 等工具来更新刚才安装的共享库。

### error1 ###

error: You must install the libyajl library & headers to compile libvirt  
直接yum安装不生效，configure libvirt的时候老是报上述错误  
从源码安装libyajl  

    git clone git://github.com/lloyd/yajl  
    cd yajl  
    ./configure
    make
    make install

源码安装libyajl的时候报错，缺失Cmake下面安装cmake  

#### 安装cmake ####

到http://www.cmake.org/cmake/resources/software.html这下载tar包  
解压，执行  

    ./bootstrap
    make
    make install

如果报错Cannot find appropriate C++ compiler on this system.  
则需要  

    yum install gcc-c++ 
    
### 继续安装libyajl ###

到libyajl目录下执行以下命令，安装libyajl

    ./configure  
    make  
    make install  

### error2 ###

配置libvirt的时候又报如下错误
configure: error: You must install device-mapper-devel/libdevmapper >= 1.0.0 to compile libvirt  
执行  

    yum install device-mapper-devel

### error3 ###

configure: error: You must install the pciaccess module to build with udev  
执行

    yum install libpciaccess-devel

### error4 ###

configure: error: libnl-devel >= 1.1 is required for macvtap support  

执行

    yum install libnl-devel

### error5 ###

configure: error: You must install the libsanlock_client library & headers to compile libvirt  

执行  

    yum install sanlock*
    

