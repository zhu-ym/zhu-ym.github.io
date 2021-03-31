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

MVC是一种架构方式，其中：

M是模型，代表一个存取数据的对象

V是视图，负责页面的显示，与用户的交互，包含各种表单。 实现视图用到的技术有html/css/jsp/js等前端技术。

C是控制器，控制器作用于模型和视图上。它控制数据流向模型对象，并在数据变化时更新视图。它使视图与模型分离开。

MVC通过分离模型、视图以及控制器在应用程序中的角色将业务逻辑从界面中解耦，模型通过封装应用程序数据在视图层展示，视图仅仅只是展示这些数据，不包含任何业务逻辑。控制器负责接收用户请求，并调用后台服务（service、dao）来处理业务逻辑。**其核心在于将业务逻辑从视图中分离出来**，允许其单独改变而不会相互影响。

具体到Spring MVC，其是基于Java实现了Web MVC设计思想的请求驱动类型的轻量级web框架。其以请求为驱动，围绕Servlet设计，将请求发给控制器，然后通过模型对象，分派器来展示请求结果视图。其中核心类是DispatcherServlet，它是一个Servlet，顶层是实现的Servlet接口。

## Spring MVC 原理

Spring MVC 的请求流程主要如下图所示：

![image-20210322100338290](/images/image-20210322100338290.png)

**服务器运行准备**

- tomcat启动SpringMVC项目，加载该项目的web.xml文件（其中listener监听器可以监听到一些请求进行分配）

- 通过web.xml文件加载其中配置的SpringMVC配置文件

- java文件扫描SpringMVC配置文件中设置的扫描路径

- 配置了@Controller注解的类可以接受相应的数据，检索所有的Controller下的@RequestMapping，每个@Controller下的@RequestMapping对应一个map，其中整个@RequestMapping的全路径为key，（类路径+方法签名）为value。所有的Map放入一个map结构中。
  

**MVC工作流程**

- 服务器接受到http格式的请求信息，将其中所需要的信息提取出来，并将请求发送至前端控制器DispatcherServlet

- DispatcherServlet收到请求调用HandlerMapping处理器映射器，通过key找到实际访问的方法路径

- 处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，通过反射生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

- DispatcherServlet调用HandlerAdapter处理器适配器。

- HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)，其实就是代理执行相应的方法

- Controller执行完成返回ModelAndView。

- HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。

- DispatcherServlet将ModelAndView传给ViewReslover视图解析器。

- ViewReslover解析后返回具体View。

- DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。

- DispatcherServlet响应用户。
  

**组件说明**

- 前端控制器DispatcherServlet,由框架提供
  作用：接收请求，响应结果，相当于转发器，中央处理器。有了dispatcherServlet减少了其它组件之间的耦合度。
  用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性。

- 处理器映射器HandlerMapping(不需要工程师开发),由框架提供
  作用：根据请求的url查找Handler
  HandlerMapping负责根据用户请求找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

- 处理器适配器HandlerAdapter
  作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler
  通过HandlerAdapter执行处理器，这是适配器模式的应用，会把处理器包装成适配器，这样就可以支持多种类型的处理器，类比笔记本的适配器（适配器模式的应用）

- **处理器Handler**(需要工程师开发)
  注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler
  Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。
  由于Handler涉及到具体的用户业务请求，所以一般情况需要工程师根据业务需求开发Handler。

- 视图解析器View resolver,由框架提供
  作用：进行视图解析，根据逻辑视图名解析成真正的视图（view）
  View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。 springmvc框架提供了很多的View视图类型，包括：jstlView、freemarkerView、pdfView等。
  一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

- **视图View**(需要工程师开发)
  View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）



# Spring MVC 环境搭建

**开发环境**

IDEA + Maven + Jdk1.8 + Tomcat

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

  