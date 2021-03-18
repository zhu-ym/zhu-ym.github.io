---
title: Mysql
typora-root-url: ../
date: 2021-03-16 10:10:34
tags:
- Mysql
categories:
- [数据库]
---

了解MySQL

<!--more-->

# MySQL概念

数据库（Database）是按照数据结构来组织、存储和管理数据的仓库。

每个数据库都有一个或多个不同的 API 用于创建，访问，管理，搜索和复制所保存的数据。

我们也可以将数据存储在文件中，但是在文件中读写数据速度相对较慢。

所以，现在我们使用关系型数据库管理系统（RDBMS）来存储和管理大数据量。所谓的关系型数据库，是建立在关系模型基础上的数据库，借助于集合代数等数学概念和方法来处理数据库中的数据。

RDBMS 即关系数据库管理系统(Relational Database Management System)的特点：

- 1.数据以表格的形式出现
- 2.每行为各种记录名称
- 3.每列为记录名称所对应的数据域
- 4.许多的行和列组成一张表单
- 5.若干的表单组成database

MySQL 是最流行的关系型数据库管理系统，在 WEB 应用方面 MySQL 是最好的 RDBMS(Relational Database Management System：关系数据库管理系统)应用软件之一。

# 安装MySQL

- **下载安装包**

    [下载链接](https://dev.mysql.com/downloads/windows/installer/8.0.html)，点击链接后，选择 第二个（为安装版），点击下载，可以选择不注册下载

- **具体安装教程**

  该部分是在网上找的，我把自己安装参考的博客转载过来

  [参考博客](https://www.cnblogs.com/2020javamianshibaodian/p/mysql8020anzhuangjiaocheng.html)

- **客户端选择**

  正常来说安装完mysql可以使用客户端，这里我使用官方的Mysql  WorkBench，按照上诉过程安装既有

  据说最好用的客户端是 Navicat，但不免费

# 基础使用

基于Mysql  WorkBench使用一些常见命令



## 表相关

- 创建一个数据库和表

  ```mysql
  create database tablelearn;
  use tablelearn;
  CREATE TABLE hero (
    id int(11) AUTO_INCREMENT,
    name varchar(30) ,
    hp float ,
    damage int(11) ,
    PRIMARY KEY (id)
  )  DEFAULT CHARSET=utf8;
  ```

  

- 修改表

  ```mysql
  
  ```

  

## 增删查改

- 插入数据

  ```mysql
  insert into hero values (null, '盖伦', 616, 100)
  ```

- 修改数据

  ```mysql
  update hero set name = "典韦" where id = 2
  ```

- 查询数据

  ```mysql
  select * from hero limit 0,2
  ```

- 删除数据

  ```mysql
  delete from hero where id = 1
  ```

  

# Mysql和IDEA的连接

该连接的效果仅仅是使IDEA充当Mysql  WorkBench可视化的功能，真正要在代码中使用还得用JDBC

- 点`view->tool windows->database`

  之后点击database的+号，添加mysql源

  ![image-20210317164459805](/images/image-20210317164459805.png)

- 填入个人信息，即下载主要是下载时设置的密码，其他可以保持默认

  ![image-20210318095103092](/images/image-20210318095103092.png)

  如果用的是mysql 8 以上版本需要注意时区问题，要在Url加上`?serverTimezone=GMT%2B8`

  最后选择`test Connection`即可

  第一次注意在左下角点击下载驱动

- 连接成功

  此时可以尝试在IDEA中编辑数据库、表



# JDBC

在代码中使用数据库数据

## 建立一个JDBC实例

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

  ![image-20210318104619541](/images/image-20210318104619541.png)

  总结编写一个JDBC代码主要需要以下几步：

  - 加载驱动
  - 连接数据库  DriverManager
  - 获得执行sql的对象 Statement
  - 获得返回结果集  resultSet
  - 数据处理
  - 释放连接

## 特殊操作

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
    }
}
    ```

在事务中的多个操作，**要么都成功，要么都失败**
通过setAutoCommit(false);**关闭自动提交**
使用 commit();进行**手动提交**



# MySQL索引

Mysql服务器逻辑架构如下

![image-20210317201823243](/images/image-20210317201823243.png)

- 其中最上服务器层主要负责连接管理与安全性，MySQL客户端与服务端的通信连接管理以及用户名密码、对数据库及表的权限校验，都是在这一层完成的
- 第二层包括MySQL核心服务功能，像查询解析、分析、优化、缓存、所有的内置函数以及所有跨存储引擎的功能（存储过程、触发器、视图等）都在这一层
- 第三层就是存储引擎层，存储引擎负责数据的存储和提取，服务器通过一套标准的API可以和不同的存储引擎进行通信，这些API屏蔽了不同存储引擎之间的差异

而mysql中的索引是在最底层的存储引擎层实现的，不同存储引擎可能使用不同的存储结构存储索引。



## B+Tree索引

InnoDB是mysql的默认事务型引擎，主要采用**B+Tree**的结构存储索引

B+Tree主要是由B树演变而来，相较于B-Tree，主要有以下区别：

- B树每个节点都存储数据，所有节点组成这棵树，B+树只有叶子节点存储数据，所有非叶节点只起到索引作用。

- B+所有的叶子结点使用链表相连
- 关键字和子树范围不一致

首先最大的区别B+树只有叶子节点存储数据，其一个关键字代表一个子树的最值，并指向该子树，所以造成了第三点区别

其次正是由于B+树只有叶子节点存储数据，所以其所有的叶子结点使用链表相连才有意义，这给B+数带来了很大的优点：

- 所有的叶子结点使用链表相连，**便于区间查找和遍历**
- B+树叶子节点顺序存储，B树则需要进行每一层的递归遍历，相邻的元素可能在内存中不相邻，所以**缓存命中性**没有B+树好。

数据库索引是存储在磁盘上的，考虑磁盘IO的影响，无论是B+树还是B树，都能**减少IO次数**，特别数据量大时，就不能把整个索引全部加载到内存了，只能逐一加载每一个磁盘页（对应索引树的节点），对于树来说，IO次数就是树的高度，而m的大小取决于磁盘页的大小。
由于采用了B+Tree结构，mysql非常适用于**全键值、键值范围或键前缀查找**

值得注意，其中mysql键前缀查找只适用于**最左前缀的查找**，所以对索引列的顺序要求很高，不可能跳过第一列索引去查找其他列索引，第一列索引也是最左前缀查找，比如以名字为索引的索引中，很难查找以名字的最后一个字为结尾的人

## 哈希索引

哈希索引基于hash表实现，只有在精确匹配索引所有列的查询才有效，即**全键值查找**

采用Hash索引时，对于每行数据，存储引擎都会根据所有索引列值计算一个哈希码，哈希索引将所有的哈希码存储在索引中，同时哈希表中保存指向每个数据行的指针

在mysql中只有Memory引擎显示支持哈希索引，并且支持非唯一值哈希索引（即可以存在hash冲突）

由于hash表的特性其在**全键值查找**中有很快的速度，但是不能支持**范围查询**或者**部分索引列查询**

InnoDB引擎有一个特殊的功能加做**自适应哈希索引**，当某些索引值被频繁使用时，会在内存中基于B+Tree的索引上再创建一个哈希索引，这个是自动的、内部的行为。



## 索引的适应范围

总的来说，索引的目的是为了快速定位到表的指定位置，但是除此之外，索引还有以下几个优点：

- 大大减少服务器需要扫描的数据量
- 使服务器避免排序和临时表
- 可以将随机I/O变成顺序I/O

但是注意索引页带来了额外的开销，只有其快速定位的好处大于这些开销时，索引才是有效的，因此索引主要用于中到大型表。对于小表，简单的全表扫描更高效，对于特大的表维护索引的开销代价更大，需要类似于分区技术等技术直接区分出查询需要的一组数据



## 高性能索引策略

