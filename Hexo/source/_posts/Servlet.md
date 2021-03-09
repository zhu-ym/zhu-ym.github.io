---
title: Servlet
typora-root-url: ../
date: 2021-03-07 10:34:06
tags:
- Servlet
categories:
- [Web]
---

学习Spring MVC前，掌握Servlet的学习

<!--more-->

# Servlet概念

Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

![image-20210307164102396](/images/image-20210307164102396.png)



Servlet 本身不能独立运行，需要在一个web应用中运行，而一个web应用是部署在tomcat中的，所以开发一个servlet需要如下几个步骤
创建web应用项目
编写servlet代码
部署到tomcat中

下面我们首先学习一个基于IDEA的简单的实例来体会这个过程

# 一个基于IDEA的简单Servlet实例

根据前面的三个步骤，我们首先要对IDEA进行一些准备

- **IDEA集成Tomcat**

  具体见该博客框架分类下Tomcat的内容

- **保证能创建Java web应用项目**

  按ctrl+Alt+Shift+/快捷键，在registry中找到`javaee.legacy.project.wizard`确保其被勾上

- **创建web应用项目**

  新建一个project，选择Java EE，点击Web Application之后，next即可

  ![image-20210307165241199](/images/image-20210307165241199.png)

  之后就是命名不再赘述

  下面是目录结构，out是运行后产生的，不用多管

  ![image-20210307165803292](/images/image-20210307165803292.png)

- **编写Servlet代码**

  现在只是一个web项目，我们要在src下新建一个包，包里新建一个类继承HttpServlet才算一个Servlet项目

  再重写HttpServlet类中的server方法，对HTTP请求进行处理，给出反馈

  最后加上`@WebServlet("/hello")`注解，指定访问的路径

  最后代码如下：

  ```java
  package com.xxx.servlet;
  
  import jakarta.servlet.ServletException;
  import jakarta.servlet.annotation.WebServlet;
  import jakarta.servlet.http.HttpServlet;
  import jakarta.servlet.http.HttpServletRequest;
  import jakarta.servlet.http.HttpServletResponse;
  
  import java.io.IOException;
  
  // 访问路径最后以/hello结尾
  @WebServlet("/hello")
  public class ServletLearn extends HttpServlet {
      @Override
      protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          // 处理请求的逻辑，这里是打印内容到控制台
          System.out.println("hello servlet!");
          // 通过流输出数据到浏览器
          resp.getWriter().write("Hello Servlet!");
      }
  }
  ```

- **Tomcat部署**

  在右上角Tomcat的下三角箭头进入Edit可以部署Tomcat 服务器

  ![image-20210307170734812](/images/image-20210307170734812.png)

  可以在这里看到访问URL，或者对URL进行一些修改

  ![image-20210307170938986](/images/image-20210307170938986.png)

- **运行及结果**

  运行代码，服务器也在运行，找到之前部署的URL，在末尾加上我们设置访问路径hello `http://localhost:8088/s01/hello`在浏览器中访问

  结果如下

  ![image-20210307171341851](/images/image-20210307171341851.png)

到此一个简单实例完成



# Servlet的继承体系和生命周期

Servlet 是服务 HTTP 请求并实现 **javax.servlet.Servlet** 接口的 Java 类。下面是Servlet接口定义的方法。

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
```

在上诉接口的方法中也蕴含了Servlet的**生命周期**，下面是Servlet生命周期的过程：

- Servlet 初始化后调用 **init ()** 方法。

- Servlet 调用 **service()** 方法来处理客户端的请求。

- Servlet 销毁前调用 **destroy()** 方法。

- 最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。

  整个Servlet工作的过程可以用下图描述

  ![image-20210308152733195](/images/image-20210308152733195.png)

- Servlet只初始化一次，它是单例的，只有一个实例，通过多线程访问。即Servlet是多线程单实例的
- 实例化过程中，先调用构造方法，再调用init方法，所以初始化操作可以覆盖写到init方法中，init 方法被设计成只调用一次。它在第一次创建 Servlet 时被调用，在后续每次用户请求时不再调用
- service() 方法是执行实际任务的主要方法，Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。
- destroy() 方法只会被调用一次，在 Servlet 生命周期结束时被调用，在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收

上诉描述还有一个重点，在实际调用service()方法时，service() 方法检查 HTTP 请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 doGet、doPost、doPut，doDelete 等方法。

但是我们的Servlet并没有实现这些方法，这时得看Servlet的**继承体系**

![image-20210308153321284](/images/image-20210308153321284.png)

所以，Web 应用程序开发人员通常编写 Servlet 来扩展 javax.servlet.http.HttpServlet，并实现 Servlet 接口的抽象类专门用来处理 HTTP 请求，我们所做的一般是继承HTTPServlet类，重写oGet、doPost定义自己的业务逻辑即可

这里可以做一个小验证，我们修改之前简单实例的代码

```java
@WebServlet("/hello")
public class ServletLearn extends HttpServlet {
    private static int count = 1;
    @Override
    public void init() throws ServletException {
        super.init();
        System.out.println("第"+count+"次调用init()");
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("hello servlet!"+count);
        resp.getWriter().write("Hello Servlet!"+count++);
    }
}
```

在网页上多次访问该资源，结果控制台情况如下

![image-20210308155311695](/images/image-20210308155311695.png)



# HTTPServletRequest对象

该对象之后简称Request对象，该对象主要作用是接收客户端发送过来的请求信息，比如请求参数、头信息等，我们的HttpServlet 的service方法中的形参就是HttpServletRequest的实例化对象，由Tomcat封装好传递过来。

我们主要需要取出该对象中的数据、进行分析、处理

## 请求信息

下列方法可以从request对象中获取相应的信息，其中，最重要的是前两个获取具体参数的方法

```java
package com.xxx.servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/method")
public class RequestMethod extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        // 常用方法

        // 获取指定名称的参数
        String  name = req.getParameter("name");
        System.out.println("uname:" + name);
        // 获取指定参数所用值
        String[] hobby = req.getParameterValues("hobby");
        if(hobby != null && hobby.length > 0)
        {
            System.out.print("hobby");
            for(String s : hobby)
                System.out.print(s + ",");
            System.out.println();
        }

        // 获取请求时URL，完整路径,从http开始，到？前结束
        String url = req.getRequestURL().toString();
        System.out.println("url:" + url);
        // 获取请求时部分路径，从站点名，到？前结束
        String uri = req.getRequestURI();
        System.out.println("uri:" + uri);
        // 获取请求时所有参数
        String str = req.getQueryString();
        System.out.println("query string:" + str);
        // 获取请求方式
        String method = req.getMethod();
        System.out.println("method:" + method);
        // 获取当前协议版本
        String protocol = req.getProtocol();
        System.out.println("protocol:" + protocol);
        // 获取项目站点名，项目对外访问路径
        String web = req.getContextPath();
        System.out.println(web);

    }
}
```

我们此时`http://localhost:8088/request/method?name=Lucy&hobby=sing&hobby=dance&hobby=rap`

结果如下：

![image-20210308172213170](/images/image-20210308172213170.png)

# ServConfig和ServerContext对象

## ServConfig接口



**request和response对象**就是封装HTTP的请求头和响应头，与外界交互

再学习Cookie和Session回话机制即可了

