---
layout: post
title: ubuntu中kdump和串口配置
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

### 串口配置 ###

创建 /etc/init/ttyS0.conf文件  

    # ttyS0 - getty
    #
    # This service maintains a getty on ttyS0 from the point the system is
    # started until it is shut down again.
    
    start on stopped rc or RUNLEVEL=[2345]
    stop on runlevel [!2345]
    
    respawn
    exec /sbin/getty -L 115200 ttyS0 vt102

启动getty  
sudo start ttyS0

查看/boot/grub/grub.cfg中对于项目是否有  
console=tty0 console=ttyS0,115200n8

#### 验证kdump####

1. echo c> /proc/sysrq-trigger 
2. bmc触发kdump


#### 验证串口 ####

找一个终端使用如下命令启动串口监听

ipmitool -I lanplus -H 192.168.202.220 -U root -P Huawei12#$ sol activate

—H是监测的机器BMC地址
-U是用户名-p是密码

![](http://i.imgur.com/sxPfsYG.png)
