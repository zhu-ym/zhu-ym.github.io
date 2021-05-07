---
title: Spring详解
typora-root-url: ../
date: 2021-04-27 10:51:31
tags:
- Spring
categories:
- [框架,Spring]
---



主要了解Spring IoC和AOP的底层原理，以及事务的一些知识

<!--more-->

# 一、IoC

## 容器介绍

IoC即控制反转，控制反转就是把创建和管理 bean 的过程转移给了第三方。而这个第三方，就是 **Spring IoC 容器**。

IoC容器负责创建、配置和管理 bean，它管理着 bean 的生命，控制着 bean 的依赖注入。

在Spring类库的 `org.springframework.beans` 和`org.springframework.context` 的两个包下包含了IoC容器的基础。

首先，`org.springframework.beans`下有一个`BeanFactory`接口，其类似于一个HashMap：

- Key - bean name
- Value - bean object

我们可以通过`getBean`方法获取bean，但它一般只有 get, put 两个功能，所以称之为“低级容器”。

此外，`org.springframework.context` 包下有另外一个接口`ApplicationContext`

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    String getId();

    String getApplicationName();

    String getDisplayName();

    long getStartupDate();

    ApplicationContext getParent();

    AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;
}
```

注意到其继承了`ListableBeanFactory`接口，实际上该接口继承了`BeanFactory`接口，所以可以说`ApplicationContext`是`BeanFactory`接口的一个子接口，但是其有实现了其他接口，具有更多的功能，我们称之为**高级容器**，对此，我们最有可能遇到下列两种应用上下文：

`AnnotationConfigApplicationContext`:基于Java配置类加载Spring应用上下文

`ClassPathXmlApplicationContext`：从类路径下的XML配置文件中加载上下文定义

## Bean的生命周期

正常Java中的对象的生命周期由GC垃圾回收控制

Spring bean的生命周期通常意义是指默认的单例模式，如果是原型模式，容器在初始化、配置、装饰或者是装配完一个prototype实例后，将它交给客户端，随后就不再管理该实例，具体如下图所示：

![image-20210303162631404](/_posts/Spring%E8%AF%A6%E8%A7%A3.assets/image-20210303162631404.png)

我们把握住几个最关键的步骤

- **实例化**：调用构造函数进行实例化，**Spring默认调用无参构造函数**
- **属性填充：**依赖注入bean的值和引用
- **初始化：**如果bean实现了**`InitializingBean`**接口，spring将调用它的`afterPropertiesSet`接口方法，如果Bean在Spring配置文件中配置了**`init-method`属性**会自动调用其配置的初始化方法

- **销毁：**在容器关闭时销毁，若bean实现了**DisposableBean**接口，spring将调用它的distroy()接口方法。同样的，如果bean使用了**destroy-method属性**声明了销毁方法，则该方法被调用；

再了解其他过程，如果该bean实现了**`XXXAware`接口**，就会在属性填充到初始化之间调用该接口对应的`setXXX`方法，让该bean**持有ID、BeanFactory、ApplicationContext等一些Spring容器的资源**，注意这些方法是实现了接口的bean才会调用

最后注意一个特殊的**`BeanPostProcessor`接口**，其有两个方法，`postProcessBeforeInitialization()方法`和`postProcessAfterInitialization()方法`，作用于**初始化**阶段的前后



**bean的作用域：**

![image-20200924104801025](/_posts/Spring%E8%AF%A6%E8%A7%A3.assets/image-20200924104801025.png)

- request、session和global session三种作用域仅在基于web的应用中使用，只能用在基于web的Spring ApplicationContext环境。
- singleton是单列模式，是bean作用域的默认形式，创建起容器时就自动创建了一个bean对象，之后每次返回都是这个对象
- prototype表示一个bean对象可以创建多个对象实例，仅在需要获取对象的时候才创建实例
- singleton和prototype区别：能对一个prototype bean的整个生命周期负责：容器在初始化、配置、装饰或者是装配完一个prototype实例后，将它交给客户端，随后就不再管理该实例

## Bean的诞生

在单例模式下，一个Bean的产生和容器的创建息息相关

由于Spring 推荐使用注解，我的容器入口是 `AnnotationConfigApplicationContext` ，如果是使用 xml 分析，那么入口即为 `ClassPathXmlApplicationContext` ，它们俩的共同特征便是都继承了 `AbstractApplicationContext` 类

首先查看 `AnnotationConfigApplicationContext`常用的构造器方法，主要是调用第一个

```java
  // 常用构造方法
  public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
        // 容器初始化
        this();
        // 注册配置类信息
        this.register(componentClasses);
      	// 容器刷新
        this.refresh();
    }
	// 无参构造器
    public AnnotationConfigApplicationContext() {
        // 首先调用父类默认构造器
        StartupStep createAnnotatedBeanDefReader = this.getApplicationStartup().start("spring.context.annotated-bean-reader.create");
        // 得到AnnotatedBeanDefinitionReader
        this.reader = new AnnotatedBeanDefinitionReader(this);
        createAnnotatedBeanDefReader.end();
        // 得到ClassPathBeanDefinitionScanner
        this.scanner = new ClassPathBeanDefinitionScanner(this);
    }
