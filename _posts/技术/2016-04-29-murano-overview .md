---
layout: post
title: murano 概览
category: 技术
tags: 随感
keywords: 
description: 
---

# openstack/murano 概览 #

##概述##
murano是openstack的一个PAAS层项目,旨在简化应用部署。
 它使用openstack提供的IAAS层服务（nova、neutron、heat）将创建虚拟机部署应用整合起来。

在管理员准备好镜像及其服务包的基础上，使用者可以一键式部署心仪的应用。
从此，将云使用者从创建虚拟机到安装应用的轮回中释放出来
极大的简化云使用者的，提高了服务体验。
比如说有客户需要一个安装了apache的虚拟机，现在需要做的就是点一下按钮:

![](http://i.imgur.com/7Zd7t6B.png)

## 名词解释 ##
Categories:就是一系列标签，支持app检索

packages：安装包，用户可以从这里导入安装包

Application：应用，安装包中可有多个应用，导入packages后，Application会增加对于的应用

Environments：相关联的应用及其网络组成一个Environments，部署的话是部署的Environments，在Environments中应用叫组件（component）

## 安装部署 ##

### 测试 ###
devstack中部署murano，在local.conf中追加如下两行即可
    
    enable_plugin murano git://git.openstack.org/openstack/murano
    
    MURANO_APPS=io.murano.apps.apache.Tomcat,io.murano.apps.Guacamole
   

另外，为使得你能够从外面访问到devstack创建出来的虚拟机，需要进行如下配置
 （具体ip自行调配，比如我的网段是192.168.1）

    FLOATING_RANGE=192.168.1.0/24
    Q_FLOATING_ALLOCATION_POOL=start=192.168.1.172,end=192.168.1.182
    PUBLIC_NETWORK_GATEWAY=192.168.1.1


### 商用 ###
请[参考这里](http://murano.readthedocs.org/en/stable-liberty/install/manual.html) （我本身只在devstac中部署过，后续部署了再补充）

## 使用简介 ##

### 上传应用 ###

####(1)Murano>Manage>Package Definitions  ####

![](http://i.imgur.com/xlpQ0bP.png)


#### (2)选择方式 ####

1. **文件:**可以安装murano的要求自己规划安装包
用Murano Programming Language (MuranoPL,也就是yaml和yaql)具体[看这里](http://murano.readthedocs.io/en/stable-liberty/draft/appdev-guide/step_by_step.html)和[这里](http://murano.readthedocs.io/en/stable-liberty/draft/appdev-guide/murano_pl.html)。
murano也支持heat模板进行转化，具体如何转换[看这里](http://murano.readthedocs.io/en/stable-liberty/draft/appdev-guide/hot_packages.html)
2. **Repository:**可以自己搭建仓库
3. **URL:**可以使用官方安装包（地址是[http://apps.openstack.org/](http://apps.openstack.org/) 间歇性抽风，需翻墙）
![](http://i.imgur.com/tnzBdAO.png)
4. 点击下一步


###部署应用###

#### (1)Murano > Application Catalog > Applications ####
找到需要部署的应用，点击Quick Deploy

![](http://i.imgur.com/IQjMCo6.png)

#### (2)配置应用，起个名儿吧 ####

![](http://i.imgur.com/VdqObRV.png)

#### (3)勾选镜像 ####


1. 镜像：上传应用的时候，如果使用的是官方源的话，会走动上传个镜像到glance
2. key pair ：注意这里一定要设置keypair，应用部署出来想登陆虚拟机的话，只能通过这里了。
![](http://i.imgur.com/gGblFY8.png)

#### (4)部署应用 ####

点击部署，这里就开始部署了，运气好的话会部署成功
 运气不好的话，等3600秒就会报失败了
![](http://i.imgur.com/NvS8Fur.png)


（不行，这一节要加一段吐槽）

 这里点击完部署后，会通过heat创建个satack，会创建网络，会创建安全组
 （注：创建完网络后，需要迅速查看，补充上DNS，我这里使用官方包，默认创建出来的
 environment创建的网络子网中默认的没有dns）

 然后就会创建虚拟机，虚拟机创建完成后，会按照相应的规划部署应用,这个阶段可以使用上文的key pair登陆到虚拟机中
 在/var/murano目录下回看到一系列的plans，就是你的安装规划了。

 然后就是应用部署了（到这，听天由命吧）：
 
 我在部署的时候，我部署了三种应用，只有数据库的部署成功
 一次由于上述的dns问题导致无法加载源，进而murano-agent加载失败
 一次由于墙的原因，导致无法下载ruby的包安装失败

 体会：如果要使用murano,需要强大的运维能力：

（1）需要管理员根据实际情况自己构建environment，比如网络相关设置等,避免网络不通

（2）需要提前将必备的包下载好，自己做仓库，否则会导致安装时候下不了包，部署失败

（3）或者把应用提前在虚拟机中装好，然后导入到murano,具体看这里

#### 删除应用 ####

kilo有个bug：
 删除应用不会释放掉资源，需要删除掉environment采才可以

# 架构 #

![](http://i.imgur.com/LzGVn50.png)

捡重要的说三点：

1. heat：资源的操控通过heat进行的，比如虚拟机网络等相关资源的创建删除，都是通过heat触发的。
2. murano-engine：干活的主体，一个自动机，通过反射等技术将用户规划的安装步骤转换为plan，指导murano-agent工作。
3. murano-agent：镜像中安装，最终运行在客户机中，接受murano-agent的指令，在客户机中按照规划执行安装工作。
4. RabbitMQ：给murano-ageent发送的消息，是通过消息队列传递的

