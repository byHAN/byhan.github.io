---
layout: post
title: gnocchi配置及与ceilometer对接指南
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

## 背景介绍 ##

![](http://i.imgur.com/NWW2RPp.png)  
ceilometer中数据是通过collector进程放置到数据库中的。  
一开始数据库是用的mongodb,由于mongo会有很明显的性能问题。  
社区逐渐废弃使用mongo,开始推广gnocchi  

原理：
通过gnocchi的处理，将采集的指标分离。索引部分可以存到mysql中，数据部分存储到文件中。  

还是上图:  
在使用mongodb的时候，storage abstration layer=mongo driver.  
在使用gnocchi的时候，storage abstration layer=gnocchi.

## 预置 ##

#### 创建数据库 ####

登陆数据库节点，为gnocchi创建数据库。

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

#### gnocchi从源码安装 ####

源码安装2.0.2版本
python setup.py install


####gnocchi-client从源码安装####

安装客户端gnocchiclient2.2.0
python setup.py install

**注意**  
请不要忘记安装gnocchiclient

## 配置gnocchi ##
 
####创建相关目录####

如果已经有以下路径，可忽略

      mkdir /var/lib/gnocchi
      mkdir /var/log/gnocchi
      

#### 配置ceph相关 ####

1、在controller节点安装ceph-common包。  
如果openstack+ceph环境已经搭建完毕的话，该包就已经安装好了。  

2、给gnocchi创建一个专用的ceph pool，用来存放计量数据。（ceph mon node）  
ceph osd pool create gnocchi 128 128  
gnocchi pool的pg_num需要根据实际的ceph环境确定。  

3、给gnocchi创建一个ceph用户。  
ceph auth get-or-create client.gnocchi mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=gnocchi'  

4、保存keyring文件。  
ceph auth get-or-create client.gnocchi | tee ceph.client.gnocchi.keyring  

5、拷贝gnocchi的keyring文件ceph.client.gnocchi.keyring到所有controller节点的/etc/ceph目录下。  

6、获取gnocchi用户的ceph key，用于gnocchi.conf中的ceph_secret配置项。  
[root@hx11 ceph_cluster]# ceph auth get-key client.gnocchi  
AQAvAr1XN4BtDRAAR8O6ubQMZhCov26b8/f7hg==  


#### 配置文件 ####

修改对应的配置项
将对于配置文件放置到/etc/gnocchi目录下
（api-paste.ini，gnocchi.conf，policy.json ）

## 初始化数据库 ##

gnocchi-upgrade

验证方法如下：
![](http://i.imgur.com/ETjjiCt.png)

## 启动服务 ##

systemctl start openstack-gnocchi-api.service openstack-gnocchi-metricd.service

### bin文件处理 ###

拷贝gnocchi文件到/usr	/bin目录下，修改权限为

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