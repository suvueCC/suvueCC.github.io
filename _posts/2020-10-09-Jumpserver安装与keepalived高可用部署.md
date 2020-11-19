---
layout: post
title:  "Jumpserver安装与keepalived高可用部署"
categories: linux运维笔记
tags: jumpserver keepalived 高可用
author: Zk1an
---

* content
{:toc}

# 一、环境准备  
| Node | IP |
| :-----: | :----: |
| master1 | 172.16.10.81 |
| master2 | 172.16.10.82 |  
| VIP | 172.16.10.80 |  
# 二、安装并配置keepalived  
## 2.1、【master1】 keepalived安装配置  
- 安装keepalived服务  
```text
[root@master1 ~]# yum install -y openssl-devel
[root@master1 ~]# wget http://www.percona.com/redir/downloads/Percona-XtraDB-Cluster/5.5.37-25.10/RPM/rhel6/x86_64/Percona-XtraDB-Cluster-shared-55-5.5.37-25.10.756.el6.x86_64.rpm
[root@master1 ~]# rpm -ivh Percona-XtraDB-Cluster-shared-55-5.5.37-25.10.756.el6.x86_64.rpm 
[root@master1 ~]# yum install keepalived 
[root@master1 ~]# vim /etc/keepalived/keepalived.conf
```  
  
- 清空默认内容，直接采用下面配置  
```text
! Configuration File for keepalived
global_defs {
router_id MASTER-HA
}
vrrp_script chk_haproxy_port { #检测web服务是否在运行。有很多方式，比如进程，用脚本检测等等
script "/opt/shell/chk_haproxy.sh" #这里通过脚本监测
interval 2 #脚本执行间隔，每2s检测一次
weight -5 #脚本结果导致的优先级变更，检测失败（脚本返回非0）则优先级 -5
fall 2 #检测连续2次失败才算确定是真失败。会用weight减少优先级（1-255之间）
rise 1 #检测1次成功就算成功。但不修改优先级
}
vrrp_instance VI_1 {
state MASTER
interface ens192 #指定虚拟ip的网卡接口 可以通过ifconfig查看
virtual_router_id 88 #路由器标识，MASTER和BACKUP必须是一致的
priority 101 #定义优先级，数字越大，优先级越高，在同一个vrrp_instance下，MASTER的优先级必须大于BACKUP的优先级。这样MASTER故障恢复后，就可以将VIP资源再次抢回来
advert_int 1
authentication {
auth_type PASS
auth_pass 123456 # 认证密码 两端需要一致
}
virtual_ipaddress {
 172.16.10.80 # 虚拟IP 地址
}
track_script {
chk_haproxy_port
}
}
```  
  
- 编写健康监测的脚本  
```text
[root@master1 ~]# mkdir -p /opt/shell
[root@master1 ~]# vi /opt/shell/chk_haproxy.sh  
#脚本中放入以下内容： 
#!/bin/bash
# 检测脚本默认检测8080端口，如实际场景为其他端口，自行修改
counter=$(netstat -ntpl|grep 8080|wc -l)
if [ "${counter}" -eq 0 ]; then
systemctl stop keepalived
fi
[root@master1 ~]# chmod 755 /opt/shell/chk_haproxy.sh  
```  
  
- 将防火墙的vvrp开启。注意命令中要修改自己的网卡名称（这里是ens192），解决脑裂问题  
```text
[root@master1 ~]# firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 --in-interface ens192 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
[root@master1 ~]# firewall-cmd --reload
```  
  
- 启动keepalived服务  
  
```text
[root@master1 ~]# systemctl start keepalived
```  
  
## 2.2、【master2】keepalived安装配置  
- 安装keepalived服务  
  
```text
[root@master2 ~]# yum install -y openssl-devel
[root@master2 ~]# wget http://www.percona.com/redir/downloads/Percona-XtraDB-Cluster/5.5.37-25.10/RPM/rhel6/x86_64/Percona-XtraDB-Cluster-shared-55-5.5.37-25.10.756.el6.x86_64.rpm
[root@master2 ~]# rpm -ivh Percona-XtraDB-Cluster-shared-55-5.5.37-25.10.756.el6.x86_64.rpm 
[root@master2 ~]# yum install keepalived 
[root@master2 ~]# vim /etc/keepalived/keepalived.conf
```  
- 清空默认内容，直接采用下面配置  
  
