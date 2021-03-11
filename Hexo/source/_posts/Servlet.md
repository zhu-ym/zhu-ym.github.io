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

## Request常见方法

**常见方法**

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



**获取头信息**

`equest.getHeader(String s)`可以获取浏览器传递过来的头信息。
`request.getHeaderNames() `获取浏览器所有的头信息名称，根据头信息名称就能遍历出所有的头信息

下面是一些常见的头信息

host: 主机地址
user-agent: 浏览器基本资料
accept: 表示浏览器接受的数据类型
accept-language: 表示浏览器接受的语言
accept-encoding: 表示浏览器接受的压缩方式，是压缩方式，并非编码
connection: 是否保持连接
cache-control: 缓存时限

```java
@WebServlet("/method")
public class RequestMethod extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        // 获取头部信息
        Enumeration<String> names = req.getHeaderNames();
        while(names.hasMoreElements()){
            String name = names.nextElement();
            String info = req.getHeader(name);
            System.out.println(name + ": " + info);
            resp.getWriter().println(name + ": " + info);
        }

    }
}
```

![image-20210311134606550](/images/image-20210311134606550.png)

## 乱码问题

原来的TomCat服务器有中文乱码问题，对于Request

- TomCat 7 以下 get 方法和post方法均有中文乱码问题,默认情况下，使用`getParameter()`请求参数时，默认使用的是ISO-8859-1编码，不支持中文

  可以采用`req.setCharacterEncoding()`方式设置正文即实体编码，但此方法只对实体有效，也即只对post方法有效；get方法的参数在url中，得设置server配置文件或者利用`new String(name.getBytes("ISO-8859-1"), "UTF-8");`将参数转化为字节码后，再编码

- TomCat 8 已经将get 方法的的url编码方法默认设为UTF-8，不会乱码，只需要考虑post方法
- TomCat 10 当我测试乱码问题时发现无论是post还是get方法都很正常，查找文档发现使用`<request-character-encoding>`和 `<response-character-encoding>`in `conf/web.xml`将默认请求和响应字符编码设置为UTF-8，所以可能对于request不用再考虑中文乱码问题了

## 请求转发

请求转发属于服务器跳转，相当于方法调用，在执行当前文件的过程中转向执行目标文件，两个文件(当前文件和目标文件)属于同一次请求，前后页共用一个request，这时可以通过下列两个方法传递、获取一些参数

```java
req.getRequestDispatcher("/loadfail.jsp").forward(req,resp);// 请求转发命令
request.setAttribute() 设置参数
request.getAttribute() 获取参数
```

值得注意的是该跳转得在web应用内，即站内跳转，可以跳转到一个servlet、html、jsp，当然如果要使用数据的话，不能跳转到html，因为其是静态页面

## 登陆界面制作

这里简单利用表单和请求转发做一个小demo

首先制作一个登陆的表单，利用post方法将其参数传递到我们的`method`Servlet中，我们再检测用户参数，利用请求转发跳转到登陆成功或者失败的界面

登陆表单

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登录界面</title>
</head>
<body>
<form action="method" method="post">
   用户名： <input type="text" name="uname"><br>
    密码：<input type="password" name="psd"><br>
    <button>登陆</button>
</form>
</body>
</html>
```

method逻辑处理和请求转发

```java
@WebServlet("/method")
public class RequestMethod extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        String name = req.getParameter("uname");
        String psw = req.getParameter("psd");
        System.out.println("uname : " + name );
        System.out.println("psw : " + psw );
        req.setAttribute("name",name);
        if(name == null || psw == null){

            req.getRequestDispatcher("/loadfail.jsp").forward(req,resp);
        }
        else{
            req.getRequestDispatcher("/loadok.jsp").forward(req,resp);
        }
    }
}
```

登陆成功或者失败界面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登陆成功</title>
</head>
<body>
<%
    String name = (String) request.getAttribute("name");
    System.out.println(name + request.getParameter("psd"));
    response.getWriter().println(name + "你好！");
%>
</body>
</html>
```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登陆失败</title>
</head>
<body>
<h1>登陆失败</h1>

</body>
</html>
```

登陆界面

![image-20210311151913935](/images/image-20210311151913935.png)

请求跳转界面，注意地址没有改变

![image-20210311151940188](/images/image-20210311151940188.png)



# HTTPServletRespose对象

Web服务器收到客户端的http请求，会针对每一次请求，分别创建一个用于代表请求的request对象、和代表响应的response对象。获取网页提交过来的数据，只需要找request对象就行了。要向网页输出数据，只需要找response对象。

HttpServletResponse对象代表服务器的响应。这个对象中封装了向客户端发送数据、发送响应头，发送响应状态码的方法。

## 消息响应

Response向客户端发送数据主要使用下面两种方法

```
getOutputStream() 该方法用于返回Servlet引擎创建的字节输出流对象，Servlet程序可以按字节形式输出响应正文。   
getWriter() 该方法用于返回Servlet引擎创建的字符输出流对象，Servlet程序可以按字符形式输出响应正文。
```

需要注意

- getOutputStream()和getWriter()这两个方法互相排斥，调用了其中的任何一个方法后，就不能再调用另一方法。

- getOutputStream()返回的字节输出流对象，类型为：ServletOutputStream，直接输出字节数组中的二进制数据。

- getWriter()方法将Servlet引擎的数据缓冲区包装成PrintWriter类型的字符输出流对象后返回，PrintWriter对象只能输出字符文本内容

## 乱码问题

response在响应时也可能会有乱码问题，其原因在于服务器和客户端的编码方式不一致

同request一样，我们可以使用`setCharacterEncoding()`方式设置正文即实体编码为UTF-8,但是如果客户端没有使用相同的编码也会出现乱码问题

```java
@WebServlet("/output")
public class OutPut extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setCharacterEncoding("UTF-8");
        resp.getWriter().println("你好");
    }
}
```

![image-20210311164049627](/images/image-20210311164049627.png)

可以通过设置响应对象头，确保浏览器或者客户端的编码方式也为"UTF-8"

```java
@WebServlet("/output")
public class OutPut extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setCharacterEncoding("UTF-8");
        resp.setHeader("content-type", "text/html;charset=UTF-8");
        resp.getWriter().println("你好");
    }
}
```

![image-20210311164317137](/images/image-20210311164317137.png)

更简洁的方法，还是利用`setContentType("text/html;charset=UTF-8")`,该方法可以控制服务器和浏览器的编码方式

```java
@WebServlet("/output")
public class OutPut extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().println("你好");
    }
}
```

之后，我并没有设置`resp.setCharacterEncoding("UTF-8")`,仅控制了客户端的编码也是正常的，可见TomCat 10 以后应该默认是UTF-8

## 重定向

利用`resp.sendRedirect(String s)`,可以实现重定向

重定向是指一个web资源收到客户端请求后，通知客户端去访问另外一个web资源，这称之为请求重定向

重定向与请求转发主要区别：在于是否能跨域以及请求转发是一次请求，重定向是两次请求 





# ServConfig和ServerContext对象

## ServConfig接口



再学习Cookie和Session回话机制即可了