```

我们的bean产生经历了什么，可以大概从第一个构造器的三个步骤来分析

**1.this（）**

调用了自己的无参构造器，而在无参构造器中，又有几步：

- 调用了父类构造器，生成了一个**beanFactory 工厂**（`DefaultListableBeanFactory`）；

- 生成一个**AnnotatedBeanDefinitionReader**，可以将对加了特定注解（如 `@Service`、`@Repository`）的类进行读取转化成 `BeanDefinition` 对象
- 生成一个**ClassPathBeanDefinitionScanner**，可以对用户指定的包目录进行扫描查找 bean 对象

上面有提到一个**BeanDefinition对象：**spring当中的BeanDefinition就是java当中的Class，Class可以用来描述一个类的属性和方法等等其他信息。BeanDefintion可以描述springbean当中的scope、lazy，bean的注入模型、以及属性和方法等等其他信息，而这些信息是Class无法抽象出来的

**2.this.register(componentClasses)**

注册配置类信息，利用生成的reader将Appconfig类解析成为一个beanDefintion对象，然后给解析出来的beanDefinition对象设置一些默认属性，继而put到beanDefintionMap当中

**3.refresh（）**

在refresh中主要有两步：

- 扫描出要放入容器的 bean，将其包装成 `BeanDefinition` 对象
- 然后通过反射创建 bean，并完成赋值操作

但是这个只是IoC容器最简单的功能，实际上在这个过程中，我们可以用`BeanFactoryPostProcessor` 后置处理器，在扫描完 bean 之后做一些自定义的操作

也可以用前文提到的`BeanPostProcessor` 后置处理器在 bean 的初始化前后做一些操作

这些更细致的操作，应该会在之后再次仔细阅读Spring源码后更新，这里只是为了理清楚时间线罢了，相当于是阅读源码前的一个辅助

最后该部分以一图收尾，下图转载自[路神](https://blog.csdn.net/java_lyvee)，其对博客的文章Spring 这一块的讲解比较深入

![img](/images/20191018151632339.jpg)



总的来说，走完这一次流程后，可以了解IoC底层原理主要靠**注解解析（xml）、工厂模式、反射**实现

# 二、AOP原理

即面向切面编程，可以对业务逻辑的各个部分进行隔离从而使业务的耦合度降低，提高程序可重用性和开发效率

我们如果把把功能分为**核心业务功能**，和**周边功能**。
所谓的核心业务，比如登陆，增加数据，删除数据都叫核心业务
所谓的周边功能，比如性能统计，日志，事务管理等等

周边功能在Spring的面向切面编程AOP思想里，即被定义为**切面**

在面向切面编程AOP的思想里面，核心业务功能和切面功能分别**独立进行开发**
然后把切面功能和核心业务功能 **"编织"** 在一起，这就叫AOP

其实和代理模式比较像，就是在不修改核心业务代码的前提下，增加其功能。

而其实，Spring的默认实现就采用的是**动态代理**。

## 动态代理

类似于在设计模式中的动态代理，使用`InvocationHandler`接口和`Proxy`类实现接口类，增强接口的功能

实例：

在电影院看电影除了电影，本身还会在电影放映前后插入广告。

希望用**动态代理模式**对电影的代理，并增加广告功能

- 首先建立Movie抽象类，再建立其一个子类

  ```java
  package proxy;
  
  public interface Movie {
  	void play();
  }
  ```

  ```java
  package proxy;
  
  public class RealMovie implements Movie{
      String name;
      public RealMovie(String name) {
  		this.name = name;
  	}
  	@Override
  	public void play() {
  		System.out.println("正在播放"+name);
  	}
  
  }
  ```

- ` InvocationHandler `实现类。

  ```java
  package proxy;
  
  import java.lang.reflect.InvocationHandler;
  import java.lang.reflect.Method;
  
  public class ProxyHandler implements InvocationHandler{
  	private Object proxied;
  	public ProxyHandler(Object obj) {
  		proxied = obj;
  	}
  
  	@Override
  	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  		System.out.println("放映前插入广告");
  		Object obj = method.invoke(proxied, args);
  		System.out.println("放映后插入广告");
  		return obj;
  	}
  }
  ```

- 客户端利用Proxy类生成实例

  ```java
  package proxy;
  
  import java.lang.reflect.Proxy;
  
  public class Client {
  	public static void main(String[] args) {
  		Movie proxied = new RealMovie("白蛇缘起");
  		ProxyHandler handler = new ProxyHandler(proxied);
  		Movie proxy =(Movie) Proxy.newProxyInstance(Movie.class.getClassLoader(),new Class[] {Movie.class}, handler);
  		proxy.play();
  		System.out.println(proxy.getClass());
  	}
  
  }
  ```

  结果

  ![image-20201010111924276](/images/image-20201010111924276.png)

观察上诉案例，发现一个要求，要求增强的目标类是一个接口

实际上**Spring默认的策略是如果需要增强目标类是接口，则使用JDK动态代理技术，否则使用Cglib来生成代理**

## Cglib

CGLIB是一个强大的、高性能的代码生成库。其被广泛应用于AOP框架（Spring、dynaop）中，用以提供方法拦截操作。CGLIB相比于JDK动态代理更加强大，JDK动态代理虽然简单易用，但是其有一个致命缺陷是，只能对接口进行代理。如果要代理的类为一个普通类、没有接口，那么Java动态代理就没法使用了。

所以当Spring要代理的对象不是接口时，就采用Cglib进行代理，创建当前类子类的代理对象增强功能但是得注意，该类需要可以继承

CGLIB底层使用了ASM来操作字节码生成新的类，是一种字节码生成技术。

# 三、Spring事务

## 事务原理

Spring的事务实际上和AOP和数据库有关，又可以说Spring事务是Spring AOP的最佳实践之一

一般来说，我们用JDBC使用事务时，有以下操作：

```
1、获取连接 Connection con = DriverManager.getConnection()

