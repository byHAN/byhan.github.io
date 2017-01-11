---
layout: post
title: ceilometer对接postgresql
category: 技术
tags: 
keywords: 
description: 
---

### 背景 ###

本文指导将ceilometer后端存储切换为postgresql

### 步骤 ###

1.登录数据库节点

创建用户，创建数据库，给数据库赋权限

    sudo su postgres
    psql
    create user ceilometer with password 'Abc12345';
    create database ceilometer;
    grant all privileges on database ceilometer to ceilometer;
    \q
    exit

注：现网环境可能已经有对应的数据库和用户名密码了，可以酌情忽略对应步骤

2.登录控制节点，修改ceilometer.conf中对接数据库部分  
将connection 替换为如下：

    connection = postgresql://ceilometer:Abc12345@10.89.151.10/ceilometer?client_encoding=utf8

注1：使用用户ceilometer,密码Abc12345对接，数据库为ceilometer，需根据实际情况替换  
注2：ip需要替换为现网具体数据库ip  
注3：所以控制节点的ceilometer.conf都需同步修改

3.同步数据库  
在controller上执行  
ceilometer-dbsync

执行完毕后，可以登录数据库节点查询数据表生成情况  
![](http://i.imgur.com/pkoVSDc.png)

4.重启ceilometer-api ceilometer-collector服务

### 验证 ###

1.登录数据库节点，查看数据是否增长  
![](http://i.imgur.com/77TaOxy.png)  
![](http://i.imgur.com/gHjktBa.png)



2.登录控制节点，通过命令行查询数据情况  
注意数据库是否最新  
![](http://i.imgur.com/ToHJSV5.png)  
![](http://i.imgur.com/Hnlu17s.png)
