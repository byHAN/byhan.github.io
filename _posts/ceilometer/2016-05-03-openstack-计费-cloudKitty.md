---
layout: post
title: ERROR 1045(28000) : openstack 计费 cloudKitty
category: 技术
tags: openstack
keywords: 
description: 
---

计费服务一般分为三个层次：

    
- 计量（Metering）
    获取到资源的指标信息，比如虚拟机的启动时间，停止时间，网卡流量等信息。
    为上层的计费服务提供数据来源，在openstack中计量由ceilomete来处理。
    详细见另外一篇博文
    
- **计费（Rating）详细见本文**
    根据收费的规则生成对应的话单，比如按照虚拟机的运行时长计费，根据启动时间和停止时间换算出费用。
    为上层的收费服务提供对应的话单，在openstack中计费有模块CloudKitty来处理。
    
- 收费（Billing）
    用户账号充值，对应服务的控制等。  


# 背景知识 #

计费模块在整个服务层次中处于承上启下的位置。  
向下使用计量产生的指标（sample），向上为上层的收费模块提供服务。  

由于ceilometer只专注于计量，而云服务（不管是公有云还是私有云）如何收费是不可或缺的部分。这就从方案的完整性上造成一定割裂，为了弥补openstack的计量和收费之间的鸿沟，提出了cloudkitty服务。  

# 使用概览 #

眼见为实，让我们先从Horizon上只管感受下cloudkitty服务。
cloudkitty整合到horizon目前分为三个部分，分别在Admin侧，Project侧，创建虚拟机页面。

### Admin侧（系统侧） ###

系统侧主要是指定一系列收费的策略。点开Admin侧，发现将cloudkitty整合到Horizon后，会相应的增加一个Rating组。
让我们先聚焦到其中的Rating Module，发现有noop|hashmap|pyscripts三个东西，这些东西在cloudkitty中叫processors，也就是处理器，他们的职责是处理从计量模块收集上来的指标。

