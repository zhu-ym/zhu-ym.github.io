---
title: Tomcat安装
typora-root-url: ../
date: 2021-03-07 14:52:36
tags:
- Tomcat
- IDEA
categories:
- [框架]
- [IDEA相关]
---

安装Tomcat，设置环境变量，并且在IDEA上进行配置

<!--more-->

# Tomcat概述

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，也是Servlet的容器

其目录结构一般如下

![image-20210307153940964](/images/image-20210307153940964.png)

其中

- bin是命令文件，我们一般只会用到其中startup和shutdown打开和关闭服务器

- conf是配置文件，我们可能会需要打开其server.xml文件，在如下位置修改端口号

  默认是8080，我们可以选择修改为8088，因为安装Oracle数据库，Oracle会占用8080端口，可能造成端口冲突

  ![image-20210307155114695](/images/image-20210307155114695.png)

- lib：tomcat运行所需要的jar包
- logs：tomcat服务器日志文件
- temp：tomcat运行产生的临时文件
- **webapps：需要发布的项目存放在webapps下，放置web应用**
- work：JSP翻译（编译）成servlet产生的代码

# Tomcat安装

- **下载zip包（免安装，解压即可）**，解压在自己常用的根目录下

  [下载链接](https://tomcat.apache.org/download-10.cgi)

  解压打开后目录一个同概述的目录一致

- **环境配置**

  - 在高级系统设置中，点击环境变量，配置系统变量

  - 新建一个TOMCAT_HOME系统变量，变量值同解压后绝对路径

    ![image-20210307150608806](/images/image-20210307150608806.png)

  - 在Path中新建%TOMCAT_HOME%\bin;

    ![image-20210307151107180](/images/image-20210307151107180.png)

  - 在Classpath（没有即新建）中最后添加%TOMCAT_HOME%\lib\servlet-api.jar

- **配置完成**

  双击解压后bin目录下的startup批处理文件，如果窗口能够打开，上面环境配置成功，此时相当于打开了Tomcat服务器

  点击http://localhost:8080/     便可以访问

# 在IDEA集成Tomcat

点击file->settings

找到如下位置

![image-20210307160037872](/images/image-20210307160037872.png)

初始时应该是空的，点击+号，添加Tomcat server

![image-20210307160209360](/images/image-20210307160209360.png)

添加路径时，到我们解压目录bin上一级即可

之后一直ok，集成完毕

