---
layout: post
title:  "CentOS卸载openJDK安装OracleJDK"
categories: linux运维笔记
tags: CentOS7 JDK
author: Zk1an
---

* content
{:toc}
## 1、查看java版本

```shell
[root@CFDB2 ~]# java -version
openjdk version "1.8.0_171"
OpenJDK Runtime Environment (build 1.8.0_171-b10)
OpenJDK 64-Bit Server VM (build 25.171-b10, mixed mode)
```

## 2、查看java安装的软件

```shell
[root@CFDB2 ~]# rpm -qa|grep java
tzdata-java-2018e-3.el7.noarch
java-1.8.0-openjdk-1.8.0.171-8.b10.el7_5.x86_64
java-1.7.0-openjdk-headless-1.7.0.181-2.6.14.8.el7_5.x86_64
java-1.7.0-openjdk-1.7.0.181-2.6.14.8.el7_5.x86_64
javapackages-tools-3.4.1-11.el7.noarch
python-javapackages-3.4.1-11.el7.noarch
java-1.8.0-openjdk-headless-1.8.0.171-8.b10.el7_5.x86_64
[root@CFDB2 ~]#
```

## 3、卸载openjdk

.noarch可以不用删除  

```shell
rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.101-3.b13.el7_2.x86_64

```

## 4、下载JDK安装包

```shell
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" https://download.oracle.com/otn-pub/java/jdk/8u201-b09/42970487e3af4f5aa5bca3f542482c60/jdk-8u201-linux-x64.tar.gz
```

## 5、解压

```shell
tar -vzxf jdk-8u201-linux-x64.tar.gz
```



## 6、设置环境变量

```shell
vim /etc/profile
```

末尾添加  

```shell
#java_home
export JAVA_HOME=/usr/local/app/jdk1.8.0_201
export PATH=$PATH:$JAVA_HOME/bin
```

保存退出，执行命令  

```shell
source /etc/profile
```



## 7、测试

```shell
[root@localhost ruanjian]# java -version
java version "1.8.0_191"
Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
```






