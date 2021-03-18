---
title: Spring
typora-root-url: ../
date: 2021-03-02 15:36:58
tags:
- Spring
categories:
- [框架]
---

初步学习Spring,主要了解IoC和AOP，并在IDEA上简单应用

<!--more-->

# Spring概述

Spring是轻量级的JavaEE框架，其主要有两个核心部分IoC和AOP

- **IoC控制反转**：控制反转是一种设计原则，减少代码之间的耦合度；基本思想是借助第三方容器实现具有依赖关系之间对象的解耦。在Spring中体现是创建对象由以前的程序员自己new 构造方法来调用，变成了交由Spring创建对象，就像控制权从本来在自己手里，交给了Spring。这里也得提到**DI依赖注入**，DI是实现控制反转的一种设计模式，是指将一个实例传入一个对象中，其可以实现解耦，而不需要对象自己关注创建实例。

- **AOP切面编程：**在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程

  一般而言，我们管切入到指定类指定方法的代码片段称为切面，而切入到哪些类、哪些方法则叫切入点。有了AOP，我们就可以把几个类共有的代码或者一个主要业务的周边功能，抽取到一个切片中，等到需要时再切入对象中去，从而改变其原有的行为。

  

# 基于IDEA的一个入门实例

## 环境配置

- 安装好IDEA的Utilmate版本，社区版不支持Spring

- 安装好的IDEA带有Maven，新建一个Maven项目

- 在Maven的pom.xml文件中增加关于Spring的依赖，自动下载管理Spring相关的jar包

  下述为依赖部分源码

  ```xml
  <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.6.RELEASE</version>
  </dependency>
  ```

- 在`src/main`目录下新建一个resources文件夹，将其设置为Resources Root （右键设置）

![image-20210303150900100](/images/image-20210303150900100.png)

- 最后在resources中新建一个xml配置文件，如果有Spring的config，则说明前面的依赖成功

  ![image-20210303151457605](/images/image-20210303151457605.png)

 

- 一个Spring项目建立完成

## 入门实例

- 在main下的包中创建一个Hello类，包含一个name属性，和一个简单方法

![image-20210303151943537](/images/image-20210303151943537.png)

```java
package org.example;

public class Hello {
    private  String name;
    public  void setName(String name){
        this.name = name;
    }
    public void sayHello(){
        System.out.println("hello " + name);
    }
}
```

- 在Spring的xml文件中的bean组件中加入Hello类的信息

  ```java
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      // 新加的bean信息，包含id，类名，属性名，属性值
      <bean id="hello" class="org.example.Hello">
          <property name="name" value="Spring"></property>
      </bean>
  </beans>
  ```

- 另建一个类，在其中新建应用上下文类，利用其getBean方法创建一个Hello类实例

  ```java
  public class App 
  {
      public static void main( String[] args )
      {
          ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
          Hello obj = (Hello) context.getBean("hello");
          obj.sayHello();
      }
  }
  ```

  运行结果如下：

  ![image-20210303152920939](/images/image-20210303152920939.png)



可见在上诉例子中App类中并没有new 一个Hello实例，而是利用Spring的应用上下文创建了一个Hello对象，实现了IoC

# IoC容器底层原理

IoC底层原理主要靠**xml解析、工厂模式、反射**实现

在上诉的案例中

- 首先在xml中加入了对于类的配置信息
- 然后，Spring会解析xml得到对应的类名和属性
- 最后，利用反射和得到的类名等信息，新建类

IoC使得bean之间的耦合度降低，IoC容器实际就是一个对象工厂

Spring实现IoC容器主要靠两大核心接口，分别是`BeanFactory`和`ApplicationContext`

- `BeanFactory`，直译Bean工厂，我们一般称`BeanFactory`为IoC容器只提供基本DI服务，而称`ApplicationContext`为应用上下文，派生于前者，提供更多服务,一般开放使用

- `ApplicationContext`是`BeanFactory`的子接口，最主要的方法就是`getBean(String beanName)`。

两者有一个显著区别，前者在使用时才创建对象，后者一加载配置文件时就创建对象

# Bean的生命周期及管理

Spring通过应用上下文将所有的将bean组件装配在一起，就可形成一个完整的应用

## Beam的生命周期

