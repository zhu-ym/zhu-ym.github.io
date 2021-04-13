---
title: Jmeter压力测试
typora-root-url: ../
date: 2021-03-29 11:12:56
tags:
- Jmeter
- 压力测试
categories:
- [项目,秒杀系统]
---

对秒杀系统进行压力测试

<!--more-->

# 1.安装Jmeter

- **下载压缩包**

  [下载地址](https://jmeter.apache.org/download_jmeter.cgi)

- **解压到安装目录**

  解压目录即安装目录，`D:\Program\apache-jmeter-5.4.1`

- **配置系统环境变量**

  - 首先保证自己之前已经建立好Java环境，然后在打开系统属性，选择高级->环境变量中新建一个JMETER_Home系统变量，值为其安装目录bin上一级，我的是`D:\Program\apache-jmeter-5.4.1`,
  - 最后在在编辑CLASSPATH对象，加上`%JMETER_HOME%\lib\ext\ApacheJMeter_core.jar;%JMETER_HOME%\lib\jorphan.jar;%JMETER_HOME%\lib\logkit-2.0.jar;`即可

- **打开Jmeter**

  在安装目录下打开bin文件夹，找到`jmeter.bat`打开即可启用

# 2.简单使用Jmeter进行压测

首先打开Jmeter后，可以在`Options`->`choose language`里面将语言改为中文

**1.添加线程组**

右键测试计划，添加线程里面选择线程组

![image-20210329185400986](/images/image-20210329185400986.png)

关键参数是线程数、Ramp-up time和循环次数

Ramp-up time是规定所有线程在时间段内把请求执行完（指一次循环），而且请求的时间间隔是固定的=Ramp-Up time/线程数。所以设置为0可以用压测服务器最大的能力持续发送请求

循环次数和线程数的参数不难理解



添加完线程组之后，应该需要对HTTP请求进行管理，下面几步就是管理步骤



**2.添加默认请求头信息**

点击线程组右键添加配置元件，添加HTTP请求默认值

![image-20210329190855888](/images/image-20210329190855888.png)

这里我填写web站点的基础信息

下面设置具体的HTTP请求参数

**3.具体HTTP请求参数**

首先声明，我这次压测是模拟所有用户对一个商品的秒杀，这里需要不同的userId作为请求报文的参数，需要对所有请求配置不同的userId参数，而商品id的参数可以固定

- **设置不同用户Id**

  - 从数据库在将userId导出为CSV文件（txt）也可以，每个ID一行，我保存在桌面data中

  ![image-20210329191532991](/images/image-20210329191532991.png)

  - 点击线程组，右键添加配置文件，添加`CSV数据文件设置`

    ![image-20210329191913682](/images/image-20210329191913682.png)

    在文件名中导入文件，并设置变量名为userId，其他默认

- **确定具体HTTP请求参数**

  点击线程组，右键取件器，添加HTTP请求

  设置请求具体路径，并且点击添加添加两个参数，一个是userId，用`${userId}`取值，其会每一个请求从data文件中读一行取值作为userId值

  另一个参数goodsId设置为固定值

  ![image-20210329192644097](/images/image-20210329192644097.png)

**4.添加监听器**

点击线程组，右键选择监听器，可以添加查看结果树、汇总报告等记录测试信息

**5.运行结果**

点击上方绿三角运行，第一次压测结束

![image-20210329193330908](/images/image-20210329193330908.png)