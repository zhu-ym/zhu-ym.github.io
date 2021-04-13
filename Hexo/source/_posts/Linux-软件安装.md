---
title: Linux-软件安装
typora-root-url: ../
date: 2021-03-31 09:57:12
tags:
- Linux
categories:
- [操作系统,Linux]
---

主要是在Linux下安装Java开发环境

<!--more-->

安装软件一般有三种方式

- rpm

- 解压缩

- yum在线安装

# JDK安装

**这里采用rpm安装**

首先可以在自己的机子上安装XFTP和Xshell方便上传文件和远程连接服务器

- **本机下载Linux下相应rpm安装包**

  [下载地址](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)

  我下载的是`jdk-8u281-linux-x64.rpm`版本

- **放置目录**

  我打算放在在Opt目录下，新建一个文件夹Java文件夹

  ```
  mkdir /opt/java
  ```

  然后在XFTP中直接将下载文件上传到服务器的`/opt/java`目录下

- **设置文件权限**

  ![image-20210331104854873](/images/image-20210331104854873.png)

  上传后可以发现其没有运行权限，所以利用

  `chmod 777 jdk-8u281-linux-x64.rpm `

  设置其有运行权限

- **rpm命令安装**

  先判断是否有jdk环境

  ```
  rpm -qa|grep jdk #检测jdk版本信息
  rpm -e --nodeps 查询到的信息  #卸载
  ```

  安装命令

  ```java
  rpm -ivk 安装包
  ```

  安装完毕后，可以`java -version`查看是否成功

  rpm安装方式，通过RPM方式安装时，是严格遵照Linux对环境变量的路径的要求,不需配置环境，不需安装相关依赖（依赖自动安装）

- **运行项目**

  一般需要将项目打包成jar包

  直接用`java -jar jar包`名执行

  

# Tomcat安装

这里采用**解压缩方式安装**

这里是针对如果Web项目不是Spring Boot ,比如SSM war包就需要放在Tomcat中运行

- **下载压缩包并上传到Linux上**

  [下载地址](https://tomcat.apache.org/download-10.cgi)

  我下载的是`apache-tomcat-10.0.4.tar.gz`

- **解压**

  在放置目录，tar命令解压

  ```
  tar-zxvf apache-tomcat-10.0.4.tar.gz
  ```

- **启动停止**

  在bin目录下`./startup.sh` 启动Tomcat

  ```
  ./startup.sh 启动
  ./shutdown.sh 关闭
  ```

- **对Tomcat8080端口防火墙进行管理**

  主要看下面后三步，看8080端口是否打开，没有则打开重启生效

  ```
  systemctl status firewalld #防火墙状态
  firewall-cmd --zone=public --list-ports #查看端口
  firewall-cmd --zone=public --add-port=8080/tcp --permanent #打开指定端口
  firewall-cmd --reload 更新修改
  ```

  做完上诉流程，则可以通过服务器地址8080端口访问Tomcat

# **Redis安装**

这里采用**yum**进行在线安装

```'

yum -y inatall 包名
```

我们安装redis

即

```
yum -y install redis 即可
```

```
对redis的基本操作：启动、查看状态、停止、重启
systemctl start redis
systemctl status redis
systemctl stop redis 
systemctl restart redis

ps -ef |grep redis          查看redis进程
systemctl enable redis      设置开机自启动

```

防火墙设置

Redis需要对6379防火墙进行设置

就是上面的老操作

```
systemctl status firewalld #防火墙状态
firewall-cmd --zone=public --list-ports #查看端口
firewall-cmd --zone=public --add-port=8080/tcp --permanent #打开指定端口
firewall-cmd --reload 更新修改
```



**Redis配置修改**

进入编辑配置文科

`vim /etc/redis.conf`

```
bind 127.0.0.1 注释掉 #bind 127.0.0.1 否则只能本机访问
daemonize no 改为 daemonize yes 可以后台运行
```



