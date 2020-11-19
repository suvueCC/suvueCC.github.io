---
layout: post
title:  "CentOs7下mysql5.7.31的安装"
categories: linux运维笔记
tags: centOS mysql
author: Zk1an
---

* content
{:toc}


>>MySQL常用运维命令  
>>启动MySQL服务：  
>>>```text
>>>systemctl start mysqld.service
>>>```  
>>重启MySQL服务：  
>>>```text
>>>systemctl restart mysqld.service
>>>```  
>>停止MySQL服务：  
>>>```text
>>>systemctl stop mysqld.service
>>>```  
>>查看MySQL服务运行状态：  
>>>```text
>>>systemctl status mysqld.service
>>>```  


## 一、下载指定版本的MySQL

[官网地址](https://dev.mysql.com/downloads/mysql/5.7.html#downloads)
- 下载的安装包必须是.rpm-bundle.tar结尾的，安装的时候，对应自己的下载版本号，不要直接复制命令。  
- 下载的同时，可以先在目标机器上创建一个存放该压缩包的文件夹  
```text
[root@cmp-mysql01 ~]#mkdir -p /usr/local/software/mysql5.7.31
```  
- 然后通过ftp将MySQL的安装包，上传到这个文件夹中。  
## 二、cd到mysql压缩包目录并解压  
```text
[root@cmp-mysql01 mysql5.7.31]#cd /usr/local/software/mysql5.7.31  
[root@cmp-mysql01 mysql5.7.31]# tar xvf mysql-5.7.31-1.el7.x86_64.rpm-bundle\ .tar  
```  
## 三、卸载掉centos7自带的mariadb-lib   
***方法一不好使的情况下，再试试方法二***  
### 3.1、方法一  
- 查询mariadb信息  
```text
[root@cmp-mysql01 mysql5.7.31]# rpm -qa|grep mariadb  
mariadb-libs-5.5.65-1.el7.x86_64  
```  
- 使用rpe -e命令卸载  
```text
[root@cmp-mysql01 mysql5.7.31]# rpm -e mariadb-libs-5.5.65-1.el7.x86_64 --nodeps  
```  
### 3.2、方法二  
- 使用yum remove 命名进行删除  
```text
[root@cmp-mysql01 mysql5.7.31]# yum remove mysql-libs  
```  
## 四、安装mysql-server服务，只需要安装如下4个软件包即可，使用rpm -ivh进行安装（按顺序安装，后面的服务依赖前面的服务）  
### 4.1、mysql-community-common安装/升级  
Linux命令：  
```text
[root@cmp-mysql01 mysql5.7.31]# rpm -ivh mysql-community-common-5.7.31-1.el7.x86_64.rpm  
```  
输出以下内容表示成功  
```text
警告：mysql-community-common-5.7.31-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-common-5.7.31-1.e################################# [100%]
```  
### 4.2、mysql-community-libs安装/升级  
Linux命令：  
```text
[root@cmp-mysql01 mysql5.7.31]# rpm -ivh mysql-community-libs-5.7.31-1.el7.x86_64.rpm  
```  
输出以下内容表示成功  
```text
警告：mysql-community-libs-5.7.31-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-libs-5.7.31-1.el7################################# [100%]
```  
### 4.3、mysql-community-client安装/升级  
Linux命令：  
```text
[root@cmp-mysql01 mysql5.7.31]# rpm -ivh mysql-community-client-5.7.31-1.el7.x86_64.rpm  
```  
输出以下内容表示成功  
```text
警告：mysql-community-client-5.7.31-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-client-5.7.31-1.e################################# [100%]
```  
### 4.4、mysql-community-client安装/升级  
Linux命令：  
```text
[root@cmp-mysql01 mysql5.7.31]# rpm -ivh mysql-community-server-5.7.31-1.el7.x86_64.rpm
```  

输出以下内容表示成功  
```text
警告：mysql-community-server-5.7.31-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-server-5.7.31-1.e################################# [100%]
```  

### 4.5、安装过程中可能出现的问题（仅供参考）  
- 缺少libaio  
```text
[root@cmp-mysql01 mysql5.7.31]# rpm -ivh mysql-community-server-5.7.31-1.el7.x86_64.rpm
警告：mysql-community-server-5.7.17-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
错误：依赖检测失败：
	libaio.so.1()(64bit) 被 mysql-community-server-5.7.17-1.el7.x86_64 需要
	libaio.so.1(LIBAIO_0.1)(64bit) 被 mysql-community-server-5.7.17-1.el7.x86_64 需要
	libaio.so.1(LIBAIO_0.4)(64bit) 被 mysql-community-server-5.7.17-1.el7.x86_64 需要
	net-tools 被 mysql-community-server-5.7.31-1.el7.x86_64 需要

解决办法：
[root@cmp-mysql01 mysql5.7.31]# yum install libaio
```  

- 缺少net-tools  

```text
[root@cmp-mysql01 mysql5.7.31]# rpm -ivh mysql-community-server-5.7.31-1.el7.x86_64.rpm 
警告：mysql-community-server-5.7.17-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
错误：依赖检测失败：
	net-tools 被 mysql-community-server-5.7.31-1.el7.x86_64 需要

解决办法：
yum install net-tools
```  

- 缺少numactl  
```text
[root@cmp-mysql01 mysql5.7.31]# rpm -ivh mysql-community-server-5.7.31-1.el7.x86_64.rpm 
 
报错：warning: mysql-community-server-5.7.9-1.el6.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
error: Failed dependencies:
        libnuma.so.1()(64bit) is needed by mysql-community-server-5.7.31-1.el6.x86_64
        libnuma.so.1(libnuma_1.1)(64bit) is needed by mysql-community-server-5.7.31-1.el6.x86_64
        libnuma.so.1(libnuma_1.2)(64bit) is needed by mysql-community-server-5.7.31-1.el6.x86_64
 
解决办法：
   yum  install numactl
```  

## 五、初始化数据库  

命令：  
```text
[root@cmp-mysql01 mysql5.7.31]# mysqld --initialize
```  
***注意：初始化后会在/var/log/mysqld.log生成随机密码***  
## 六、修改mysql数据库目录的所属用户及其所属组，然后启动mysql数据库  
```text
[root@cmp-mysql01 mysql5.7.31]# chown mysql:mysql /var/lib/mysql -R
[root@cmp-mysql01 mysql5.7.31]# systemctl start mysqld.service
[root@cmp-mysql01 mysql5.7.31]# systemctl status mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 二 2020-09-29 12:07:01 CST; 24s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 4330 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 4306 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 4333 (mysqld)
    Tasks: 27
   CGroup: /system.slice/mysqld.service
           └─4333 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

9月 29 12:06:57 cmp-mysql01 systemd[1]: Starting MySQL Server...
9月 29 12:07:01 cmp-mysql01 systemd[1]: Started MySQL Server.
```  
## 七、登录mysql，并修改root用户的密码（系统强制要求，否则不能操作mysql）  
***初始登录密码可在/var/log/mysqld.log文件中找到***   
```text
[root@cmp-mysql01 mysql5.7.31]# mysql -uroot -p'-4iq<tyjVpLb'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.23
 
Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.
 
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
 
mysql> set password=password('123456');
Query OK, 0 rows affected, 1 warning (0.00 sec)
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
 
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```  
## 八、修改访问权限  
目的：任何主机通过用户root和密码123456连接到mysql服务器，并授权所有权限  
```text
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456
' WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.00 sec)
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```  

## 九、重启MySQL服务  
```text
[root@cmp-mysql01 mysql5.7.31]# systemctl restart mysqld.service
```  
## 十、防火墙暴露3306端口  
开放3306端口并重新加载：  
```text
[root@cmp-mysql01 mysql5.7.31]# firewall-cmd --zone=public --add-port=3306/tcp --permanent
[root@cmp-mysql01 mysql5.7.31]# firewall-cmd --reload
```
查看所有打开的端口：  
```text
firewall-cmd --zone=public --list-ports
```  
 






