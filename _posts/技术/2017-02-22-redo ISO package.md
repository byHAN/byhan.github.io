---
layout: post
title: 通过原始Centos ISO来定制自己的ISO
category: 技术
tags: 
keywords: 
description: 
---

本文修订自[肖丁博客](http://ssdxiao.github.io/linux/2015/11/17/Build-My-OS-with-Centos.html)

### 背景 ###

通过本方法可以基于Centos原始mini版本镜像，裁剪或者增加rpm包，从而制作自己的镜像。  

### 步骤 ###

#### 1.准备 ####

下载centos7的镜像 mount 镜像到本地


    mount CentOS-7.0-1406-x86_64-DVD .iso /mnt  


因为镜像本身是只读的，所以需要将镜像cp出来

    cp -r /mnt/* /home/myiso/  
    cp -r /mnt/.treeinfo /home/myiso/  
    cp -r /mnt/.discinfo /home/mysio  

修改rpm包并createrepo  
在repodata目录下会看到一个xml文件，该文件主要定制了安装模式，以及每一种模式下需要安装的rpm包。  
如果要定制自己的ISO，主要就是在这里进行修改。

复制原来的c30db98d87c9664d3e52acad6596f6968b4a2c6974c80d119137a804c15cdf86-c7-minimal-x86_64-comps.xml  
为c7-x86_64-comps.xml,然后创建仓库

    rm repodata/*comps.xml.gz -rf  
    mv repodata/*c7-x86_64-comps.xml repodata/c7-x86_64-comps.xml  
    
如果需要修改安装包  
需要在c7-x86_64-comps.xml中做修改  
比如下图中，在core这个group中增加了相应的包（放在core中是为了使得默认安装，不用交互，新建group也行）  
![](http://i.imgur.com/RLPlTsC.png)

拷贝所需的rpm包到/home/myiso/Packages目录下（包含安装包和所有依赖的包）

创建本地仓库  

    createrepo -g repodata/c7-x86_64-comps.xml  ./

修改.treeinfo中的对应校验值使用sha256sum

    sha256sum repodata/repomd.xml
    
#### 2.打包 ####

注意我们这里修改了包名称（后面加了fh）  
需要对应的在/home/myiso/isolinux/isolinux.cfg的启动命令  
![](http://i.imgur.com/dcKFi2O.png)
负责打出来的ISO在安装的时候，由于mount不上，导致无法安装

    mkisofs -J -R -T -v -V "CentOS 7 x86_64 fh" -boot-info-table  -no-emul-boot -boot-load-size 4 -b isolinux/isolinux.bin -c isolinux/boot.cat -o  ../CentOS-DevOps.iso ./


    
#### 3.添加md5的校验 ####

    implantisomd5 CentOS-DevOps.iso