2、开启事务con.setAutoCommit(true/false);

3、执行CRUD

4、提交事务/回滚事务 con.commit() / con.rollback();

5、关闭连接 conn.close();

```

采用Spring 事务后，只需要关注第3步的实现，其他的步骤都是Spring 完成（准确的是事务切入的是2、4步）。

Spring事务的本质  其实就是 **AOP和数据库对事务的支持**，**Spring 将数据库的事务操作提取为切面，通过AOP的方式增强事务方法。**

当一个方法被使用`@Transactional`注解后就会生成相应的代理类，增强该方法

最后注意一下Spring事务的回滚机制：**当所拦截的方法有指定异常（不受检查的异常）抛出，事务才会自动进行回滚**，当具体编码时，如果事务结果未满足具体业务需求，可以手动抛出异常回滚

## Spring事务隔离级别

Spring的事务隔离级别和数据库的隔离级别是一回事，就是那四个

**READ_UNCOMMITTED （读未提交）**
这是事务最低的隔离级别，它允许另外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读。

**READ_COMMITTED （读已提交）**
保证一个事务修改的数据提交后才能被另外一个事务读取，另外一个事务不能读取该事务未提交的数据。这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读。

**REPEATABLE_READ （可重复读）**
这种事务隔离级别可以防止脏读、不可重复读，但是可能出现幻读。它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了不可重复读。

**SERIALIZABLE（串行化）**
这是花费最高代价但是最可靠的事务隔离级别，事务被处理为顺序执行。除了防止脏读、不可重复读外，还避免了幻读。

spring只是在此基础上抽象出一种隔离级别为**default**，表示以数据库默认配置的为主。例如，mysql默认的事务隔离级别为repeatable-read。而Oracle 默认隔离级别为读已提交。

并且如果数据库设置的隔离级别和Spring设置的隔离级别有冲突，将以Spring的为主，但如果spring设置的隔离级别数据库不支持，效果取决于数据库。

## 事务传播行为

事务的传播行为主要是指多个使用了`@Transactional`注解的方法互相调用时，Spring对事务的处理。

Spring 对事务传播有七个行为类型

**1.required**

`如果当前有事务，则当前事务加入到当前层事务，一块提交，一块回滚。如果外层没有事务，新建一个事务执行`

Spring的默认事务传播行为，也是最常见的事务传播行为

下面用字母代表方法、大写代表是事务方法、小写不是,B、C方法是在DAO层，事务传播行为为`required`

```
class XXXService{

