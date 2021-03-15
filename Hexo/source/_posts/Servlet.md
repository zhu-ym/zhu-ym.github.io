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



## 文件下载demo

了解response后，需要学会利用其做一些小demo，比如文件下载、页面自动刷新、生成验证码图片等

下面是一个文件下载的小demo

首先可以在web应用中建立一个资源文件夹，作为存储供下载文件的文件夹，这里我命名为download

![image-20210312202353988](/images/image-20210312202353988.png)

设置一个download.jsp页面，打开这个界面可以呈现资源的下载链接

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>下载资源</title>
</head>
<body>
<a href="/response/do?fileName=捕获.JPG">图片</a>
<a href="/response/do?fileName=捕获.docx">文档</a>
</body>
</html>
```

效果如下

![image-20210312202703167](/images/image-20210312202703167.png)

下载文件的链接指向我们定义的一个servlet类，传递文件名作为参数

我们就在这个servlet类中进行下载处理，代码如下

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

@WebServlet("/do")
public class DownloadFile extends HttpServlet {


    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        download(req, resp);

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        download(req, resp);
    }
    
    protected void download(HttpServletRequest req, HttpServletResponse resp)throws ServletException, IOException{
        
        resp.setContentType("UTF-8");
        // 获取文件名
        String filename = req.getParameter("fileName");
        // 获取资源所在位置的决对路径
        String path = req.getServletContext().getRealPath("/download/" + filename);
        // 设置响应对象头，使浏览器以附件方式打开资源
        resp.setHeader("Content-Disposition","attachment;filename="+filename);
        
        // 下载资源的步骤
        FileInputStream in = new FileInputStream(path);
     
        ServletOutputStream out = resp.getOutputStream();
        int len = 0;
        byte[] bytes = new byte[1024];
        while((len = in.read(bytes)) > 0){
            out.write(bytes,0,len);
        }
        in.close();
        out.flush();
        out.close();
    }
}
```

其中下载资源步骤就是设置文件输入流，设置文件输出流、创建一个缓存数组，读写文件的过程



# Cookie和Session

我们认为会话是指客户端和web服务器之间连续发生的一系列请求和响应的过程，而Cookie和Session的作用就在于保存会话过程中产生的数据

## Cookie

 当用户通过浏览器访问Web服务器时，服务器会给客户端发送一些信息，这些信息会保存在Cookie中。这样，当浏览器再次访问服务器时，会在请求头中将Cookie发送给服务器，方便服务器对浏览器做出正确的响应。 

下面是一个简单的Cookie学习demo

首先做个一个post的登陆界面表单

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>load...</title>
</head>
<body>
<form method="post" action="co">
    用户名：<input name="uname" type="text"><br>
    密码：<input name="psd" type="password"><br>
    <button>登陆</button>
</form>

</body>
</html>
```

由于收到post方法的数据，所以重写doPost方法逻辑

```java
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet("/co")
public class CookieLearn extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {


        req.setCharacterEncoding("UTF-8");
        // 遍历cookie
        Cookie[] cookies = req.getCookies();
        for (Cookie cookie : cookies) {
            System.out.println(cookie.getName()+" " + cookie.getValue());
        }

        // 读取表单参数
        String uname = req.getParameter("uname");
        String psd = req.getParameter("psd");
        
       // 新建cookie
        Cookie cookie1 = new Cookie("uname",uname);
        Cookie cookie2 = new Cookie("psd",psd);
       
        // 设置cookie作用路径
        cookie1.setPath(req.getContextPath() );
        cookie2.setPath(req.getContextPath());

        // cookie 持久化时间
        cookie1.setMaxAge(100);
        cookie2.setMaxAge(100);
        
