---
layout: post
title:  "CentOS7使用firewalld打开关闭防火墙与端口"
categories: Linux
tags: 防火墙 CentOS7
author: Zk1an
---

* content
{:toc}

## 1、firewalld的基本使用
启动  
```shell script
systemctl start firewalld
```
  
关闭  
```shell script
systemctl stop firewalld
```
  
查看状态  
```shell script
systemctl status firewalld
```
  
开机禁用  
```shell script
systemctl disable firewalld
```
  
开机启用  
      
```shell script
systemctl enable firewalld
```
 
 
## 2.systemctl是CentOS7的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体。  
启动一个服务  
```shell script
systemctl start firewalld.service
```
  
关闭一个服务  
```shell script
systemctl stop firewalld.service
```
  
重启一个服务  
```shell script
systemctl restart firewalld.service
```
  
显示一个服务的状态  
```shell script
systemctl status firewalld.service
```
  
在开机时启用一个服务  
```shell script
systemctl enable firewalld.service
```
  
在开机时禁用一个服务  
```shell script
systemctl disable firewalld.service
```
  
查看服务是否开机启动  
```shell script
systemctl is-enabled firewalld.service
```
  
查看已启动的服务列表  
```shell script
systemctl list-unit-files|grep enabled
```
  
查看启动失败的服务列表  
```shell script
systemctl --failed
```
  
## 3.配置firewalld-cmd  
  
查看版本    
```shell script
firewall-cmd --version
```
  
查看帮助  
```shell script
firewall-cmd --help
```
  
显示状态  
```shell script
firewall-cmd --state
```
  
查看所有打开的端口  
```shell script
firewall-cmd --zone=public --list-ports
```
  
更新防火墙规则  
```shell script
firewall-cmd --reload
```
  
查看区域信息  
```shell script
firewall-cmd --get-active-zones
```
  
查看指定接口所属区域  
```shell script
firewall-cmd --get-zone-of-interface=eth0
```
  
拒绝所有包  
```shell script
firewall-cmd --panic-on
```
  
取消拒绝状态  
```shell script
firewall-cmd --panic-off
```

  
查看是否拒绝  
```shell script
firewall-cmd --query-panic
```
   
那怎么开启一个端口呢  
  
添加（--permanent永久生效，没有此参数重启后失效）  
  
```shell script
firewall-cmd --zone=public --add-port=80/tcp --permanent
```
  
重新载入  
  
```shell script
firewall-cmd --reload
```
  
查看  
```shell script
firewall-cmd --zone= public --query-port=80/tcp
```
  
删除  
```shell script
firewall-cmd --zone= public --remove-port=80/tcp --permanent
```