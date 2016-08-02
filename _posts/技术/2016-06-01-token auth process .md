---
layout: post
title: token auth process 
category: 技术
tags: keystone
keywords: 
description: 
---

## 认证 ##

Authentication，认证，用来确定你是谁  
你说你是谁，我需要鉴定一下  
也就是你提供证据，keystone来认证那个哲学问题  
“你妈是你妈”

认证方式有多种，token,公钥，数字证书等等  
这里重点分析基于token的keystone服务认证流程

## 一句话总结 ##

有请求的时候，从header中找到token  
然后校验token合法性  
非法：抛出异常返回401
合法：业务继续向下流动

## 代码分析 ##

wsgi服务中可以指定token的认证方式  
详见[之前的博文](http://www.hanbaoying.com/2016/05/12/openstack-restful%28nova-api%29.html)

比如指定在nova.conf中指定auth_strategy为keystone认证  
![](http://i.imgur.com/xAUj2vv.png)  

也就是说，调用比如nova的api接口的时候，会触发如下的认证过程  

之前认证的代码是在keystoneclient中  
现在代码已整理至keystonemiddleware中间件项目中  
![](http://i.imgur.com/rbkngKH.png)  

让我们打开代码，会发现实际上起作用的类是AuthProtocol  
![](http://i.imgur.com/aATsb7S.png)  

类图如下  
![](http://i.imgur.com/uM2uZ90.png)  

通过之前的wsgi知识可以知道  
消息在中间件中传递的时候是调用的中间件的call方法  
这里是调用了父类BaseAuthProtocol的call方法  
![](http://i.imgur.com/iAHkiFX.png)  

然后调用到AuthProtocol的process_request方法  
1.去除header中已有的认证信息，防止token伪造  
2.初始化_token_cache，后续用来缓存token  
3.调用父类BaseAuthProtocol的process_request方法  
   3.1这块后面需要仔细分析
4.后续处理，认证不通过抛出异常，通过在请求体中塞入认证信息等  
![](http://i.imgur.com/957ni5n.png)  

下面仔细看下BaseAuthProtocol的process_request方法  
根据header中塞入的内容不同分别走走不同的流程  

#### X-Auth-Token和X-Storage-Token ####

注：X-Storage-Token is supported for swift/cloud files and for legacy Rackspace use

![](http://i.imgur.com/mdViVjX.png)

#### X-Service-Token ####

如果带了X-Service-Token，在上面的X-Auth-Token和X-Storage-Token认证通过后  
会进行X-Service-Token认证  
如果失败，则会返回认证失败
（在不考虑delay_auth_decision的情况下）


![](http://i.imgur.com/TRBCE2g.png)

可见上面两种，流程大致相同，我们一个个分析

#### _do_fetch_token ####

需要注意的调用fetch_token后，会使用access去构造一个对象  
access在Kilo版本中是在keystoneclient中  
新版本中已经挪到keystoneauth  
![](http://i.imgur.com/gwU8iYb.png)

重点关注fetch_token  
这里判断token是否已经缓存  
1.如果缓存了，判断当前token是否被撤销了（通过revoked.pem）  
![](http://i.imgur.com/f6InpUK.png)  

2.如果没有缓存，则先使用离线方式校验  
3.再不行，就通过keystoneclient调用接口校验  
![](http://i.imgur.com/kKO3NBa.png)  

下面先看下离线校验方式，也就是_validate_offline  
可见先根据token类型不同进行解压，然后_cms_verify进行处理
![](http://i.imgur.com/coyOnrX.png)  

这里补充uuid和pkiz类型token的认证过程  
![](http://i.imgur.com/hwYwkKJ.png)  
![](http://i.imgur.com/DWSTPiz.png)

继续看_cms_verify  
![](http://i.imgur.com/fNYTwR9.png)  
这里cms是的keystoneclient.common下的cms.py  
仔细看下也就是通过openssl尝试通信，来验证CMS签名的合法性  

_identity_server.verify_token不多看了  
主要是使用/tokens/tokenid获取token详情

#### _validate_token ####

主要验证token是否快超期  
和  
v2版本需要auth_ref.project_id

#### _confirm_token_bind ####

根据下面的策略  
    class _BIND_MODE(object):
    DISABLED = 'disabled'
    PERMISSIVE = 'permissive'
    STRICT = 'strict'
    REQUIRED = 'required'
    KERBEROS = 'kerberos'

然后判断绑定的是否合理  

至于什么是绑定，为什么绑定，后续再补充吧
TODO

#### 参考文档 ####
[http://docs.openstack.org/developer/keystonemiddleware/middlewarearchitecture.html](http://docs.openstack.org/developer/keystonemiddleware/middlewarearchitecture.html)