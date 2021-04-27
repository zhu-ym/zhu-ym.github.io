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

# 一、IoC原理

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

注意到其继承了`ListableBeanFactory`接口，实际上该接口继承了`BeanFactory`接口，所以可以说`ApplicationContext`是`BeanFactory`接口的一个子接口，但是其有实现了其他接口，具有更多的功能，我们称之为**高级容器**

## bean的生命周期



# 二、AOP原理

# 三、Spring事务

