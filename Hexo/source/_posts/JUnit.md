---
title: JUnit
typora-root-url: ../
date: 2021-03-01 15:41:48
tags:
categories:
---

简单学习如何在IDEA上用JUnit进行单元测试

<!--more-->

# 单元测试概念

单元测试是对软件或程序的基本（最小）组成单元的测试，对方法或者对象进行测试。这种以测试为驱动的开发模式最大的好处就是确保一个程序模块的行为符合我们设计的测试用例。在将来修改的时候，可以极大程度地保证该模块行为仍然是正确的

在编写单元测试的时候，我们要遵循一定的规范：

一是单元测试代码本身必须非常简单，能一下看明白，决不能再为测试代码编写测试；

二是每个单元测试应当互相独立，不依赖运行的顺序；

三是测试时不但要覆盖常用测试用例，还要特别注意测试边界条件，例如输入为`0`，`null`，空字符串`""`等情况。

在Java中，一般使用Junit进行单元测试，这样比我们用main方法手动测试有以下优点

- 新增加的测试，对原来的测试没有影响
- 如果测试失败了，会立即得到通知

# 配置IDEA上的Junit环境

- 安装插件
  - 打开`File`菜单的下拉菜单`settings[设置]`
  - 点击左侧`Plugins[插件]`菜单
  - 在输入框中输入`JUnitGenerator 2.0`进行`Install`
  - 重启`IDEA`

- 配置插件
  - 继续在`settings[设置]`中点击`other setting`
  - 点击`JUnit Generator`，在`Properties`中修改`Default Template`为`JUnit4`
  - 再在`JUnit4`页签将`package test.$entry.packageName;`修改成`package $entry.packageName;`即去掉`test`
  - 配置完成

# 测试用例

## 简单入门

创建一个项目，下面是一个项目结构

![image-20210301165254576](/images/image-20210301165254576.png)

在项目文件结构中，一般会将测试的代码放在`test/java`文件中,可以点击该目录，右键，在`open Medule settings` 中确保`Test Source Folders`的目录是正确的

![image-20210301165549571](/images/image-20210301165549571.png)

之后可以新建应该被测试类

```java
public class Sum {
    public static int sum1(int a, int b){
        return  a + b;
    }
    public static int sum2(){
        return  4;
    }
    public static  void main(String[] args){

    }
}
```

直接在测试类中`ctrl + shift + T`,就可以创造对应的测试类

```java
import org.junit.Assert;
import org.junit.Test;

import static org.junit.Assert.*;

public class SumTest {

    @Test
    public void sum1() {
        int result = Sum.sum1(1,2);
        Assert.assertEquals(3,result);
    }
    @Test
    public  void sum2(){
        Assert.assertEquals(3,Sum.sum2());
    }
    
}
```

对应需要测试的方法加上Test注解，两个方法结果如下

![image-20210301180553442](/images/image-20210301180553442.png)

## 进阶用法

一般用Assert进行测试，除了判断是否相等，还有以下测试方法

- **Assert方法**

  ```
  assertTure();assertFalse();//期待结果为true或false
  assertNotNull();//期待结果非null
  assertArrayEquals();//期待结果为数组并与期望数组每个元素的值均相等
  assertEquals(expected,actual);//期待相等
  assertEquals(expected,actual,delta);//当使用浮点数的时候，需要加上误差
  ```

- **其他注解**

除了@Test注解，Junit还可以在测试中添加`@Before ,@After `测试框架注解,其注解的方法会在所有`@Test`方法前后自动运行，做准备工作和结束清理工作

下面是一个实例

被测试类

```java
public class StringFactory {
    private int index ;
    StringFactory(){
        index = 0;
    }

    public String getter(){
        return "" + index++;
    }

    public int getIndex() {
        return index;
    }
}
```

测试类

```java
import org.junit.After;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;

import static org.junit.Assert.*;

public class StringFactoryTest<Public> {

    @Before
    public void setUp() throws Exception {
        System.out.println("测试前的准备工作，比如链接数据库等等");
    }
    @After
    public void tearDown() throws Exception {
        System.out.println("测试结束后的工作，比如关闭链接等等");
    }
    @Test
    public void getter() {
        StringFactory factory = new StringFactory();
        Assert.assertNotNull(factory.getter());
    }

    @Test
    public void getIndex() {
        Assert.assertEquals(0,new StringFactory().getIndex());
    }


}
```

![image-20210301184524338](/images/image-20210301184524338.png)

## 

