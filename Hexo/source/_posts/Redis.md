---
title: Redis
typora-root-url: ../
date: 2021-03-30 10:24:35
tags:
- Redis
- 数据库
categories:
- [数据库]
---

学习Redis，了解其常见数据结构及应用场景

<!--more-->



# 一、缓存

缓存的本质思想就是空间换时间，CPU Cache 缓存的是内存数据用于解决 CPU 处理速度和内存不匹配的问题，内存缓存的是硬盘数据用于解决硬盘访问速度过慢的问题。

回到我们对数据库的访问来说，我们为了避免用户在请求数据的时候获取速度过于缓慢，所以我们在数据库之上增加了缓存这一层来弥补

## 常见缓存方案

缓存的类型分为：**本地缓存**、**分布式缓存**和**多级缓存**

- **本地缓存**

  直接在进程的内存中进行缓存，比如我们的 JVM 堆中，我的秒杀系统1.0版就是应用了一个`HashMap`作为本地缓存。Java中本地缓存使用 `Ehcache`这样的工具来实现,主要增加了过期机制。

  由于本地缓存是内存访问，没有远程交互开销，性能最好，但是受限于单机容量，一般缓存较小且无法扩展。

- **分布式缓存**

  本地缓存对分布式架构支持不友好，并且明显受服务部署所在的机器限制，分布式缓存可以很好得解决这个问题。

  分布式缓存一般都具有良好的水平扩展能力，对较大数据量的场景也能应付自如。缺点就是需要进行远程请求，性能不如本地缓存

- **多级缓存**

  多级缓存可以用来平衡上诉两种情况，在本地缓存只保存访问频率最高的部分热点数据，其他的热点数据放在分布式缓存中。（这时候就应该想到操作系统中的快表、LRU这些东西了）

  这也是最常用的缓存方案，单考单一的缓存方案往往难以撑住很多高并发的场景

## 缓存读写策略

但使用缓存后，就得考虑如何将数据更新回数据库，下面是几种方案



# 二、Redis

## Redis简介

## Redis常见数据结构及应用场景

在学习这里时，可以先在Linux上安装Redis，安装过程可以见我博客`操作系统—>Linux—>Linux软件安装`博客

下面开始介绍Redis常用的数据结构

### String

Redis 是用 C 语言写的，但是 Redis 并没有使用 C 的字符串表示，而是自己构建了一种 **简单动态字符串**（simple dynamic string，**SDS**）这意味着使用者可以修改，它的底层实现有点类似于 Java 中的 ArrayList。此外，Redis 还对内存做极致的优化，不同长度的字符串使用不同的结构体来表示。

```c
struct __attribute__ ((__packed__)) sdshdr5 {
    unsignedchar flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsignedchar flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsignedchar flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsignedchar flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsignedchar flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```

C 语言使用了一个长度为 N+1 的字符数组来表示长度为 N 的字符串，并且字符数组最后一个元素总是 `'\0'`,但是其没有存储字符串的长度，所以其获取长度的时间复杂度是**O(N)**,在Redis的SDS中，只需要**O(1)**,此外Redis 的 SDS API 是安全的，不会造成缓冲区溢出。

下面是进入Redis中对String的简单使用，我是在阿里云的服务器上进行操作

**前置步骤**

```
systemctl start redis 启动Redis
redis-cli -h 127.0.0.1 -p 6379 进入Redis
```

**基本操作**

Redis的数据库数量是16个，进入默认是第0个，可以用` select index`切换

![image-20210331185213397](/images/image-20210331185213397.png)

上诉是对普通字符串的基本操作，主要是`set、get、exists、strlen、del`

**批量操作**

使用`mset 、mget`进行批量设置，当然`del`本身就可以批量删除

![image-20210331185541332](/images/image-20210331185541332.png)

**计数加减**

在**value字符串**为整数时，使用`incr、decr`可以操作`key`对其进行加减

值得注意，可以这个操作是 **原子性**的，多个客户端操作一个key也不会导致竞争

![image-20210331190202747](/images/image-20210331190202747.png)

对字符串，还有一个 `GETSET` ，它的功能跟它名字一样：为 key 设置一个值并返回原值，结合计数操作，这可以对于某一些需要隔一段时间就统计的 key 很方便的设置和查看，例如：系统每当由用户进入的时候你就是用 `INCR` 命令操作一个 key，当需要统计时候你就把这个 key 使用 `GETSET` 命令重新赋值为 0，这样就达到了统计的目的。

**过期时间设置**

使用`expire、setex`可以设置过期时间，`ttl`用来查询多久过期

![image-20210331190543996](/images/image-20210331190543996.png)

可以对 key 设置过期时间，到时间会被自动删除，这个功能常用来**控制缓存的失效时间**

**后置处理**

```
flushdb 清空当前数据库
flushall 清空所有数据库
dbsize 查看当前数据库大小
```



一般String常用在需要计数的场景，比如用户的访问次数、热点文章的点赞转发数量等等。

### List