- **bean的生命周期**

  ![image-20210303162631404](/images/image-20210303162631404.png)

  首先明确通常意义bean的生命周期是指默认的单例模式，如果是原型模式，容器在初始化、配置、装饰或者是装配完一个prototype实例后，将它交给客户端，随后就不再管理该实例

  在上诉图中，bean主要有四个生命周期

  - 实例化：调用构造函数进行实例化，**Spring默认调用无参构造函数**
  - 属性填充：依赖注入bean的值和引用
  - 初始化：如果bean实现了`InitializingBean`接口，spring将调用它的`afterPropertiesSet`接口方法，如果Bean在Spring配置文件中配置了`init-method`属性会自动调用其配置的初始化方法
  - 销毁:若bean实现了DisposableBean接口，spring将调用它的distroy()接口方法。同样的，如果bean使用了destroy-method属性声明了销毁方法，则该方法被调用；

  上诉只是主要的生命周期，而实际上还有不少的扩展点比如诸多Aware类型的接口的作用就是让我们能够拿到Spring容器中的一些资源，其在初始化之前调用,并且只能作用于单一的bean，是bean级生命周期

  实例化时期间的`InstantiationAwareBeanPostProcessor`和初始化之前的`BeanPostProcessor`接口的这些后处理器方法作用于每一个bean，是容器级生命周期

- **bean的作用域：**

  ![image-20200924104801025](/images/image-20200924104801025.png)

  - request、session和global session三种作用域仅在基于web的应用中使用，只能用在基于web的Spring ApplicationContext环境。
  - singleton是单列模式，是bean作用域的默认形式，创建起容器时就自动创建了一个bean对象，之后每次返回都是这个对象
  - prototype表示一个bean对象可以创建多个对象实例，仅在需要获取对象的时候才创建实例
  - singleton和prototype区别：能对一个prototype bean的整个生命周期负责：容器在初始化、配置、装饰或者是装配完一个prototype实例后，将它交给客户端，随后就不再管理该实例

## xml管理bean

- **实例化**

  - 无参构造方法实例化

  ```xml
   <bean id="hello" class="org.example.Hello">
   </bean>
  ```

  - 有参构造方法实例化

  在之前的hello类中添加一有参构造方法

  ```java
  public Hello(String name){
          this.name = name;
      }
  ```

  再在xml中配置

  ```xml
  <bean id="hello" class="org.example.Hello">
          <constructor-arg name="name" value="constructor"/>
  </bean>
  ```

  结果如下：

  ![image-20210303170847683](/images/image-20210303170847683.png)

