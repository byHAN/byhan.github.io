---
layout: post
title: openstack/ceilometer/gnocchi之Grafana简介
category: 技术
tags: openstack
keywords: 
description: 
---

#### # 资源链接（部分资源会被墙，或者周期性不稳定，请自备梯子） #####
官网：[http://grafana.org/](http://grafana.org/)  
入门视频：[https://www.youtube.com/watch?v=sKNZMtoSHN4&index=7&list=PLDGkOdUX1Ujo3wHw9-z5Vo12YLqXRjzg2](https://www.youtube.com/watch?v=sKNZMtoSHN4&index=7&list=PLDGkOdUX1Ujo3wHw9-z5Vo12YLqXRjzg2)  
源码：[https://github.com/grafana/grafana.git](https://github.com/grafana/grafana.git)  
插件源码：[https://github.com/grafana/grafana-plugins.git](https://github.com/grafana/grafana-plugins.git)  
gnocchi对接介绍：[https://julien.danjou.info/blog/2015/openstack-gnocchi-grafana](https://julien.danjou.info/blog/2015/openstack-gnocchi-grafana)  
gnocchi对接配置：[http://gnocchi.xyz/grafana.html](http://gnocchi.xyz/grafana.html)  


# 简介 #

grafana是一个前端界面项目，可以用来展示时间序列的metric数据。  
它很火，很多人用它外加InfluxDB打造监控系统。  
看github上得到的星星，如下图  

![](http://i.imgur.com/DkyHIMZ.png)

另一篇博文（openstack/ceilometer/gnocchi杂谈）已经讲到  
使用gnocchi的情况下，把ceilometer采集到的数据引入到gnocchi。  
如何展示是这些数据呢？  
最终决定使用Grafana，效果如下图  

![](http://i.imgur.com/HOZMWfQ.png)

# 安装 #

本人是devstack中安装的  
原则上只需在local.conf中配置enable_service gnocchi-grafana即可  
但是实验环境没有翻墙，资源被墙，故通过修改安装脚本实现安装  
先下载好包，放到指定目录，然后修改gnocchi里的devstack安装脚本plugin.sh即可。  

# 配置 #

需要在gnocchi和keystone中均正确的配置cors  
否则，会导致keystone异常。  
本人在事先安装好的环境中，追加grafana安装，一直导致keystone异常，卡了好久。  

# 与gnocchi对接 #

gnocchi在grafana看来只是一个数据源，在grafana中叫Data Source，如下图  

![](http://i.imgur.com/Epzvm1R.png)  

从实现的角度讲，grafana有个grafana-plugins项目  
用户想将自己的数据拿来给grafana展示的话，只需在参照grafana-plugins里，实现相应的逻辑即可。  
gnocchi已派一个成员实现其功能（如果满足要求，此处代码会回合至grafana主干，据说gnocchi要回合）  

![](http://i.imgur.com/WVLSqwV.png)  

上面说了有关配置的背景知识  
下面看下如何使用：  
增加一个Data Sources，类型拉选Gnocchi  
有两种方式proxy和direct  
有谁知道token如何获取，又不涉及安全问题的，欢迎告知  

![](http://i.imgur.com/djUoQmH.png)  

配置完上述就可以在Dashboard中增加画板，查看相关数据了。  

# 问题 #

本人使用中发现几处问题  