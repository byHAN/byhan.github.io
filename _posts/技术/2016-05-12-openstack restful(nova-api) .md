---
layout: post
title: openstack Restful and RPC
category: 技术
tags: openstack
keywords: 
description: 
---

## 简介 ##

OpenStack的通信方式有两种，一种是基于HTTP协议的RESTFul API方式，另一种则是RPC调用。

两种通信方式的应用场景有所不同：  
RESTFul API主要用于各组件之间的通信（如nova与glance的通信）  
RPC用于同一组件中不同模块间的通信（如nova组件中nova-compute与nova-scheduler）。

## Restful ##

### 基础知识 ###

rest构建在HTTP协议之上(其实这俩都是一个哥牵头搞出来的)  

rest处理流程由HTTP请求、解析HTTP请求、发送HTTP响应等一系列活动组成  
如果使用rest需要进行上述动作，可以想象整个过程十分繁杂  
所以，将上述活动屏蔽起来，对外展现一个统一的接口（也就是wsgi,Web Server Gateway Interface）。  

这样，只要按照wsgi规范实现对应的接口，就可以使用rest服务了。  
开发者专注于开发自己的业务流程就可以了。

### WSGI ###

想使用wsgi，我们首先要明白wsgi只是一个规范  
它规定了组件，组件间的接口等

让我们先看下wsgi的三个组件：  
- Application
- Server
- client

三个组件间消息流向为：  
client发送请求给server,server转发这个请求给application.  
然后application进行一系列的操作后，将响应返送给server,server再将相应转给client.  
server只起到一个中转的作用。


#### Application ####

Application是具体干活的组件，当有请求的时候会调用对应Application处理消息。

因此，Application要能被调用，即Callable。  
换句话说，它应该是一个函数或者定义了`__call__`方法的对象，`__call__`方法的签名是这样的：

    def __call__(environ: Dict[String, Any], start_response: (String, List) -> Any)): 
        Iterable[String]

#### Server ####

如前文所述：
Server唯一的任务就是接收来自 client 的请求，然后将请求传给 application，最后将 application 的response 传递给 client

#### 简单例子 ####

    from eventlet import wsgi  
    import eventlet  
    def hello_world(env, start_response):  
        start_response('200 OK', [('Content-Type', 'text/plain')])  
        return ['Hello, World!\r\n']  
    wsgi.serve(eventlet.listen(('', 8090)), hello_world)  

本例中hello_world是一个Application  
使用eventlet.wsgi生成的server调用hello_world  
然后返回结果

#### 中间件（Middleware） ####

有些情况下，请求不会直接流向Application  
而是经历一系列的中间件处理，如下图