        // 添加cookie
        resp.addCookie(cookie1);
        resp.addCookie(cookie2);

    }
}
```

当两次登陆后，打印结果如下

![image-20210315105919816](/images/image-20210315105919816.png)

可见第一次登陆没有新建cookie的信息，第二次就多了用户名和密码信息

对于上诉cookie，除了新建和添加cookie比较固定，还有一些关键点

- **cookie作用路径**

  可以通过`setPath()`方法设置Cookie的作用域，即在那些地方可以看到cookie，可以作用域当前目录及其子目录

  所以主要如下

  ```java
  默认不设置为新建Cookie的目录
  
  // 当前Tomcat下的所有web项目
      setPath("/");
  // 具体目录
      setPath(req.getContextPath() + "/load.jsp");
  	setPath("/cookie/load.jsp")
  ```

  如果需要跨域访问或者说是不同Tomcat访问同一个Cookie，就需要用到`setDomain()`方法

- **cookie持久化时间**

  cookie的默认存活时间是浏览器关闭，可以通过`setMaxAge()`方法设置其存活时间，将Cookie存到硬盘中，注意单位是秒，这时可以关闭浏览器后打开也能获取

## Session

### **Session底层原理**

- 浏览器访问某个Servlet，这时如果服务器要从请求对象中获取Session对象（第一次获取也是创建），那么服务器会为这个Session对象创建一个id：JSESSIONID

- 同时在对浏览器的响应过程中，这个Session会将JSESSIONID这个id以Cookie形式回送给客户端浏览器，记住，这时候Cookie服务器没有设置有效时间，因此是存在浏览器的缓存中，而不是在硬盘文件。

- 当用户继续在这个会话过程中访问其他Servlet，这时候这个Servlet再从请求对象中获取Session对象，注意这时候获取Session对象是从浏览器发来的请求中查询是否有名为JSESSIONID的这个Cookie，如果有，那么这个Session就不用再创建，而是直接根据查询服务器中这个相同JSESSIONID值的Session，换句话说就可以取得之前存在这个Session中的数据。

即Session的底层原理是通过**Cookie实现**的

### **Session常用方法**

下面是一个创建Session，并且利用Session的实例

首先创建一个Servlet，在其内部可以创建Session

```java
package com.xxx.servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;

import java.io.IOException;
import java.util.Enumeration;

@WebServlet("/se")
public class SessionBase extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("UTF-8");
        HttpSession session = req.getSession();
        System.out.println(session.getId());
        if(session.isNew()){
        String uname = req.getParameter("uname");
        String hobby = req.getParameter("hobby");
        session.setAttribute("hobby",hobby);
        session.setAttribute("uname",uname);}
        else{
            Enumeration<String> it = session.getAttributeNames();
            while(it.hasMoreElements()){
                String name = it.nextElement();
                System.out.println(name + ": " + session.getAttribute(name));
            }
        }
        resp.sendRedirect(req.getContextPath() + "/show.jsp");
    }
}
```

最后在重定向的页面中查看Session携带的数据

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>show</title>
</head>
<body>
<%
    HttpSession session1 = request.getSession();
    if(!session1.isNew()){
        Enumeration<String> it = session1.getAttributeNames();
        while(it.hasMoreElements()){
            String name = it.nextElement();
            response.getWriter().println(name + ": " + session1.getAttribute(name));
        }
    }
%>
</body>
</html>
```

两次访问`http://localhost:8088/session/se?uname=andy&hobby=rap`

`http://localhost:8088/session/se`

第一次带参，第二次不带参结果均如下

![image-20210315153733059](/images/image-20210315153733059.png)

控制台结果如下

![image-20210315153830459](/images/image-20210315153830459.png)

可见其确实靠JSESSIONID作为标识





### **对Session进行设置**

我们已经知道Session是靠名为JSESSIONID的Cookie的ID值标识定位的，所以Session的生命周期和作用域都和这个Cookie有关。

由于这个cookie没有设置`setMaxAge`，因此这个cookie只存在于浏览器的缓冲，浏览器关闭即被销毁。

又由于这个cookie不设置路径默认为新建Cookie的目录,所以可能访问这个web应用的其他资源就无法再使用Session了，得注意一下路径问题

综上，如果需要对这些进行修改，主要是对JSESSIONID的Cookie进行修改

通过`Cookie cookie = new Cookie("JSESSIONID", session.getId());`就可以对此Cookie进行修改。

此外当不关闭浏览器用户不进行操作时，session默认维持30分钟，可以在Tomcat的`web.xml`进行全局设置修改，或者在每个web应用中的web.xml文件中自定义添加`<session-config>`和`<session-timeout>`进行设置。这里得注意**设置的cookie有效时间就不要超过服务器默认的`session-timeout`时间。**

## 表单重复提交demo

学习完Cookie和Session后，我们考虑一个运用场景

在平时开发中，如果网速比较慢的情况下，用户提交表单后，发现服务器半天都没有响应，那么用户可能会以为是自己没有提交表单，就会再点击提交按钮重复提交表单，我们在开发中必须防止表单重复提交。

可以在表单提交后，将提交按钮设置为不可用，但是如果用户刷新界面、或者利用回退提交表单，该场景就很难在客户端解决

这时可以用Session在服务端解决该问题

**思路：**

