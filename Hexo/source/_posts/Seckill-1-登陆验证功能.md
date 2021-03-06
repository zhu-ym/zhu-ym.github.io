---
title: Seckill 1.登陆验证功能
typora-root-url: ../
date: 2021-03-26 16:07:21
tags:
- Seckill
categories:
- [项目,秒杀系统]
---

秒杀系统第一步，创建搭建环境和编写第一个demo，增加登陆验证功能

这里我主要学习后端，前段的界面是在`github`的仓库找的，会在末尾添上链接

<!--more-->

# 一、环境搭建

基础环境

- IDEA 2021
- Spring Boot 2.4.4
- thymeleaf
- Mysql 8.0.23
- MyBatis  2.1.4
- jdk 1.8
- maven 3.6.3

## 1.数据库搭建

登陆功能涉及到用户类，在mysql数据库中新建一个seckil数据库，并且新建一个User表

下图是我建表包含的具体列

![image-20210329200837037](/images/image-20210329200837037.png)

## 2.新建Spring Boot 项目

- **利用Spring Initializer 搭建项目**

  新建一个`Spring Initializer`项目，导入依赖时导入`Spring web`、`Thymeleaf`、`mysql driver`、`MyBatis`等依赖

- **pom.xml文件中再导入MD5相关的jar包**

  登陆功能涉及到两次MD5加密，先导入相关依赖包

  ```xml
  <dependency>
              <groupId>commons-codec</groupId>
              <artifactId>commons-codec</artifactId>
              <version>1.6</version>
          </dependency>
  ```

- **对web应用进行基础配置**

  在resource文件下对application文件进行配置，这里我把原来的文件删除了，而是新建了`application.yml`进行配置，比较清晰

  ```yml
  spring:
    thymeleaf:
      cache: false
  
    datasource:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/seckill?useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8
      username: root
      password: root
      hikari:
        pool-name: DateHikar
        minimum-idle: 5
        idle-timeout: 1800000
        maximum-pool-size: 9
        auto-commit: true
        max-lifetime: 1800000
        connection-timeout: 30000
        connection-test-query: SELECT 1
  
  
  
  mybatis:
    mapper-locations: classpath:mapper/*Mapper.xml
    type-aliases-package: com.zhu.seckill.pojo
    configuration:
      map-underscore-to-camel-case: true
  
  logging:
    level:
      com.zhu.seckill.mapper: debug
  ```

  里面包含了对数据库、MyBatis、thymeleaf等配置

- **建立完整目录结构**

  下面是我的目录结构，后面可能会有补充，大概了解一些每个类应处的层次

  ```
  
  ├── pom.xml  -- 项目依赖
  └── src
      ├── main
      │   ├── java
      │   │   └── com.zhu.seckill
      │   │       
      │   │           ├── SeckillApplication.java  -- SpringBoot启动器
      │   │           ├── controller  -- 控制层，MVC中的处理器
      │   │           ├── dto  -- 统一封装的一些结果属性，和pojo类似
      │   │           ├── pojo  -- 普通Java类，存放数据库对应实体
      │   │           ├──  vo  -- 与视图层传输数据的实体
      │   │           ├── exception  -- 统一的异常结果
      │   │           ├── mapper  -- Mybatis-Mapper层映射接口，即DAO层
      │   │           ├── utils  -- 一些工具类的编写
      │   │           └── service  -- 业务层
      │   └── resources
      │       ├── application.yml  -- SpringBoot核心配置文件
      │       ├── mapper  -- Mybatis-Mapper层XML映射文件
      │       ├── static  -- 存放页面静态资源，可通过浏览器直接访问
      │       │   ├── css
      │       │   ├── js
      │       │   └── lib
      │       └── templates  -- 存放Thymeleaf模板引擎所需的HTML，不能在浏览器直接访问
      └── test  -- 测试文件
  ```

到此一个基本的Spring Boot 项目基本搭建完成，下面是具体业务的编写

# 二、基本原理

这里我们主要设计的是登陆验证功能，其原理主要使用了常见的MD5

MD5是一种数据摘要算法，可以用于数据的加密，文件快传，文件校验，数据压缩等方面，其用哈希函数把任意长度的报文信息转换为固定长度的哈希，其特点是: 

