---
title: JDBC
typora-root-url: ../
date: 2021-03-18 15:12:33
tags:
- JDBC
- 数据库
categories:
- [Java语法,进阶]
- [数据库]
---

简单介绍Java的JDBC，并了解一下连接池

<!--more-->

# 建立一个JDBC实例

- 新建一个简单的Maven项目

- 在pom.xml文件中添加数据库驱动的依赖

  可以在该网站查询https://mvnrepository.com/，最后查询的我mysql 8.0.23对对应的数据库驱动的依赖如下

  ```xml
  <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>8.0.23</version>
       </dependency>
  ```

- 编写一个JDBC代码

  ```java
  import java.sql.*;
  
  public class jdbc {
  
      public static void main(String[] args) throws ClassNotFoundException, SQLException {
  
  
          // 数据库url，tablelearn为数据库的名字
          String url = "jdbc:mysql://localhost:3306/tablelearn?serverTimezone=GMT%2B8";
          String user = "root";
          String psd = "root";
  
          // 注册驱动
          Class.forName("com.mysql.cj.jdbc.Driver");
          // 连接数据库
          Connection connection = DriverManager.getConnection(url,user,psd);
          //   实例化statement对象
          Statement statement = connection.createStatement();
          //定义数据库查询语句
          String sql = "select id,name from hero";
          // 执行语句返回结果
          ResultSet resultSet = statement.executeQuery(sql);
          // 查询结果处理
          while(resultSet.next()){
              String name = resultSet.getString("name");
              int id = resultSet.getInt("id");
              System.out.println("id: " + id + " name: " + name);
          }
          resultSet.close();
          statement.close();
          connection.close();
      }
  }
  ```

  结果

  ![image-20210318104619541](/_posts/JDBC.assets/image-20210318104619541.png)

  总结编写一个JDBC代码主要需要以下几步：

  - 加载驱动
  - 连接数据库  DriverManager
  - 获得执行sql的对象 Statement
  - 获得返回结果集  ResultSet
  - 数据处理
  - 释放连接



此外下面有两点需要注意：

- **PreparedStatement**

正常来说，我们用Statement执行增删查改等操作，就如上诉案例所示，但Java中还有`PreparedStatement`也是用来执行sql语句的，与创建Statement不同的是，需要根据sql语句创建`PreparedStatement`,除此之外，还能够通过设置参数，指定相应的值，而不是Statement那样使用字符串拼接

此外`PrepareStatement`还有两个优点

- `PreparedStatement`有预编译机制，性能比Statement更快
- `PreparedStatement`预编译可以防止SQL注入式攻击

具体见下面一个小例子

```java
// 利用PreparedStatement每一个下标就是一个参数，从1开始
        String sql = "insert into hero values(null,?,?,?)";
        Statement statement = connection.createStatement();
        PreparedStatement preparedStatement = connection.prepareStatement(sql);

        long t1 = System.currentTimeMillis();
        for (int i = 0; i < 1000; i++) {

            // Statement需要进行字符串拼接，可读性和维修性比较差
            String sql0 = "insert into hero values(null," + "'提莫'" + ","
                    + 313.0f + "," + 50 + ")";
            statement.execute(sql0);
        }
        long t2 = System.currentTimeMillis();
        System.out.println("Statement时间： " + (t2 - t1));

        long t3 = System.currentTimeMillis();
        for (int i = 0; i < 1000; i++) {
            preparedStatement.setString(1, "提莫");
            preparedStatement.setFloat(2, 313.0f);
            preparedStatement.setInt(3, 50);
            preparedStatement.execute();
        }
        long t4 = System.currentTimeMillis();
        System.out.println("preparedStatement: " + (t4 - t3));
```

结果如下

![image-20210318154537497](/images/image-20210318154537497.png)

反而发现`preparedStatement`更慢，因为**mysql不支持`preparedStatement`**



- **executeUpdate**

`executeUpdate`同`execute`：都可以执行增加，删除，修改;但是`execute`还可以执行查询语句.

`execute`返回boolean类型，true表示执行的是查询语句，false表示执行的是insert,delete,update等等
`executeUpdate`返回的是int，表示有多少条数据受到了影响

# 事务操作

