---
layout: post
title: 通过原始Centos ISO来定制自己的ISO
category: 技术
tags: 
keywords: 
description: 
---

本文转自[肖丁博客](http://ssdxiao.github.io/linux/2015/11/17/Build-My-OS-with-Centos.html)

### 背景 ###

通过本方法可以基于Centos原始镜像，裁剪或者增加rpm包，从而制作自己的镜像。  

### 步骤 ###

#### 1.准备 ####

下载centos7的镜像 mount 镜像到本地


    mount CentOS-7.0-1406-x86_64-DVD .iso /mnt  


因为镜像本身是只读的，所以需要将镜像cp出来

    cp /mnt/* /home/myiso/*  
    cp /mnt/.treeinfo /home/myiso/  
    cp /mnt/.discinfo /home/mysio  

修改rpm包并createrepo 在repodata目录下会看到一个xml文件，该文件主要定制了安装模式，以及每一种模式下需要安装的rpm包。  
如果要定制自己的ISO，主要就是在这里进行修改。

复制原来的2bc0054a9f0f4cd3d2806d983edbe3d0dfc484d9f275d12be79eb67a040ba942-c7-x86_64-comps.xml  
为c7-x86_64-comps.xml,然后创建仓库

    rm repodata/*comps.xml.gz -rf  
    mv repodata/*c7-x86_64-comps.xml repodata/c7-x86_64-comps.xml  
    createrepo -g repodata/c7-x86_64-comps.xml  ./

修改.treeinfo中的对应校验值使用sha256sum

    sha256sum repodata/repomd.xml
    
#### 2.打包 ####

    mkisofs -J -R -T -v -V "CentOS 7 x86_64" -boot-info-table  -no-emul-boot -boot-load-size 4 -b isolinux/isolinux.bin -c isolinux/boot.cat -o  ../CentOS-DevOps.iso ./
    
#### 3.添加md5的校验 ####

    implantisomd5 CentOS-DevOps.iso