![](http://i.imgur.com/pEiTDXD.png)

#### noop ####  

仅是自测模块，忽略
#### hashmap（重点模块） ####

通过类似hashmap的结构，一层层的将各个服务的metadata和cloudkitty进行关联。



- 最顶层是service,表示目前可以计费的项目，包括：
compute  
image  
volume  
network.bw.in  
network.bw.out  
network.floating  
我们这里创建了compute和volume服务
![](http://i.imgur.com/yJ6KaNt.png)

- Hashmp Service  
可以在service建立一系列的field，与对应服务中的meatadata进行对应。以compute服务为例,我们在compute这个service下新增两个field：flavor和image_id
意味着对flavor和image进行计费  
![](http://i.imgur.com/TiUet56.png)

- Hashmp field
可以为field指定mapping,也就是这个field下的资源如何收费。  
flavor为例，新建两个field mapping，值分别为m1.tiny和m1.small，意味针对m1.tiny和m1.samll指定收费标准。  
![](http://i.imgur.com/x3wvn5n.png)


#### pyscripts ####

以自己指定脚本的形势处理获取到的指标。

### project侧 ###

租户侧主要给出当前租户计费的总值

![](http://i.imgur.com/CtY1wet.png)

并能分析给出各个计费值的统计信息（本人环境由于信息量过大，这块崩溃了，下图为盗图）

![](http://i.imgur.com/NMRErz7.png)

### 创建虚拟机页签 ###

cloudkitty将相关服务整合到创建虚拟机的流程中，可以在创建时候直观体现费用信息。
上文中，我们已经把模板：m1.tiny和镜像：cirros_raw进行了收费量化
当勾选到这两个资源项时，会展现出收费信息，起到一个预览的作用。

![](http://i.imgur.com/PYQhupQ.png)


# 优缺点 #

### 优点 ###

- 能够做到基本资源的计费
- 极大程度的使用了openstack已有的基础组件
- 可与ceilometer进行对接
- 可与horizon整合
- 代码实现模块化
- 计费做到了租户粒度

### 缺点 ###

- 易用性不好，使用不够贴近用户。
	- 计费策略是在专门的计费页面，建立资源项与计费的关系，而不是直接整合到业务中，增大了管理员的工作量
	- 好多地方是隐式约定的，无相应说明，给使用者造成误解。
- 无法做到按照用户计费。
- 计费的产生周期，无法做到各个资源迥异。
- 项目热度不足。

# 核心总结 #

### 逻辑功能 ###

- input data sources (collectors)  数据收集，默认是从ceilometer收集。
- rating policies (rating pipeline) 指定计费策略， 如hashmap这种通过指定meatadata对应项收费标准来
- output storage (storage) 数据存储到哪，一般配置到数据库节点的MYSQL中。
- output file format (writers, used to generate reports)

![](http://i.imgur.com/lNXa0dz.png)

### 数据模型 ###

#### group ####

定义了一直规则，同组的规则处理的时候一起处理。逻辑概念。

#### service ####

cloudkitty的服务单元，分别处理不同的模块信息，如compute等，详细见上面

#### field ####

用来和资源的metadata进行匹配。

#### mapping ####

可以和service或者field进行关联
指定metadata的价格及其收费类型(flat或者rate)

rate:费率累乘。
flat:当其他费率小于当前值的时候，使用当前值，否则使用其他费率。

#### threshold ####

可以和service或者field进行关联  
可以定义不同阈值的收费策略，比如超过一定阈值，收费减少，实现折扣功能。

# 安装部署 #

大体流程可以参考[官方文档](http://docs.openstack.org/developer/cloudkitty/installation.html)。

将本人安装过程及其遇到的坑梳理如下（由于项目还在进化中，这里依照2015.12最新源码为准）。

### 安装后台服务 ###

本人是在Ubuntu 14平台，采用源码方式进行的安装。

    git clone git://git.openstack.org/openstack/cloudkitty
    cd cloudkitty
    python setup.py install

以上步骤执行完成，安装了如下服务：

    cloudkitty-api: API service
    cloudkitty-processor: Processing service (collecting and rating)
    cloudkitty-dbsync: Tool to create and upgrade the database schema
    cloudkitty-storage-init: Tool to initiate the storage backend
    cloudkitty-writer: Reporting tool

处理配置文件：

    mkdir /etc/cloudkitty
    cp etc/cloudkitty/cloudkitty.conf.sample /etc/cloudkitty/cloudkitty.conf
    cp etc/cloudkitty/policy.json /etc/cloudkitty

### 配置cloudkitty ###

打开/etc/cloudkitty/cloudkitty.conf参考本人配置项进行配置。
按照官方文档的配置项进行配置特点配置会出错，耗费大量时间去debug出来的。
本人的配置项目请参考这里cloudkitty（当前cloudkitty 为鉴权，可能个别配置项不准确，请审视。）

### 设置数据存储相关 ###

登录数据库节点，创建cloudkitty所需的数据库，运行脚本创建所需的表。
执行完可以登录数据库节点，user cloudkitty然后show tables;进行初步验证。

![](http://i.imgur.com/7LRJFgL.png)

### 设置keystone ###

    openstack user create  cloudkitty --password 密码为用户添加角色
    openstack role add --project service --user cloudkitty admin创建角色（一定要创建，代码中硬编码，必须要有rating角色）
    openstack role create rating为用户添加rating角色
    openstack role add --project service --user cloudkitty rating创建cloudkitty服务对应endpoint(ip请酌情替换)
    openstack service create --name CloudKitty rating
    openstack endpoint create --region-id RegionOne \
    --publicurl http://192.168.1.170:8888 \
    --adminurl http://192.168.1.170:8888 \
    --internalurl http://192.168.1.170:8888  rating

### 启动cloudkitty服务 ###

本人在服务启动的时候，遇到较多问题，多是官方参考配置项不对引入，已跟源码修正，请参考本人配置项 

    cloudkitty-api --config-file /etc/cloudkitty/cloudkitty.conf
    cloudkitty-processor --config-file /etc/cloudkitty/cloudkitty.conf附（将服务整合成守卫进程）

### 整合horizon ###

下载dashboard源码并安装 

    git clone git://git.openstack.org/openstack/cloudkitty-dashboard
    cd cloudkitty-dashboard
    python setup.py install

 确认horizon的包路径，官方文档给的参考方法如下
PY_PACKAGES_PATH=`pip --version | cut -d' ' -f4`
然而并没有什么卵用，本人环境生效的路径不是这，安装完一直没有对应的页签。
请前往etc/apache2/sites-available 中的horizon.conf 中的这个设置： <Directory /opt/horizon/>确认
本人环境horizon生效目录为/opt/horizon/

将cloudkitty的文件与horizon文件目录做一个软连接，使得portal内嵌到horizon中

    ln -s $PY_PACKAGES_PATH/cloudkittydashboard/enabled/_[0-9]*.py \ /opt/horizon/openstack_dashboard/enabled/

# 源码分析 #

### 服务入口 ###

按照租户依次生成一个worker并调用run方法处理  

![](http://i.imgur.com/bJkp4GE.png)

### _collect方法 ###

调用_collect方法进行数据的收集，然后以此调用processor对数据进行处理

![](http://i.imgur.com/OurMsHn.png)

#### _collector数据收集 ####

collector通过setup.config的cloudkitty.collector.backends字段指定

![](http://i.imgur.com/fWO70DV.png)

retrieve方法，通过反射，根据服务不同调用调用到ceilometer.py中的get_**方法

![](http://i.imgur.com/ihIbcoc.png)

各个方法不同，但大体都是调用ceilometer的statistics方法获取到资源情况，然后调用ceilometer的resource获取到对应的meta信息，封装成对象返回。  
![](http://i.imgur.com/WeOQB6B.png)  
![](http://i.imgur.com/KBxbuf9.png)

#### processor ####
不同的processor有不同的处理方法，这里以hashmap为例，可见以依次理了service的mapping，field的mapping，然后在数据中添加了计费信息  
![](http://i.imgur.com/Qd8LrED.png)  
![](http://i.imgur.com/myskuFP.png)

#### 计费信息后续处理 ####

默认是保存到数据表rated_data_frames中了  
![](http://i.imgur.com/OgRLkqL.png)


# 可优化方向 #

- 按照用户计费  
  当前逻辑粒度为租户，暂时无法做到用户级别 计费。
- 计费周期
	- 当前所有资源的计费周期统一，一改俱改，无法做到按照资源划分。按照最小粒度，会产生大量无用信息。
	- 计费周期不灵活。计费方式分多钟，按照小时，按照月，按照年等，当前无法满足。  
    ![](http://i.imgur.com/Cm7gxh7.png)


- 规格变更
- 齐所需计费内容