	// 1
	A(){
	B();
	C();
	...
	}
	// 2
	a(){
	B();
	C();
	}
}
```

- 对1，BC方法加入到A的事务中，但凡BC中或者A中出现异常，所有事务将回滚
- 对2，B，C将各自新建一个事务，两者独立不影响

**2.nested**

`如果当前存在事务，支持嵌套事务，否则同required一致`

嵌套事务一个非常重要的概念就是**内层事务依赖于外层事务**。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚。

即在上诉1案例中，BC异常不影响A，A异常会影响BC

**3.requires_new**
`新建事务，如果当前存在事务，把当前事务挂起。`

ABC相互独立

**即是外围事务和内部事务之间均独立**

**4.supports**

`支持当前事务，如果当前没有事务，就以非事务方式执行（没有事务）`

根据调用它的方法来决定是否有事务

**5.mandatory**

`使用当前的事务，如果当前没有事务，就抛出异常`

和4基本一致，但是没有事务即2中会抛出异常

**6.never**

`以非事务方式运行，如果当前存在事务，抛出异常`

1中抛出异常

2中没有事务

**7.not_supported**

`以非事务方式执行操作，如果当前存在事务，就把当前事务挂起`

1中BC不是事务

2中没有事务

## 事务同步器

Spring把JDBC 的 `Connection`或者`Hibernate`的`Session`等访问数据库的链接（会话）都统一称为资源，显然我们知道`Connection`这种是线程不安全的，同一时刻是不能被多个线程共享的。

我们需要同一时刻我们每个线程持有的Connection应该是独立的，且都是互不干扰和互不相同的

Spring里面有一个事务同步管理类`org.springframework.transaction.support.TransactionSynchronizationManager`来解决这个问题。它的做法是**内部使用了很多的ThreadLocal为不同的事务线程提供了独立的资源副本，并同时维护这些事务的配置属性和运行状态信息** （比如强大的事务嵌套、传播属性和这个强相关）。