在服务器端生成一个唯一的随机标识号，专业术语称为Token(令牌)，同时在当前用户的Session域中保存这个Token。然后将Token发送到客户端的Form表单中，在Form表单中使用隐藏域来存储这个Token，表单提交的时候连同这个Token一起提交到服务器端，然后在服务器端判断客户端提交上来的Token与服务器端生成的Token是否一致，如果不一致，那就是重复提交了，此时服务器端就可以不处理重复提交的表单。如果相同则处理表单提交，处理完后清除当前用户的Session域中存储的标识号。

**步骤：**

- 创建一个FormServlet类，生成token保存到Session中并传递到表单

  ```java
  package com.xxx.servlet;
  
  import jakarta.servlet.ServletException;
  import jakarta.servlet.annotation.WebServlet;
  import jakarta.servlet.http.HttpServlet;
  import jakarta.servlet.http.HttpServletRequest;
  import jakarta.servlet.http.HttpServletResponse;
  
  import java.io.IOException;
  
  @WebServlet("/form")
  public class FromServlet extends HttpServlet {
      @Override
      protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
  
          String token = Token.getInstance().getToken();
          System.out.println(token);
          req.getSession().setAttribute("token",token);
          req.getRequestDispatcher("/form.jsp").forward(req,resp);
      }
  }
  ```

- 在表单中传递隐藏数据

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>提交界面</title>
  </head>
  <body>
  <form method="post" action= "${pageContext.request.contextPath}/doform" >
      <input type="hidden" name="token" value="<%=request.getSession().getAttribute("token") %>">
      提交数据:<input type="text" name="userdata">
      <input type="submit" value="提交">
  </form>
  </body>
  </html>
  ```

- 对表单数据进行处理，判重

  ```java
  @WebServlet("/doform")
  public class DoForm extends HttpServlet {
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          if(isRepeated(req)){
              System.out.println("你已重复提交数据");
          }
          else{
              System.out.println("正在处理提交数据");
          }
          req.getSession().removeAttribute("token");
      }
  
  
      private boolean isRepeated(HttpServletRequest request){
          String client_token = request.getParameter("token");
          if(client_token == null)
              return true;
          String server_token = (String) request.getSession().getAttribute("token");
          if(server_token == null)
              return true;
          return !client_token.equals(server_token);
      }
  }
  ```

- 用一个Token类来生成token

  ```java
  package com.xxx.servlet;
  
  import java.util.Random;
  
  public class Token {
      private static final Token token = new Token();
      private Token(){
      }
  
      public static Token getInstance() {
          return token;
      }
      public String getToken(){
          return (System.currentTimeMillis() + new Random().nextInt(999999999)) + "";
      }
  }
  ```

  



![image-20210315171953967](/images/image-20210315171953967.png)

# ServletConfig和ServerContext对象

## ServletConfig

当servlet配置了初始化参数后，web容器在创建servlet实例对象时，会自动将这些初始化参数封装到ServletConfig对象中，并在调用servlet的init()方法时，将ServletConfig 对象传递给servlet。进而，通过ServletConfig对象就可以得到当前servlet的初始化参数。

## ServerContext

每一个web应用都有且仅有一个ServletContext对象，其与应用程序相关，在Web容器启动时为web应用创建

其主要用来作为**域对象共享数据（整个程序中）**和保存了一些**应用程序相关信息**或者**读取资源文件**

**常用方法**

```java
@WebServlet("/con")
public class ContentLearn extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext c1= req.getServletContext();
        ServletContext c2 = getServletContext();
        ServletContext c3 = req.getSession().getServletContext();
        ServletContext c4 = getServletConfig().getServletContext();
        
        // 结果为true
        System.out.println(c1 == c2 && c3 == c4 && c1 == c4);

        System.out.println("服务器信息：" + c1.getServerInfo());
        System.out.println("真实路径：" + c1.getRealPath("/"));
    }
}
```

ServletContext对象的获取比较多元化，有直接获取或者利用其它域对象获取，一般主要是利用其来获取一些信息，很少用其来共享数据，其共享数据的方法命名和Request和Session一致



**两者的区别**

- 定义

ServletConfig：Servlet的配置对象，容器在初始化Servlet时通过它传递信息给Servlet。

ServletContext：上下文对象，提供了一系列方法供Servlet与Web容器交互。

- 创建时机

 ServletConfig：在容器初始化Servlet的时候，并为其提供上下文初始化参数的名/值对的引用。

 ServletContext：容器启动的时候，并为其提供Servlet初始化参数的名/值对的引用。

- 作用范围（可见性）

 ServletContext：每个Web应用一个ServletContext。

 ServletConfig：每个Web应用的每个Servlet一个ServletConfig。







