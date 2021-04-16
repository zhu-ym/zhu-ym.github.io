---
title: Java基本类型和包装类型的坑
typora-root-url: ../
date: 2021-04-14 16:08:46
tags:
 - Java语法
categories:
 - [Java语法,基础]
---

了解一下Java基本类型和包装类和一些常见的坑

<!--more-->

# 一、基本类型和包装类

## 1.**基础**

Java基本类型有`boolean、byte、short、char、int、long、float、double`八大类，对应也有包装类型`Boolean,Byte,Short,Character,Integer,Long,Float,Double`

Java一切皆对象，但是八大基本类型不是对象，包装类的产生解决基本数据类型没有属性，方法，不能对象化交互的问题，但是也带来了性能消耗

基本类型和包装类型主要有以下区别：

- **包装类型是对象，可以为空**
- **两者在JVM上内存分配位置不一致**
- **两者通常判断值是否相等的方式不一致**，一个用`==`，另一个最好用`equals`
- **包装类可以用于泛型，而基本类型不能**

在上诉差异，最好利用包装类型的特性，仅在需要的时候使用包装类型，包装类型主要有以下应用场景

- **泛型或者泛型参数**
- **POJO类属性必须使用包装数据类型**，主要为了防止**NPE**
- **RPC 方法的返回值和参数必须使用包装数据类型**，包装类返回的null能有额外信息



## **2.自动装箱拆箱过程**

Java为了优化基本类型和包装类型转化，有自动装箱和拆箱机制，**自动装箱时编译器调用valueOf将原始类型值转换成对象，同时自动拆箱时，编译器通过调用类似intValue(),doubleValue()这类的方法将对象转换成原始类型值。**而这些调用都不用我们显式调用

下面以Integer为例看着两个方法

装箱时

```java
 public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

拆箱时

```java
private final int value;
public int intValue() {
        return value;
    }
```



## 3.缓存机制

注意到前面`Integer`的`valueOf`中，对int的值有条件判断，在一定范围内，返回的是`IntegerCache.cache[i + (-IntegerCache.low)];`,否则才会新建一个对象

再查看`IntegerCache`

```java
  private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

**IntegerCache内部实现了一个Integer的静态常量数组，在类加载的时候，执行static静态代码块进行初始化-128到127之间的Integer对象.**

**如果缓存的这些对象都是经常使用到的，使用缓存防止每次自动装箱都创建一次对象的实例。**

**除了Float和Double，其他包装类都有缓存机制**

# 二、踩坑集合

## 1.泛型和泛型参数相关

```java
// error
int[] arr =  {1,2,3,5,8,2,5,4};
Arrays.sort(arr,( a, b)->{ return  b - a;});
```

该方法接口如下

`public static <T> void sort(T[] a, Comparator<? super T> c)`

在使用泛型参数的时候使用了基本类型，错误

## 2.缓存机制相关

所有的相同类型的包装类对象之间值的比较，全部使用 equals 方法比较,不然可能会因为缓存机制带来出乎意料的结果。包装类重写了equals方法，都是值比较，甚至，Java的值类型的equals方法都是值比较

```java
  public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
```

此外最好使用Objects的比较方法，避免`a.equals(b)`中a为空

```java
 public static boolean equals(Object a, Object b) {
        return (a == b) || (a != null && a.equals(b));
    }
```