C 语言并没有实现链表，所以 Redis 实现了自己的链表数据结构。Redis 的 list 的实现为一个 **双向链表**。Redis 的列表相当于 Java 语言中的 **LinkedList**，注意它是链表而不是数组。这意味着 list 的插入和删除操作非常快，时间复杂度为 O(1)，但是索引定位很慢，时间复杂度为 O(n)。

list可以用于发布与订阅或者说消息队列、慢查询

**基本操作**

- `LPUSH` 和 `RPUSH` 分别可以向 list 的左边（头部）和右边（尾部）添加一个新元素；
- `LRANGE` 命令可以从 list 中取出一定范围的元素；
- `LINDEX` 命令可以从 list 中取出指定下表的元素，相当于 Java 链表操作中的 `get(int index)` 操作；

![image-20210331192136776](/images/image-20210331192136776.png)

list可以用来实现栈或者队列

**进阶操作**

`rpoplpush`

```
rpoplpush source  destination
```

该命令原子性地返回并移除 source 列表的最后一个元素， 并把该元素放入 destination 列表的头部。使用这个命令就可以实现**安全队列**。

前面的rpop命令的特性：会移除list的队尾元素(消息)，并将这个元素(消息)返回给客户端。这意味着该元素就只存在于客户端的上下文中，redis服务器中没有这个元素了，如果客户端在处理这个返回元素的过程崩溃了，那么这个元素就永远丢失了。这种情况导致：客户端虽然成功收到了消息，但是却没有处理它。是不安全的

因为使用 RPOPLPUSH 获取消息时，RPOPLPUSH 会把消息返给客户端，同时把该消息放入一个备份消息列表，并且这个过程是原子的，可以保证消息的安全。当客户端成功的处理了消息后，就可以把此消息从备份列表中移除了。如果客户端因为崩溃的原因没有处理某个消息，那么就可以从备份列表destination中重新获取并处理这个消息。

### **hash**

ash 类似于 JDK1.8 前的 HashMap，内部实现也差不多(数组 + 链表)。不过，Redis 的 hash 做了更多优化。另外，hash 是一个 string 类型的 field 和 value 的映射表，**特别适合用于存储对象**，后续操作的时候，你可以直接仅仅修改这个对象中的某个字段的值。 比如我们可以 hash 数据结构来存储用户信息，商品信息等等。

**基本操作**

`hset,hmset,hexists,hget,hgetall,hkeys,hvals` 

下面是一个基本操作过程

![image-20210331195518979](/images/image-20210331195518979.png)

### set

set 类似于 Java 中的 `HashSet` 。Redis 中的 set 类型是一种无序集合，集合中的元素没有先后顺序。当你需要存储一个列表数据，又不希望出现重复数据时，set 是一个很好的选择，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。

可以基于 set 轻易实现交集、并集、差集的操作。比如：你可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis 可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能。这个过程也就是求交集的过程。

**基本操作**

 `sadd,spop,`添加删除

`smembers`查看所有set中元素

`sismember`检查是否包含

`scard`长度

`sinterstore`求交集，注意是三个参数，求后两者交集放在第一个集合中

`sunion` 求并集

### zset

有序列表

和 set 相比，sorted set 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列，还可以通过 score 的范围来获取元素的列表。有点像是 Java 中 HashMap 和 TreeSet 的结合体。

有序列表可以用于需要对数据根据某个权重进行排序的场景。比如在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息。

**常见操作**

`zadd,zcard,zscore,zrange,zrevrange,zrem` 等。

## 事务

`multi`

`exec`

`discard`

## 乐观锁

`watch`

详见该分类下的`Redis-锁相关`文章

# Java操纵Redis

同对MySQL进行增删查改需要进行连接一样，在Java中使用Redis也要导入驱动，使用时进行连接

## 1、利用Jedis

在pom文件中导入Jedis依赖

```xml
<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.3.0</version>
</dependency>

```

使用时同JDBC类似，建立连接即可

```java
import redis.clients.jedis.Jedis;
 
public class RedisJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        // 如果 Redis 服务设置来密码，需要下面这行，没有就不需要
        // jedis.auth("123456"); 
        System.out.println("连接成功");
        //查看服务是否运行
        System.out.println("服务正在运行: "+jedis.ping());
    }
}
```

其使用的语法基本同Redis客户端在操作一致

```java
import java.util.List;
import redis.clients.jedis.Jedis;
 
public class RedisListJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
        //存储数据到列表中
        jedis.lpush("site-list", "Runoob");
        jedis.lpush("site-list", "Google");
        jedis.lpush("site-list", "Taobao");
        // 获取存储的数据并输出
        List<String> list = jedis.lrange("site-list", 0 ,2);
        for(int i=0; i<list.size(); i++) {
            System.out.println("列表项为: "+list.get(i));
        }
    }
}
```

## 2.在Spring Boot 中集成

在Spring Boot中集成也需要导入相关依赖包

```xml
 <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
 </dependency>
```

主要