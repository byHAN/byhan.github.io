---
layout: post
title: 计算节点间互信配置
category: nova
tags: nova
keywords: 
description: 
---

## 一句话介绍 ##

计算节点间需要互信来完成相关功能  
如迁移时候，nova会在目标节点创建相关文件  

## 背景介绍 ##

重装计算节点os，不知道怎么的，把互信给搞没了  
导致迁移和resize等功能报错如下  
![](http://i.imgur.com/uzVq3oZ.png)

## 步骤##

#### 1.设置nova账号 ####

设置nova账号可以登录

    $ usermod -s /bin/bash nova  
    $ su - nova  

#### 2.生成公钥私钥 ####

可以使用已有的，也可以通过sshken-gen生成  
注意：所有参加互信的节点使用一套公钥和私钥。  
也就是说，根据公钥认证的原理，这样可以做到互信。  
注意，需要参加互信的计算节点都需要拷贝到  

    nova@compute01:~$ mkdir -p -m 700 .ssh
    nova@compute01:~$ cat > .ssh/config <<EOF
    Host *
    StrictHostKeyChecking no
    UserKnownHostsFile=/dev/null
    EOF
    nova@compute01:~$ ssh-keygen -f id_rsa -b 1024 -P ""
    nova@compute01:~$ scp /var/lib/nova/.ssh/id_rsa.pub root@compute01:/var/lib/nova/.ssh/authorized_keys
    nova@compute01:~$ scp /var/lib/nova/.ssh/id_rsa.pub root@compute02:/var/lib/nova/.ssh/authorized_keys
    nova@compute01:~$ scp /var/lib/nova/.ssh/* root@compute02:/var/lib/nova/.ssh/

切换回root，修改下相关权限  

    $ sudo chown -R nova:nova /var/lib/nova/
    $ sudo chmod 700 /var/lib/nova/.ssh
    $ sudo chmod 600 /var/lib/nova/.ssh/authorized_keys

#### 3.验证 ####

nova账号下登陆互信的站点，查看是否需要密码  


## 疑问 ##

1.正常情况下互信是如何设置的，何时设置的？  
（TODO）  
2.nova用户互信路径为什么是/var/lib/nova/.ssh/?   

## 参考文档 ##

优先参考这个：  
[https://www.sebastien-han.fr/blog/2015/01/06/openstack-configure-vm-migrate-nova-ssh/ ](https://www.sebastien-han.fr/blog/2015/01/06/openstack-configure-vm-migrate-nova-ssh/ )  
[http://docs.openstack.org/admin-guide/cli_nova_migrate_cfg_ssh.html](http://docs.openstack.org/admin-guide/cli_nova_migrate_cfg_ssh.html)  (官网写的有点模糊)  
[http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)  


