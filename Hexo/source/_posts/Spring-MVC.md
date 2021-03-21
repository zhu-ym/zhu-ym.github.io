---
title: Spring MVC
typora-root-url: ../
date: 2021-03-16 10:10:13
tags:
- Spring MVC
categories:
- [框架]
---

学习Spring MVC框架

<!--more-->

# Spring MVC原理

## MVC

MVC是一种架构方式，其中

M是模型，代表一个存取数据的对象

V是视图，负责页面的显示，与用户的交互，包含各种表单。 实现视图用到的技术有html/css/jsp/js等前端技术。

C是控制器，控制器作用于模型和视图上。它控制数据流向模型对象，并在数据变化时更新视图。它使视图与模型分离开。

在Spring MVC中，模型是bean、dao，控制器使用Servlet实现。

总的来说，SpringMVC框架是以请求为驱动，围绕Servlet设计，将请求发给控制器，然后通过模型对象，分派器来展示请求结果视图。其中核心类是DispatcherServlet，它是一个Servlet，顶层是实现的Servlet接口。





# 基于IDEA新建一个Spring MVC项目

这里主要基于Maven配置

- **首先创建一个Maven项目**

  注意图中勾选

  ![image-20210316151825513](/images/image-20210316151825513.png)

  之后命名，一路next

- **在pom.xml文件中导入相关依赖**

  主要是导入Spring，Servlet，Spring MVC 等相关依赖

  这里是我在网上导入的一锅端

  ```xml
  <dependencies>
      <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
      </dependency>
      <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>1.7.21</version>
    </dependency>
  
    <!--J2EE-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.2</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
    <!--mysql驱动包-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.35</version>
    </dependency>
    <!--springframework-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>4.2.6.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.2.6.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>4.2.6.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>4.2.6.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>4.2.6.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>com.github.stefanbirkner</groupId>
      <artifactId>system-rules</artifactId>
      <version>1.16.1</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.9</version>
    </dependency>
    <!--其他需要的包-->
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>3.4</version>
    </dependency>
    <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.3.1</version>
    </dependency>
    </dependencies>
  
  ```

- **导入依赖后，添加Spring MVC 框架支持**

  点击项目右键添加框架支持

  ![image-20210316153142758](/images/image-20210316153142758.png)

  **注意：**如果没有上诉界面，说明该项目有一定的Spring项目，但是不一定齐全，我们可以在`File->project structure ->facets`里面把Spring删除掉，再继续上一步步骤

  这一步成功，会添加两个xml配置文件，MVC的配置主要在这里进行

  ![image-20210316153652673](/images/image-20210316153652673.png)

- **部署到Tomcat服务器上**

  点击右上角`add configuration`,在点击左上角+号选择`TomCat Server  local`

  然后在`DeployMent`下点击+号`Artifact`添加本项目如下

  ![image-20210316154055526](/images/image-20210316154055526.png)

- **运行服务器**

  运行服务器，打开网页说明配置成功

  