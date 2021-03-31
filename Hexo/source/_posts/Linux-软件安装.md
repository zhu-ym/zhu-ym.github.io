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

这里采用rpm安装

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

  