- 经过MD5转化后的数据，是不能被破解的，无法得到原有的明文内容
- 经过加密的数据，都是128位2进制数据组成。通常会把它书写成32位16进制数据
- 同一份数据经过md5加密之后，一定会得到同一个结果

注意前面说的破解是通过算法倒推出原文，但是存在人可以用暴露枚举的方式用一个字典挨个生成MD5加密后的密文，再用输入的密文反查询

首先，当前端明文密码传到服务器时，需要**一次MD5加密**，这里主要是**避免明文密码被抓包获取**。

如果这里就将该密文存储在数据库，这里有一个问题,攻击成本问题，如果被人有了整个数据库的密文，只要代价足够，其可以通过构建彩虹表即前文说的暴露枚举破解密文得到明文密码，因为他建立彩虹表是针对所以用户的

这里解决方案是**加盐**，可以给每一个用户的密文插入随机值的字符串，再次进行MD5加密，每个用户都有不同的盐（字符串），别人的攻击成本就会上升，其再想建立彩虹表，也只是针对一个用户的，破解代价很大

所以最后，当服务器加密码保存到数据库中是时，需要**第二次MD5+随机盐值加密**，这时是为了**增加彩虹表反查的难度**

当然第一次MD5加密也可以加盐值，但是必须是固定的，因为MD5不能破解，那就不能每次传递过来的密码不一致

# 三、代码

- **MD5加密的工具类编写**

  ```java
  package com.zhu.seckill.utils;
  
  import org.apache.commons.codec.digest.DigestUtils;
  
  public class MD5Utils {
  
      // 固定salt值
      private static final String salt = "2z5b864m";
  
      // 单纯获取MD5值
      public static String getMD5(String src){
          return DigestUtils.md5Hex(src);
      }
  
      // 第一次加密的MD5值，实际是在前端做的
      public static String inputPassToFormPass(String inputPass){
          String base ="" + salt.charAt(3)+salt.charAt(6)+inputPass+salt.charAt(0)+salt.charAt(5);
          return DigestUtils.md5Hex(base);
      }
  
      // 第二次加密的MD5值,这里的salt应该是每个用户随机生成的一个，这也是真正验证和写入数据库时调用的主要方法
      public static String formPassToDBPass(String formPass,String salt){
          String base = ""+ salt.charAt(3)+salt.charAt(6)+formPass+salt.charAt(0)+salt.charAt(5);
          return DigestUtils.md5Hex(base);
      }
  
      public static  String inputPassToDBPass(String inputPass,String databasesalt){
          String formpass = inputPassToFormPass(inputPass);
          return formPassToDBPass(formpass,databasesalt);
      }
  }
  ```

- **验证逻辑**

  ```java
      public RespBean doLogin(LoginVO loginVO) {
          String mobile = loginVO.getMobile();
          String password = loginVO.getPassword();
  
          if(mobile == null || password == null)
              return RespBean.error(RespBeanEnum.LOGIN_ERROR);
          if(!VoliationUtils.isMoblie(mobile))
              return RespBean.error(RespBeanEnum.MOBILE_ERROR);
          // 查询数据库存储的密文和盐值
          User user = userMapper.getUserById(Long.valueOf(mobile));
          if(user == null)
              return RespBean.error(RespBeanEnum.SIGNIN_ERROR);
          // 将第一次密文转化成第二次密文后比较
          if(!Objects.equals(MD5Utils.formPassToDBPass(password,user.getSalt()),user.getPsd()))
              return RespBean.error(RespBeanEnum.PSSSWARD_ERROR);
  
          return RespBean.success(user);
      }
  ```

- **查询逻辑**

  验证逻辑中需要查询用户数据，下面是查询的sql语句,主要在UserMapper中实现

  ```java
  @Repository
  public interface UserMapper {
  
  
      @Select("select * from seckill.user where id = #{id}")
      User getUserById(long id);
      @Insert("insert  into user(id,nickname,psd,salt) values(#{id},#{name},#{psd},#{salt})")
      void  InsertUser(Long id ,String name,String psd,String salt);
  }
  ```

  

# 四、前端的一些东西的连接

[hfbin的github仓库](https://github.com/hfbin/Seckill)

我前端的页面和资源如要参照这个大佬的仓库，再自己对其中一些接口进行了修改

代项目完毕后，也可以看我的仓库

