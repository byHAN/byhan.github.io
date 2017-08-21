---
layout: post
title: ubuntu中kdump的配置
category: 技术
tags: openstack
keywords: multipath call failed exit 1
description: 
---

sudo apt-get install linux-crashdump  
（具体需要先将相关软件加入源中）

这里选择yes  
![](http://i.imgur.com/cNxSOcQ.png)


vim /etc/default/kdump-tools  
将配置USE_KDUMP修改为1

![](http://i.imgur.com/OOMUn3s.png)


修改/etc/sysctl.d/10-magic-sysrq.conf  
修改配置项  
kernel.sysrq = 1


#### 验证kdump####

1. echo c> /proc/sysrq-trigger  
2. 可以配上串口，从bmc查看触发的kdump  
![](http://i.imgur.com/vHVqiwL.png)  

重启主机后  
本地也可以这样查看确认  
![](http://i.imgur.com/IjiGD6j.png)

生成的转储文件如下这个样子：  
![](http://i.imgur.com/M9ZlloK.png)


### 定位 ###

这里用crash定位 


首先需要安装从源安装systemtap  

    sudo apt-get install systemtap

查看内核版本

    uname -a
    
![](http://i.imgur.com/TiWUXrk.png)

到这里下载对应版本的debug-info（如果没有对应的版本，需要从网上找到对应的发行版本）  

    http://ddebs.ubuntu.com/pool/main/l/linux/
    
安装deb包

    dpkg -i linux-image-3.19.0-25-generic-dbgsym_3.19.0-25.26~14.04.1_amd64.ddeb


使用crash调试

    crash /usr/lib/debug/boot/vmlinux-3.19.0-25-generic /var/crash/201708160959/dump.201708160959
 

