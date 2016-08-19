---
layout: post
title: gnocchi配置指南
category: 技术
tags: 虚拟化层
keywords: 
description: 
---


## 预置 ##


#### 创建数据库 ####

    mysql -u root -p
    
    CREATE DATABASE gnocchi;
     
    GRANT ALL PRIVILEGES ON  gnocchi.* TO 'gnocchi'@'localhost' IDENTIFIED BY 'Abc12345';
     
    GRANT ALL PRIVILEGES ON  gnocchi.* TO 'gnocchi'@'%' IDENTIFIED BY 'Abc12345';

#### 创建gnocchi用户 ####

    openstack user create --domain default --password-prompt gnocchi

![](http://i.imgur.com/brU0SN5.png)

#### 给gnocchi用户赋admin角色 ####

    openstack role add --project service --user gnocchi admin

#### 创建服务 ####

    openstack service create --name gnocchi --description "OpenStack Resource metering as a Service" metric

![](http://i.imgur.com/AikAV5p.png)

#### 创建endpoint ####

注意：
如果需要HA，则需要HAproxy配置
并把这里的Ip在实际环境中替换为haproxy的IP

    openstack endpoint create --region RegionOne metric public http://192.168.2.203:8041

    openstack endpoint create --region RegionOne metric internal http://192.168.2.203:8041

    openstack endpoint create --region RegionOne metric admin http://192.168.2.203:8041


## 安装gnocchi的包 ##

#### gnocchi ####
pip install -e .
pip install -e .[mysql,file]
(官网给出的是这样)
（也可以使用python setup.py install）


####gnocchi-client####

安装客户端gnocchiclient
python setup.py install

**注意**  
请不要忘记安装gnocchiclient

## 配置gnocchi ##
 
####创建相关目录####

如果已经有以下路径，可忽略

      mkdir /var/lib/gnocchi
      mkdir /var/log/gnocchi
      
#### 配置文件 ####

将对于配置文件放置到/etc/gnocchi目录下
（api-paste.ini，gnocchi.conf，policy.json ）

## 初始化数据库 ##

gnocchi-upgrade

验证方法如下：
![](http://i.imgur.com/ETjjiCt.png)

## 启动服务 ##

systemctl start openstack-gnocchi-api.service openstack-gnocchi-metricd.service

## 设置gnocchi ##

    gnocchi archive-policy create -d granularity:5m,points:12 -d granularity:1h,points:24 -d granularity:1d,points:30 low

## 配置ceilometer ##

将dispatchers配置为gnocchi
在dispatcher_gnocchi制定uri

    
    meter_dispatchers = gnocchi
    event_dispatchers = gnocchi
    [dispatcher_gnocchi]
    filter_service_activity = false
    url = http://192.168.2.203:8041
    archive_policy = low

将/etc/ceilometer/ceilometer.conf以上内容修改完成(注意，如有需要url的地址修改为haproxy的ip)  
重启ceilometer-collector服务  

    systemctl restart openstack-ceilometer-collector.service

## 验证 ##

使用gnocchi相关的命令查询到指标，说明配置正确  
![](http://i.imgur.com/Di6Pw3r.png)

**注意：**
使用gnocchi后，原来的ceilometer接口被废弃。

## 问题解决指引##
error1:
![](http://i.imgur.com/vG7MxvF.png)
pip install osc-lib


## 参考文档 ##

[http://blog.sina.com.cn/s/blog_6de3aa8a0102vgp1.html](http://blog.sina.com.cn/s/blog_6de3aa8a0102vgp1.html)  
[http://gnocchi.xyz](http://gnocchi.xyz/)