- **依赖注入**

  - 注入属性(setter方法注入，即该类有对应的setter方法)

    如之前入门实例

    ```xml
     <bean id="hello" class="org.example.Hello">
            <property name="name" value="Spring"></property>
     </bean>
    ```

  - 注入对象

    我们新建一个`ControlHello`类，持有一个Hello对象

    ```java
    package org.example;
    
    public class ControlHello {
        private String name;
        private  Hello hello;
        public void myControl(){
            System.out.println(name + " control " + "say hello!");
            hello.sayHello();
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public void setHello(Hello hello) {
            this.hello = hello;
        }
    }
    ```

    利用ref注入对象

    ```xml
    <bean id="hello" class="org.example.Hello">
            <constructor-arg name="name" value="constructor"/>
    </bean>
    <bean id="c" class="org.example.ControlHello">
            <property name="name" value="helloControl" ></property>
            <property name="hello" ref="hello"></property>
    </bean>
    ```

  - 集合属性注入

    新建一个Student类如下,包含对应的属性和setter方法

    ```java
    package org.example;
    
    import java.util.List;
    import java.util.Map;
    
    public class Student {
        private String name;
        private Map<String,Integer> scores;
        private List<Course> courses;
    
        public Student(String name) {
            this.name = name;
        }
    
        public void setScores(Map<String, Integer> scores) {
            this.scores = scores;
        }
    
        public void setCourses(List<Course> courses) {
            this.courses = courses;
        }
    
        @Override
        public String toString() {
            return "Student{" +
                    "name='" + name + '\'' +
                    ", scores=" + scores +
                    ", courses=" + courses +
                    '}';
        }
    }
    
    ```

    创建一个Course类

    ```java
    package org.example;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    public class Course {
        private String name;
        private String tea;
    
        public void setName(String name) {
            this.name = name;
        }
    
        public void setTea(String tea) {
            this.tea = tea;
        }
    
        @Override
        public String toString() {
            return "Course{" +
                    "name='" + name + '\'' +
                    '}';
        }
    
        public static  void main(String[] args){
            ApplicationContext context = new ClassPathXmlApplicationContext("CollectionBean.xml");
            Student student = (Student) context.getBean("stu");
            System.out.println(student.toString());
    
        }
    }
    ```

    这时我们尝试利用xml来为集合注入属性

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
        <bean id="stu" class="org.example.Student">
            <constructor-arg name="name" value="Andy"></constructor-arg>
            <property name="courses">
                <list>
                    <ref bean="course1"></ref>
                    <ref bean="course2"></ref>
                    <bean id="course3" class="org.example.Course">
                        <property name="name" value="Hadoop"></property>
                    </bean>
    
                </list>
            </property>
            <property name="scores">
                <map>
                    <entry key="计算机网络" value="91"></entry>
                    <entry key="深度学习" value="94"></entry>
                    <entry key="Hadoop" value="88"></entry>
                </map>
            </property>
    
        </bean>
        <bean id="course1" class="org.example.Course">
            <property name="name" value="计算机网络"></property>
        </bean>
        <bean id="course2" class="org.example.Course">
            <property name="name" value="程度学习"></property>
        </bean>
    
    </beans>
    ```

    当需要多个属性时，声明其对应的type就可以添加多组值

    注意一下ref对应的是对象

- **bean的作用域**

  前面已知bean默认是单例对象，可以通过Scope属性设置为原型模式

  ```xml
  <bean id="c" class="org.example.ControlHello" scope="prototype">
          <property name="name" value="helloControl" >
          </property>
          <property name="hello" ref="hello"></property>
  </bean>
  ```

  两者处理除上述生命周期管理有区别外，创建的时机也有区别，几乎就是`BeanFactory`和`ApplicationContext`的区别

## 注解管理bean

使用注解可以简化xml文件配置，

Spring管理bean中的注解

```java
@Configuration:设置配置类
@ComponScan:设置扫描路径
@Component:表明此类是bean
@AutoWired: 根据属性类型注入，指注入对象
@Qualifier:根据属性名称进行注入，指注入对象
@Resource:可以根据类型或名称注入,javax包中，指注入对象
@Value：注入普通类型属性
```

步骤

- 通过设置一个配置类

  @Configuration:设置配置类
  @ComponScan:设置扫描范围，来代替xml文件的作用

- 通过@Component注解一个类，表明此类是bean，又通过

  @AutoWired: 根据属性类型注入，指注入对象
  @Qualifier:根据属性名称进行注入，指注入对象，如果属性类型有多种，就再用这个
  @Resource:可以根据类型或名称注入,javax包中，指注入对象，推荐使用上面两个组合
  @Value：注入普通类型的属性

  等注解来注入属性

- 在建立运用上下文时，标明的是配置类的类名，就可以利用其生成bean对象了

下面是一个没有xml文件的Spring注解综合实例

**配置类**

```java
package com.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration // 设置配置类，代替xml文件
@ComponentScan(basePackages = {"com.springioc"}) // 确定扫描空间
public class SpringConfig {
}
```

**bean类**

Course

```java
package com.springioc;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class Course {
    @Value( value =  "计网")
    private String name;

    @Override
    public String toString() {
        return "Course{" +
                "name='" + name + '\'' +
                '}';
    }
}

```

Student

```java
package com.springioc;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
public class Student {
    @Value(value = "Andy")
    private String name;
    @Autowired
    private Course course;

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", course=" + course +
                '}';
    }
}
```

**测试**

```java
package com.config;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import static org.junit.Assert.*;

public class SpringConfigTest {
    @Test
    public void test(){
        ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
        System.out.println(context.getBean("student").toString());
    }

}
```

结果

![image-20210303210646137](/images/image-20210303210646137.png)

# AOP

即面向切面编程，可以对业务逻辑的各个部分进行隔离从而使业务的耦合度降低，提高程序可重用性和开发效率

我们如果把把功能分为**核心业务功能**，和**周边功能**。
所谓的核心业务，比如登陆，增加数据，删除数据都叫核心业务
所谓的周边功能，比如性能统计，日志，事务管理等等

周边功能在Spring的面向切面编程AOP思想里，即被定义为**切面**

在面向切面编程AOP的思想里面，核心业务功能和切面功能分别**独立进行开发**
然后把切面功能和核心业务功能 **"编织"** 在一起，这就叫AOP

其实和代理模式比较像，就是在不修改核心业务代码的前提下，增加其功能。

## AOP底层原理

**动态代理**

- 如果有接口，则使用jDK的动态代理

即类似于在设计模式中的动态代理，使用`InvocationHandler`接口和`Proxy`类实现接口类，增强接口的功能

- 如果没有接口，使用了CGLIB动态代理

  创建当前类子类的代理对象增强功能

## 一个AOP实例（基于注解）

- **环境配置（引入依赖）**

Spring的AOP实际是利用aspectJ框架实现，除了上诉入门实例引入的依赖，需要在Maven项目中的pom文件中引入aspectj的相关依赖，下载相关资源

```xml
<dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.1.5.RELEASE</version>
</dependency>
<dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjrt</artifactId>
      <version>1.6.12</version>
