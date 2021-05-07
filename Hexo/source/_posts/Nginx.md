---
title: Nginx
typora-root-url: ../
date: 2021-04-21 14:55:07
tags:
- Nginx
categories:
- [中间件]
---

了解Nginx的安装、使用、以及一些常用功能

<!--more-->

# 一、Nginx安装

Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。

其主要功能是**反向代理、负载均衡**

特点是占有内存少，并发能力强，并且配置也比较简单

下面是其在window环境下的安装：

- **1.下载安装包**

  Nginx安装通过解压zip压缩包到指定目录即可

   [下载地址](http://nginx.org/en/download.html)

- **2.启动**

  可以点击压缩文件下的exe程序直接打开

  也可以打开cmd命令窗口，切换到nginx解压目录下，输入命令 `start nginx` ，回车即可

  最后点击http://localhost:80   

  出现如下界面即成功

  ![image-20210506155920084](/images/image-20210506155920084.png)

- **3.关闭**

  最后可以用任务管理器或者在命令行输入`nginx -s stop`关闭

- **4.修改配置**

  Nginx可以在conf文件中修改配置，并且是热部署方式，可以通过以下指令不需要stop Nginx，不需要中断请求，就能让配置文件生效

  `nginx -s reload `



# 二、反向代理

## 原理

## 实例

首先在本机上运行一个web项目

然后在`nginx.conf`文件中的`server`块中修改`location`中内容如下

```yaml
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass http://211.67.16.199:8080/seckill;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
```

其中，监听本地服务器的80端口

**location /：** 表示处理所有请求
**proxy_pass http://211.67.16.199:8080/seckill：** 表示把请求都交给该网站来处理

修改后`nginx -s reload `让配置生效

这时再访问本机的80端口就会反向代理到web项目部署的网站

# 三、负载均衡

## 原理

## 实例

启动两台机子上的web项目

添加一个增加一个upstream，用来指向不同的服务器

```yaml
   upstream mysever {
        #weigth参数表示权值，权值越高被分配到的几率越大
        server 211.67.16.199:8080 weight=3;
        server 119.23.65.18:8080 weight=1;
    }
```

最后还是修改location好反向代理请求

```yaml
    location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://myserver/seckill;
        }
```

这里注意不能直接在upstream中直接添加项目名seckill,否则会报错