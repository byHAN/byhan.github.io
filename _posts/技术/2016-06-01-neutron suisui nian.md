---
layout: post
title: ovs入门指南
category: 技术
tags: 虚拟化层
keywords: ovs.ovs-vsctl
description: 
---

## 功能 ##

neutron中含有如下几个虚拟设备  
- network
- router  
  用来连接网络，使得不同的网络间能够相互通信。  
  分别使用interface和网络进行连接。  

- subnet  
  包含在network中  

- port  
  类似于交换机上的插入网线的网口。  
  给虚拟机连接网络用的槽口  

![](http://i.imgur.com/GZuSPqA.png)

## 架构 ##

- neutron-server  
  接收api请求，然后将请求转给后面的agent去处理，另外数据该入库的入库。  

- neutron-plugin-XXX  
  有很多很多的插件，提供琳琅满目的网络功能  
  根据不同情况选择所需 

## 网络分类 ##

- provider网络  
  管理侧的网络，可以有多种形式Flat、VLAN、GRE等  
  
- tenant网络  
  租户侧的网络，默认租户间隔离  
  所以，可以创建router，使用l3-agent使得不同的网络实现互相通信。  

## ovs实例模型 ##

- **package**:  

  包，网络转发的最小数据单元，来自某一个端口，被转发到一个或者多个端口。  

- **bridge**：  

  网桥，Linux中用于表示一个能连接不同网络设备的虚拟设备  
  linux中传统实现的网桥类似一个hub设备，而ovs管理的网桥一般类似交换机。  
  其作用是根据一定规则，把从端口收到的数据包转发到另一个或者多个端口。  
  1. br-int：综合网桥，常用于表示实现主要内部网络功能的网桥。  
  2. br-ex：外部网桥，通常表示负责跟外部网络通信的网桥。  

- **port**:
  openvSwitch中，每个端口都属于一个特定的网桥。  
  端口收到的数据包会经过流规则的处理，发往其他端口  
  也会把其他端口来的数据包发送出去.  
  主要有如下几种端口：  
  1. normal:操作系统中的网卡绑定到ovs上，ovs会生成一个普通端口处理这块网卡进出的数据包。
  2. Internal:端口类型为internal时，ovs会创建一块虚拟网卡，端口收到的所有数据包都会交给该网卡，发出的包会通过该端口交给ovs。当ovs创建一个新网桥时，默认会创建一个与网桥同名的Internal Port.  
  3. Patch:当机器中有多个ovs网桥时，可以使用Patch Port把两个网桥连起来。Patch Port总是成对出现，分别连接在两个网桥上，在两个网桥之间交换数据。  
  VETH：虚拟ethernet接口，通常以pair的方式出现，一端发出的网包，会被另一端接收，可以形成两个网桥之间的通道。  
	- qvb：neutron veth, Linux Bridge-side  
	- qvo：neutron veth, OVS-side  

  4. Tunnel:隧道端口是一种虚拟端口，支持使用gre或vxlan等隧道技术与位于网络上其他位置的远程端口通讯。  
  GRE：一种通过封装来实现隧道的方式，在openstack中一般是基于L3的GRE，即original pkt/GRE/IP/Ethernet


- TAP设备：模拟一个二层的网络设备，可以接受和发送二层网包。  

- TUN设备：模拟一个三层的网络设备，可以接受和发送三层网包。  

- iptables：Linux 上常见的实现安全策略的防火墙软件。

- Vlan：虚拟 Lan，同一个物理 Lan 下用标签实现隔离，可用标号为1-4094。

- VXLAN：一套利用 UDP 协议作为底层传输协议的 Overlay 实现。一般认为作为 VLan 技术的延伸或替代者。

- namespace：用来实现隔离的一套机制，不同 namespace 中的资源之间彼此不可见。  
  这里使用namaspace作为租户间隔离的。  

![](http://i.imgur.com/i3D6nHV.png)

## 命令简述 ##

![](http://i.imgur.com/4afPYnV.png)

- ovs-dpctl  
  datapath控制器,可以创建删除DP,控制DP中的FlowTables,最常使用show命令，其他很少手动操作  
  常用命令：
    ovs-dpctl show -s

- ovs-ofctl	流表控制器，控制bridge上的流表，查看端口统计信息等  

    ovs-ofctl show, dump-ports, dump-flows, add-flow, mod-flows, del-flows

- ovsdb-tool  专门管理ovsdb的client

    ovsdb-tools show-log -m

- ovs-vsctl	最常用的命令,通过操作ovsdb去管理相关的bridge,ports什么的  

1.ovs-vsctlshow 显示数据库内容  
  关于桥的操作ovs-vsctl add-br, list-br, del-br, br-exists.  
  关于port的操作ovs-vsctl list-ports, add-port, del-port, add-bond, port-to-br.  
  关于interface的操作ovs-vsctl list-ifaces, iface-to-br  
  ovs-vsctl list/set/get/add/remove/clear/destroy table record column [value], 常见的表有bridge, controller,interface,mirror,netflow,open_vswitch,port,qos,queue,ssl,sflow.  
  
- ovs-appctl	这个可以直接与openvswitch daemon进行交互,上图中没有列出来,这么命令较少使用  

    ovs-appctl list-commands, fdb/show, qos/show

## 实验 ##

创建虚拟机的时候，会给虚拟机分配一个tap设备作为虚拟机的网卡，名为tapXXX。  
在计算节点使用ifconfig查看，可以看到  
![](http://i.imgur.com/Y9iSxUZ.png)

建立一个linux网桥  
![](http://i.imgur.com/huszaQd.png)  

把上面的tap接在qbr网桥上，使用brctl-show查看网桥设备和端口  
![](http://i.imgur.com/06f1EjC.png)

小结：  
![](http://i.imgur.com/QiREtO9.png)
  
  

veth是linux的虚拟网络设备，总是成对出现  
一对veth设备的数据总是从一个流入,一个流出  
neutron会建立一对分别命名为qvbXXX和qvoXXX的veth设备  
并使用他们把qbr和br-int进行连接，使用brctl show查  
![](http://i.imgur.com/9bk5tHd.png)  
使用ovs-vsctl show查看br-int上绑定情况  
![](http://i.imgur.com/tujZTXl.png)　　

小结：虚拟机的tap已经联通到br-int  
![](http://i.imgur.com/8FSi48d.png)  

br-int和br-tun之间通过patch-in和patch-tun进行连接  
![](http://i.imgur.com/KWKl3Ep.png)  

小结：
至此，网络数据已经可以达到br-tun  
通过GRE隧道与其他节点上的tap设备通信（要求拥有相同segmentation，也就是项目中同一网络）  
![](http://i.imgur.com/4Uyzdf1.png)


## 参考文献 ##

http://www.tuicool.com/articles/neQBzmr
《openstack实战指南》