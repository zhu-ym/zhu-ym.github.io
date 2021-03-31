---
title: Mybatis
typora-root-url: ../
date: 2021-03-23 09:21:54
tags:
- MyBatis
categories:
- [数据库]
- [中间件]
---

学会使用MyBatis来简化JDBC开发

<!--more-->

# MyBatis简介

## 简介

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）访问修改数据库中的记录。

其简单原理在于其内部封装了JDBC。Mybatis是通过xml或注解的方式将需要执行的各种statement配置起来。通过Java对象和statement中的sql动态参数映射生成最终执行的sql语句，最终由Mabtais框架执行sql并将结果映射为Java对象并返回。MyBatis 支持定制化 SQL、存储过程以及高级映射。MyBatis 是可以双向映射的，可以将数据集映射为Java对象，也可以将Java对象映射为数据库中的记录。

总的来说，MyBatis有以下优点：

- 简单便捷
- 消除了JDBC大量冗余的代码，不需要手动开关连接
- 能够与Spring很好的集成；
- 解除sql与程序代码的耦合，便于统一管理和优化，并可重用。
- 提供映射标签，支持对象与数据库的ORM字段关系映射。

## 原理

MyBatis工作原理或者说是工作流程如下所示

![image-20210323095338627](/images/image-20210323095338627.png)

Mybatis应用程序通过**SqlSessionFactoryBuilder**从**mybatis-config.xml**配置文件（也可以用Java注解方式配置）来构建**SqlSessionFactory**（一般设置为单例模式）。然后**SqlSessionFactory**实例一般在一个工具类里面，可以返回一个**SqlSession**，再通过**SqlSession**实例获得`Mapper`对象并运行`Mapper`映射的SQL语句，完成对数据库的CRUD和事务提交，之后**关闭SqlSession**。
综上，`SqlSessionFactoryBuilder`的作用在于生成一个`SqlSessionFactory`工厂实例，而`SqlSessionFactory`作用在于产生`SqlSession`对象，这里的`SqlSession`相当于JDBC中的`Connection`,底层就封装了JDBC连接,是个持久化对象。



# 简单MyBatis实例

开发环境

- IDEA 2021
- Maven 3.6.3
- Mysql 8.2.3
- MyBatis
- Junit

个人最终文件目录如下，下文生成的文件、类均在对应目录下生成

![image-20210323144235237](/images/image-20210323144235237.png)

**1.数据库准备**

利用`Mysql  WorkBench`,创建一个学生表，并且插入几条数据

```sql
create database mybatis;
use mybatis;
create table user (
id int(8) not null,
name varchar(20) default null,
psd varchar(30) default null,
sex char(2),
primary key(id)
)engine=InnoDB default charset=utf8;
insert into user values(1,'张三','123456','女');
insert into user values(2,'李四','23456','男');
insert into user values(3,'王五','3456','女');
```

**2.导入依赖**

创建Maven工程文件，在[Maven查询](https://mvnrepository.com/)中找到最新的MyBatis依赖,再加上之前的mysql驱动的依赖和Junit进行测试

```xml
<!--mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.3</version>
</dependency>
<!--mysql driver-->	
 <dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
     <version>8.0.23</version>
 </dependency>
<!-- junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>

```

**3.配置mybatis-config.xml文件**

在resources文件中添加以下mybatis-config.xml配置文件,并把以下内容复制进去

XML 配置文件中包含了对 MyBatis 系统的核心设置，包括获取数据库连接实例的数据源（DataSource）以及决定事务作用域和控制方式的事务管理器（TransactionManager）。后面会再探讨 XML 配置文件的详细内容，这里先给出一个简单的示例：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
</configuration>
```

这里主要修改以下driver，自己mysql数据库的url和登陆账号密码即可,注意mysql 8 以上有时区问题

```xml
<property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?serverTimezone=GMT%2B8&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
```

以上信息可以确保在MyBatis可以和mysql中我们新建的数据库建立连接

**4.编写MyBatis工具类**

主要是通过通过`SqlSessionFactoryBuilder`从`mybatis-config.xml`配置文件来构建`SqlSessionFactory`，之后里getter方法可以生成一个`SqlSession`对象

```java
package com.xxx.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class MybatisUtils {
    private  static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        }catch (IOException e){
            e.printStackTrace();
        }
        
    }
    
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```

**5.编写User类**

User类是一个POJO对象，其与数据库中的user表属性对应，即ORM

```java
package com.xxx.pojo;

public class User {
    private int id;
    private String name;
    private String psd;
    private  String sex;

    public User() {
    }

    public User(int id, String name, String psd, String sex) {
        this.id = id;
        this.name = name;
        this.psd = psd;
        this.sex = sex;
    }
     @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", psd='" + psd + '\'' +
                ", sex='" + sex + '\'' +
                '}';
    }
      
    // 后面getter、setter方法自动生成，这里省略不写了
    
}
```

**6.建立User类的dao接口和其对应的Mapper.xml实现**

建立User类的Dao接口，但是在MyBatis里面，一般命名为Mapper,这个接口中包含了一些对数据库操作的封装方法

```java
package com.xxx.dao;

import com.xxx.pojo.User;

import java.util.List;

public interface UserMapper {
    // 获取用户列表
    List<User> getUserList();
    
}
```

一般来说，使用JDBC是需要实现 UserMapper接口，在MyBatis中，只要用一个xml文件与该接口绑定即可，就可以xml文件在写对应方法的sql语句,下面的Mapper接口对应的xml绑定文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespece=和对应Mapper绑定-->
<mapper namespace="com.xxx.dao.UserMapper">
    <!-- id和方法绑定，type指定返回类型-->
    <select id="getUserList" resultType="com.xxx.pojo.User">
        select * from mybatis.user
    </select>
</mapper>
```

**7.注意把一组Mapper放到最开始的`mybatis-config.xml`**

这里通过资源注册

```xml
 <mappers>
        <mapper resource="com/xxx/dao/UserMapper.xml"/>
    </mappers>
```

这时基本所有配置完毕

**8.Maven引发的异常**

这时尝试运行会传来以下错误

![image-20210323150442996](/images/image-20210323150442996.png)

主要是因为maven项目中有一个目录标准，**其中src/main/java下的xml文件构建时不会被输出到`target/classes`目录下**.所以运行时找不到下面来解决它，只需在pom.xml中配置

```xml
  <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```

将java下的xml文件也能作为资源文件导出

再运行

![image-20210323151723651](/images/image-20210323151723651.png)

成功，第一个案例简单完成