![](http://i.imgur.com/3qSI8Un.png)



### oenstack源码剖析 ###

#### 几个库 ####

openstack中使用了一系列的库，组合完成wsgi功能，如下：
- Paste.deploy  
  Paste Deployment是用于发现和配置WSGI appliaction和server的插件。  
  对于WSGI application，用户提供一个单独的函数（loadapp），用于从配置文件或者python egg中加载WSGI application。  
  
- webob  
  Request和Response处理。  
  简单理解为从HTTP中抽取并组装成Request对象。

- eventlet.wsgi  
  eventlet提供一个WSGI server的功能

- routes.Mapper  
  ![](http://i.imgur.com/5MAwjQm.png)  
  connect()方法，将url和controller建立链接  
  routematch（）匹配到返回一个router对象，匹配不到，返回None

### 服务启动 ###

以nova-api服务为例，在setup.cfg中可以看到nova-api服务入口如下  
![](http://i.imgur.com/O2zFC4l.png)

main方法中，主要流程为，构造出一个WSGIService对象，交付给launcher去启动  
![](http://i.imgur.com/7Jcgo6B.png)

WSGIService初始化的时候，会使用Paste.deploy.loadapp这个插件加载wsgi的配置文件（/etc/nova/api-paste.ini）  
（具体加载过程见**api-paste.ini解析**小节）  
![](http://i.imgur.com/JXIMQmW.png)  
最终是使用的Paste.deploy  
![](http://i.imgur.com/uabhFbV.png)  

然后使用加载的app作为参数，构造出wsgi.Server对象  
![](http://i.imgur.com/uYDDzK1.png)  

launcher启动的时候，会调用到wsgi.Server的start方法  
![](http://i.imgur.com/xatdvpu.png)  
也就孵化了个线程，去执行eventlet.wsgi.server方法  

至此，wsgi服务启动
服务启动了socket去监听端口（默认8774），当有http请求时候进入消息处理阶段

#### api-paste.ini解析 ####

刚刚看到加载app是通过Paste.deploy  
与Paste.deploy的交互式通过api-paste.ini  
下面我们看下解析api-paste.ini的解析过程  

    [composite:osapi_compute]  #入口会是这里 
      use = call:nova.api.openstack.urlmap:urlmap_factory #调用urlmap_factory方法，后面几行作为入参传入
      /: oscomputeversions
      /v1.1: openstack_compute_api_v2
      /v2: openstack_compute_api_v2
      /v2.1: openstack_compute_api_v21
      /v3: openstack_compute_api_v3

调用nova.api.openstack.urlmap:urlmap_factory方法，将后面四行作为参数  
urlmap_factory以此加载这几个composite，组装成urlmap字典  
![](http://i.imgur.com/ljQGDrH.png)

以openstack_compute_api_v2为例

    [composite:openstack_compute_api_v2]
    use = call:nova.api.auth:pipeline_factory
    noauth2 = compute_req_id faultwrap sizelimit noauth2 ratelimit osapi_compute_app_v2
    keystone = compute_req_id faultwrap sizelimit authtoken keystonecontext ratelimit osapi_compute_app_v2
    keystone_nolimit = compute_req_id faultwrap sizelimit authtoken keystonecontext osapi_compute_app_v2

nova.api.auth:pipeline_factory方法根据在nova.conf中配置的认证策略选择到不同的处理  
![](http://i.imgur.com/HmBhV5Y.png)  

这假设auth_strategy配置的为keystone  
则会选到下面这条处理链

    keystone = compute_req_id faultwrap sizelimit authtoken keystonecontext ratelimit osapi_compute_app_v2

![](http://i.imgur.com/9XLph8l.png)

首先会加载osapi_compute_app_v2（下一小节单独分析）

然后用前面的filter依次装饰osapi_compute_app_v2
注意是逆向，keystonecontext（ratelimit（osapi_compute_app_v2）），类似穿了多层衣服。  

消息处理的时候，再一层层脱衣服。


#### osapi_compute_app_v2加载 ####

    [app:osapi_compute_app_v2]
    paste.app_factory = nova.api.openstack.compute:APIRouter.factory

![](http://i.imgur.com/AMtkWKu.png)

调用nova.api.openstack.compute:APIRouter.factory方法  
实际上是调用的父类nova.api.openstack.APIRouter的factory方法     
也就是其__init__方法  
![](http://i.imgur.com/IfOq2Kv.png)  

- ExtensionManager初始化  
  加载nova.api.openstack.compute.legacy_v2.contrib目录下除了init外的py文件  
  每个py文件内有个同名类，这里将同名类，放置到ExtensionManager队列中  
  实现Extension的加载  

  ![](http://i.imgur.com/ELMVCzX.png)  
  调用nova.api.openstack.compute.legacy_v2.ExtensionManagerd的init方法  
  ![](http://i.imgur.com/4HdW90k.png)  
  其中，CONF.osapi_compute_extension为默认:  
  nova.api.openstack.compute.legacy_v2.contrib.standard_extensions  
  进而调用父类的_load_extensions()->load_extension()  
  ![](http://i.imgur.com/14LpbUH.png)  
  这里factory为nova.api.openstack.compute.legacy_v2.contrib.standard_extensions（）  
  self为ExtensionManager初始化  
  ![](http://i.imgur.com/NMaEjNz.png)  
  load_standard_extensions会把包路径下的文件一个个加载进来  
  ![](http://i.imgur.com/2zgm4xX.png)  
  这里以Admin_actions为例  
  ![](http://i.imgur.com/TpPdtqx.png)  
  可见同多数的extension一样，都继承自nova.api.openstack.ExtensionDescriptor  
  而ExtensionDescriptor在初始化的时候会将自己注册给ExtensionManager
  ![](http://i.imgur.com/BYWCyDh.png)  
  看下ExtensionManager的register()会发现，是把Extension放到self.extensions里  
  ![](http://i.imgur.com/OA1e8ce.png)  
  到这里ExtensionManager初始化完成，至此，ExtensionManager的exensions中缓存加载的extension.  
  ![](http://i.imgur.com/3KFWfA3.png)

- nova.api.openstack.APIRouter->_setup_routes  
  将url和controller建立对应关系  
  不同资源之间建立层级父子关系

- nova.api.openstack.APIRouter->_setup_ext_routes()  
  添加extension的资源相关

- _setup_extensions()
- super(APIRouter, self).__init__(mapper)  
  routes.middleware.RoutesMiddleware,将接受到的url，自动调用map.match()方法  
  将url进行路由匹配并将结果存入request请求的环境变量['wsgiorg.routing_args']  
  最后会调用其第一个参数给出的函数接口，即self.dispatch。  
  ![](http://i.imgur.com/MCbwl3F.png)  
  ![](http://i.imgur.com/9slPNtw.png)

### 请求处理篇 ###

    keystone = compute_req_id faultwrap sizelimit authtoken keystonecontext ratelimit osapi_compute_app_v2

这里
    [filter:compute_req_id]
    paste.filter_factory = nova.api.compute_req_id:ComputeReqIdMiddleware.factory

会依次调用call方法

    #############
    # OpenStack #
    #############

### v3与v2版本差异 ###

v3版本不再区分core api和extension api  
所有的都通过extension api方式提供  
![](http://i.imgur.com/04pigtm.png)  

加载的时候，读取配置文件setup.cfg中nova.api.v3.extensions部分
![](http://i.imgur.com/JpJigwo.png)

也就是说v3或者v2.1使用的代码目录是这里  
![](http://i.imgur.com/sIcJZeY.png)  

v2版本的实现代码是这里  
![](http://i.imgur.com/fkfyk0M.png)

### api-paste.ini全集 ###
    
    [composite:osapi_compute]  #入口会是这里         
    use = call:nova.api.openstack.urlmap:urlmap_factory #调用urlmap_factory方法，后面几行作为入参传入
    /: oscomputeversions
    /v1.1: openstack_compute_api_v2
    /v2: openstack_compute_api_v2
    /v2.1: openstack_compute_api_v21
    /v3: openstack_compute_api_v3
    
    [composite:openstack_compute_api_v2]
    use = call:nova.api.auth:pipeline_factory
    noauth2 = compute_req_id faultwrap sizelimit noauth2 ratelimit osapi_compute_app_v2
    keystone = compute_req_id faultwrap sizelimit authtoken keystonecontext ratelimit osapi_compute_app_v2
    keystone_nolimit = compute_req_id faultwrap sizelimit authtoken keystonecontext osapi_compute_app_v2
    
    [composite:openstack_compute_api_v21]
    use = call:nova.api.auth:pipeline_factory_v21
    noauth2 = compute_req_id faultwrap sizelimit noauth2 osapi_compute_app_v21
    keystone = compute_req_id faultwrap sizelimit authtoken keystonecontext osapi_compute_app_v21
    
    [composite:openstack_compute_api_v3]
    use = call:nova.api.auth:pipeline_factory_v21
    noauth = request_id faultwrap sizelimit noauth_v3 osapi_compute_app_v3
    noauth2 = request_id faultwrap sizelimit noauth_v3 osapi_compute_app_v3
    keystone = request_id faultwrap sizelimit authtoken keystonecontext osapi_compute_app_v3
    
    [filter:request_id]
    paste.filter_factory = oslo_middleware:RequestId.factory
    
    [filter:compute_req_id]
    paste.filter_factory = nova.api.compute_req_id:ComputeReqIdMiddleware.factory
    
    [filter:faultwrap]
    paste.filter_factory = nova.api.openstack:FaultWrapper.factory
    
    [filter:noauth2]
    paste.filter_factory = nova.api.openstack.auth:NoAuthMiddleware.factory
    
    [filter:noauth_v3]
    paste.filter_factory = nova.api.openstack.auth:NoAuthMiddlewareV3.factory
    
    [filter:ratelimit]
    paste.filter_factory = nova.api.openstack.compute.limits:RateLimitingMiddleware.factory
    
    [filter:sizelimit]
    paste.filter_factory = oslo_middleware:RequestBodySizeLimiter.factory
    
    [filter:legacy_v2_compatible]
    paste.filter_factory = nova.api.openstack:LegacyV2CompatibleWrapper.factory
    
    [app:osapi_compute_app_v2]
    paste.app_factory = nova.api.openstack.compute:APIRouter.factory
    
    [app:osapi_compute_app_v21]
    paste.app_factory = nova.api.openstack.compute:APIRouterV21.factory
    
    [app:osapi_compute_app_v3]
    paste.app_factory = nova.api.openstack.compute:APIRouterV3.factory
    
    [pipeline:oscomputeversions]
    pipeline = faultwrap oscomputeversionapp
    
    [app:oscomputeversionapp]
    paste.app_factory = nova.api.openstack.compute.versions:Versions.factory
    
    ##########
    # Shared #
    ##########
    
    [filter:keystonecontext]
    paste.filter_factory = nova.api.auth:NovaKeystoneContext.factory
    
    [filter:authtoken]
    paste.filter_factory = keystonemiddleware.auth_token:filter_factory

## RPC ##

（TODO）待分析rabbitmq