</dependency>
<dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.6.12</version>
</dependency>
```

同上，可以在`src/main`目录下新建一个resources文件夹，将其设置为Resources Root （右键设置

最后生成一个该项目的Spring xml配置文件

（前面我们完全基于注解完成，用注解类代替xml，这里我们尝试利用两者共同工作）

- **基础知识和继承类**

首先了解一些AOP的一些术语

连接点：可以被增强的方法

切入点：实际增强的方法

通知：实际增强的逻辑部分：有前置、后置、环绕、异常、最终通知

切面：把通知应用到切入点的过程

了解这些术语后我们首先创建一个用户类，包含核心业务，之后在创建一个增强类，增加日志的辅助功能

用户类

```java
package com.springaop;
public class User {
    public void core(){
        System.out.println("核心业务执行中");
    }
}
```

增强类

```java
package com.springaop;
import java.text.SimpleDateFormat;
import java.util.Date;

public class UserProxy {
    public void before(){
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd  HH:mm:ss");
        Date date = new Date(System.currentTimeMillis());
        System.out.println("业务执行时间:"+formatter.format(date));
    }
}
```

- **xml文件配置**

  之前我们是用注解开启来组件扫描的区域，这里我们用在Spring xml文件进行配置，需要先导入context名称空间，在context中确定要扫描的区域

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                             http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd   
                             http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
  
      <context:component-scan base-package="com.springaop"></context:component-scan>
      <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
  </beans>
  ```

  最后利用`<context:component-scan base-package="com.springaop"></context:component-scan>`,确定扫描的包

  此外，在这里我们还加入了aop的命名空间，目的在于利用`<aop:aspectj-autoproxy></aop:aspectj-autoproxy>`可以自动生成功能增强的代理类

- **增加对应注解**

首先这两个类都是bean，都需要增加`@Component`注解

当增强方法是，我们要知道，哪个类是增强类，需要添加`@Aspect`注解

前面说到通知有五种：有前置、后置、环绕、异常、返回通知，对应以下注解

![image-20210306164521962](/images/image-20210306164521962.png)

我们将对应的通知弄到增强类的对应方法上，这里是一个日志，我设为`@Before`

最后一步，确定该增强的方法给谁增强，即找到正确的切入点

这里有一个切入点表达式

`execution([返回类型][类全路径][方法名称]([参数列表]))`

Spring 通过这个正则表达式判断具体要拦截的是哪一个类的哪一个方法

将这个切点表达式放在`@Before`注解的值里即可

最终注解代码如下

```java
@Component
public class User {
    public void core(){
        System.out.println("核心业务执行中");
    }
}

// 两个顶级类是分开的，这里只是为了便于观察

@Component
@Aspect
public class UserProxy {
    @Before( value = "execution(* com.springaop.User.core())")
    // 切点表达式是*表示任意返回值，之后是包路径里面User类的被增强的方法
    public void before(){
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd  HH:mm:ss");
        Date date = new Date(System.currentTimeMillis());
        System.out.println("业务执行时间:"+formatter.format(date));
    }
}
```

- 结果

  我们可以在一个Test中查看最后的结果

  ```java
  @Test
      public  void test(){
          ApplicationContext context = new ClassPathXmlApplicationContext("aopBean.xml");
          User user = (User) context.getBean("user");
          user.core();
      }
  ```

  ![image-20210306165608843](/images/image-20210306165608843.png)

可以发现，我们只是创建了一个User bean，但是Spring直到帮我们创建了代理类，并成功将其日志方法增加到了核心业务执行前，实现了核心业务和周边事物的解耦