```text
! Configuration File for keepalived
global_defs {
router_id BACKUP-HA
}
vrrp_script chk_haproxy_port { #检测web服务是否在运行。有很多方式，比如进程，用脚本检测等等
script "/opt/shell/chk_haproxy.sh" #这里通过脚本监测
interval 2 #脚本执行间隔，每2s检测一次
weight -5 #脚本结果导致的优先级变更，检测失败（脚本返回非0）则优先级 -5
fall 2 #检测连续2次失败才算确定是真失败。会用weight减少优先级（1-255之间）
rise 1 #检测1次成功就算成功。但不修改优先级
}
vrrp_instance VI_1 {
state BACKUP
interface ens192 #指定虚拟ip的网卡接口 可以通过ifconfig
virtual_router_id 88 #路由器标识，MASTER和BACKUP必须是一致的
priority 99 #定义优先级，数字越大，优先级越高，在同一个vrrp_instance下，MASTER的优先级必须大于BACKUP的优先级。这样MASTER故障恢复后，就可以将VIP资源再次抢回来
advert_int 1
authentication {
auth_type PASS
auth_pass 123456
}
virtual_ipaddress {
172.16.10.80 #虚拟IP地址
}
track_script {
chk_haproxy_port
}
}
```  
- 编写健康监测的脚本  
```text
[root@master2 ~]# mkdir -p /opt/shell
[root@master2 ~]# vi /opt/shell/chk_haproxy.sh  
#脚本中放入以下内容： 
#!/bin/bash

# 检测脚本默认检测8080端口，如实际场景为其他端口，自行修改
counter=$(netstat -ntpl|grep 8080|wc -l)
if [ "${counter}" -eq 0 ]; then
systemctl stop keepalived
fi
[root@master2 ~]# chmod 755 /opt/shell/chk_haproxy.sh  
```  
- 将防火墙的vvrp开启。注意命令中要修改自己的网卡名称（这里是ens192），解决脑裂问题  
```text
[root@master2 ~]# firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 --in-interface ens192 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
[root@master2 ~]# firewall-cmd --reload
```  
- 启动keepalived服务  
```text
[root@master2 ~]# systemctl start keepalived
``` 
# 三、【master1、master2】jumpserver堡垒机单机安装  
可参照官网上的jumpserver安装手册操作，大致过程如下:  
## 3.1、解压安装包到路径/opt/jumpserver-release  
## 3.2、运行./jmsctl.sh install 进行安装  
## 3.3、数据库和redis用的是外部服务，部署在一台机子上，因此我们将VIP地址配置为数据库的VIP（虚拟IP），其余按正常来配  
## 3.4、启动堡垒机服务  
```text
./jmsctl.sh start
```  
**注意：安装成功后，要将jumpserver的访问端口（这里是8080）在防火墙设置中暴露出来，
一定不要关闭防火墙！  
一定不要关闭防火墙！！  
一定不要关闭防火墙！！！  
切记！不然会遇到访问504的问题**  
# 四、堡垒机+keepalived高可用测试  
查看VIP  
```text
[root@master1 ~]# ip addr|grep 172.16
    inet 172.16.10.81/21 brd 172.16.151.255 scope global noprefixroute ens192
    inet 172.16.10.80/32 scope global ens192  
[root@master2 ~]# ip addr|grep 172.16  
    inet 172.16.10.82/21 brd 172.16.151.255 scope global noprefixroute ens192
```  
发现VIP在master1节点上，我们先暂时关掉master1的MySQL服务，观察VIP的变化  
```text
[root@master1 ~]# ./jmsctl.sh down
[root@master1 ~]# ip addr|grep 172.16  
    inet 172.16.10.81/21 brd 172.16.151.255 scope global noprefixroute ens192
[root@master2 ~]# ip addr|grep 172.16  
    inet 172.16.10.82/21 brd 172.16.151.255 scope global noprefixroute ens192
    inet 172.16.10.80/32 scope global ens192    
```  

上面我们模拟了当master1故障时，VIP会漂移到master2节点上，下面我们再将master1的MySQL和keepalived服务恢复正常，再看一下VIP的变化  

```text
[root@master1 ~]# ./jmsctl.sh start 
[root@master1 ~]# systemctl start keepalived  
[root@master1 ~]# ip addr|grep 172.16   
    inet 172.16.10.81/21 brd 172.16.151.255 scope global noprefixroute ens192
    inet 172.16.10.80/32 scope global ens192 
[root@master2 ~]# ip addr|grep 172.16  
    inet 172.16.10.82/21 brd 172.16.151.255 scope global noprefixroute ens192  
```  
最后，我们发现当master1节点恢复后，VIP又自动漂移到了master1节点  

