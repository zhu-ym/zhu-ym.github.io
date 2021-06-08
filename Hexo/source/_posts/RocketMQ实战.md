---
title: RocketMQ实战
typora-root-url: ../
date: 2021-05-24 16:41:49
tags:
- 消息队列
categories:
- [中间件]
---

了解RocketMQ的简单实战

<!--more-->

# RocketMQ安装

## window环境

### 安装

**下载**

[下载地址](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.8.0/rocketmq-all-4.8.0-bin-release.zip)

**解压到安装目录**

![image-20210524170858192](/images/image-20210524170858192.png)

**配置系统环境变量**

```
  变量名：ROCKETMQ_HOME
  变量值：MQ解压路径\MQ文件夹名
```

![image-20210524171242901](/images/image-20210524171242901.png)

配置完毕后可能需要重启电脑

**启动NameServer**

在bin目录下打开cmd

执行`start mqnamesrv.cmd`可在另一个窗口启动`NameServer`

![image-20210524185157066](/images/image-20210524185157066.png)

**启动Broker**

在bin目录下打开cmd

执行`start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true`

可以可在另一个窗口启动`Broker`，并连接上之前启动的服务器

![image-20210524185236973](/images/image-20210524185236973.png)

### 可视化插件安装

可视化插件主要是可视化的监控，可以看到消息消费的具体情况

**下载**

首先在github上下载相关zip包

[下载地址](https://github.com/apache/rocketmq-externals.git)

**解压**

同理解压到相关路径

**修改配置文件**

修改进入`\rocketmq-externals-master\rocketmq-console\src\main\resources`
下的`application.properties`配置文件

这里我修改了其服务器默认端口为`8888`以及监听的是本机的rocketmq

![image-20210524190511648](/images/image-20210524190511648.png)



**编译项目**

在`\rocketmq-externals-master\rocketmq-console`下打开cmd

执行`mvn clean package -Dmaven.test.skip=true`编译插件

**启动插件**

最后在同级的target目录下，打开cmd

执行`java -jar rocketmq-console-ng-2.0.0.jar`启动插件

可以在浏览器中访问目的ip+端口查看

![image-20210524193036744](/images/image-20210524193036744.png)

## linux环境

同windows，我利用ftp将zip文件复制到了Linux服务器中

`unzip rocketmq-all-4.8.0-bin-release.zip && mv rocketmq-all-4.8.0-bin-release /root/rocketmq`

然后将其解压到`root`目录下



# RocketMQ实战