- **事务**

  可以利用connext的下列两个方法规定事务的多组操作,这些操作要么都执行，要么都不执行

  ```java
  setAutoCommit(false);
  ....
  commit();
  ```

   下面是一段代码

    ```java
  import java.sql.*;
  public class Transaction {
      public static void main(String[] args) throws ClassNotFoundException {
          // 数据库url，tablelearn为数据库的名字
          String url = "jdbc:mysql://localhost:3306/tablelearn?serverTimezone=GMT%2B8";
          String user = "root";
          String psd = "root";
          // 注册驱动
      	Class.forName("com.mysql.cj.jdbc.Driver");
  
      	// try-with-resources
     	 try(Connection connection = DriverManager.getConnection(url,user,psd);
          Statement statement = connection.createStatement()){
  
          // 事务开始点
          connection.setAutoCommit(false);
          String sql1 = "insert into hero values(null,"+"'凯'"+","+400+","+120+")";
          String sql2 = "insert into hero values(null,"+"'刘备'"+","+400+","+100+")";
          String sql3 = "select id,name from hero";
          statement.execute(sql1);
          statement.execute(sql2);
          ResultSet resultSet = statement.executeQuery(sql3);
          // 事务结束点
          connection.commit();
          while(resultSet.next()){
              String name = resultSet.getString("name");
              int id = resultSet.getInt("id");
              System.out.println("id: " + id + " name: " + name);
          }
  
      }
      catch (SQLException e){
          e.printStackTrace();
      }
  }}
    ```

  上诉操作必须在3个操作都成功才能算是完成，这样不会因为某个操作错误而修改了数据库

# 连接池

有多个线程，每个线程都需要连接数据库执行SQL语句的话，那么每个线程都会创建一个连接，并且在使用完毕后，关闭连接。

创建连接和关闭连接的过程也是比较消耗时间的，当多线程并发的时候，系统就会变得很卡顿。

同时，一个数据库同时支持的连接总数也是有限的，如果多线程并发量很大，那么数据库连接的总数就会被消耗光，后续线程发起的数据库连接就会失败。

类似于线程池的思想连接池在使用之前，就会创建好一定数量的连接。如果有任何线程需要使用连接，那么就从连接池里面借用，，而不是自己重新创建使用完毕后，又把这个连接归还给连接池供下一次或者其他线程使用。倘若发生多线程并发情况，连接池里的连接被借用光，那么其他线程就会临时等待，直到有连接被归还回来，再继续使用。整个过程，这些连接都不会被关闭，而是不断的被循环使用，从而节约了启动和关闭连接的时间

下面是一个简易实现连接池的代码

```java
package jdbc;
  
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
  
public class ConnectionPool {
  
    List<Connection> cs = new ArrayList<Connection>();
  
    int size;
  
    public ConnectionPool(int size) {
        this.size = size;
        init();
    }
  
    public void init() {
          
        //这里恰恰不能使用try-with-resource的方式，因为这些连接都需要是"活"的，不要被自动关闭了
        try {
            Class.forName("com.mysql.jdbc.Driver");
            for (int i = 0; i < size; i++) {
                Connection c = DriverManager
                        .getConnection("jdbc:mysql://127.0.0.1:3306/how2java?characterEncoding=UTF-8", "root", "admin");
  
                cs.add(c);
  
            }
        } catch (ClassNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (SQLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
  
    public synchronized Connection getConnection() {
        while (cs.isEmpty()) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        Connection c = cs.remove(0);
        return c;
    }
  
    public synchronized void returnConnection(Connection c) {
        cs.add(c);
        this.notifyAll();
    }
  
}
```

Java编写连接连接池需要实现`DataSource`接口

```java
public interface DataSource  extends CommonDataSource, Wrapper {

  Connection getConnection() throws SQLException;
  Connection getConnection(String username, String password)
    throws SQLException;
}
```

Tomcat默认使用了开源的DBCP连接池

# ORM和DAO

`ORM=Object Relationship Database Mapping`

即对象和关系数据库的映射

比如下列根据id返回一个对象

```java
package jdbc;
   
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
  
import charactor.Hero;
   
public class TestJDBC {
   
    public static Hero get(int id) {
        Hero hero = null;
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
 
        try (Connection c = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/how2java?characterEncoding=UTF-8","root", "admin");
            Statement s = c.createStatement();) {
 
            String sql = "select * from hero where id = " + id;
   
            ResultSet rs = s.executeQuery(sql);
   
            // 因为id是唯一的，ResultSet最多只能有一条记录
            // 所以使用if代替while
            if (rs.next()) {
                hero = new Hero();
                String name = rs.getString(2);
                float hp = rs.getFloat("hp");
                int damage = rs.getInt(4);
                hero.name = name;
                hero.hp = hp;
                hero.damage = damage;
                hero.id = id;
            }
   
        } catch (SQLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return hero;
   
    }
   
```

DAO=**D**ata**A**ccess **O**bject

数据访问对象

实际上就是运用了ORM中的思路，把数据库相关的操作都封装在这个类里面，其他地方看不到JDBC的代码

DAO接口

```java
public interface DAO{
    //增加
    public void add(Hero hero);
    //修改
    public void update(Hero hero);
    //删除
    public void delete(int id);
    //获取
    public Hero get(int id);
    //查询
    public List<Hero> list();
    //分页查询
    public List<Hero> list(int start, int count);
}
```

