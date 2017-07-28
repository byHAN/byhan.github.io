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