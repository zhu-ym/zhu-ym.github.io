---
title: Redis-锁相关
typora-root-url: ../
date: 2021-04-13 15:44:00
tags:
- Redis
categories:
- [数据库]
---

了解一些Redis常见的锁实现以及应用场景

<!--more-->

# 一、乐观锁

Redis的乐观锁主要基于`watch`和`事务`实现

其中`watch`可以用来检测一个key或多个key是否被修改，如果被修改执行的事务将会被打断，即大概是如下一个工作流程

```java
watch key
multi
	对key进行一系列非原子的操作
exec
```

下面是Redis集成在Spring Boot中的一个乐观锁实例，主要功能1是实现对Redis中商品名称为key的库存的扣减

```java
    public boolean updateCacheByWatch(String key) throws Exception {

        String count = (String) get(key);
        if(null == count || count == "0")
            return false;
        for(int i = 0; i < 5 ; i++){

        try {
            redisTemplate.watch(key);

            redisTemplate.multi();
            String oldvalue = (String) get(key);
            if(oldvalue != null && Integer.valueOf(oldvalue) > 0){
                System.out.println(oldvalue);
                redisTemplate.opsForValue().decrement(key);
            }

            List<Object> exec = redisTemplate.exec();
            if(!exec.isEmpty())
                return true;

        } catch (RuntimeException e) {

            //出现异常撤销
            redisTemplate.discard();

        } finally {
            redisTemplate.unwatch();
        }

        Thread.sleep(100);
    }
          return false;
    }
```

这里当修改失败时会再尝试重复五次，每次间隔100ms



# 二、分布式锁（悲观锁）

乐观锁是基于watch的重试机制，其只会在数据被其他客户端抢先修改了的情况下通知执行了这个命令的客户端，而不能阻止其他客户端对数据进行修改

而当使用了Redis实现悲观锁锁后又有一个显而易见的好处，在多机环境中Redis的范围是对所有服务器可见的，Redis的悲观锁是分布式的悲观锁，可以很好的适用于分布式系统，这时基于JVM的锁只能作用于本机，将不再适用。

![image-20210413161557941](/images/image-20210413161557941.png)

实现分布式锁主要有主要几个要求

- 排他性，只有一个进程可以获取锁
- 安全性，防止一个锁的持有者挂了锁无法释放
- 一致性，解锁和释放锁锁的是一个人

基于上述要求，自己用Redis实现分布式锁的时候主要考虑用`setnx`、`setex`结合的原子指令实现加锁操作，并且设置过期时间防止锁一直无法释放。并且每个进程加锁时设置一个唯一标识token，防止解锁时锁被其他进程释放

大概逻辑如下

```
trylock:
setnx key:lockname value:token timeout

unlock:
检查进程传入token和lockname对应value是否一致,一致则解锁
(由于上诉操作不是原子的，要么使用watch监控，要么使用lua实现原子操作，防止并发条件下释放了其他进程的锁)

```

下面是我的逻辑的具体实现

```java
 

 private static  DefaultRedisScript<Long> defaultRedisScript = new DefaultRedisScript<>();
    static{
        defaultRedisScript.setResultType(Long.class);
        defaultRedisScript.setScriptText("if redis.call('get', KEYS[1]) == KEYS[2] then return redis.call('del', KEYS[1]) else return 0 end");
        }


public boolean tryLock(String lockName,String value,long tryTime){

        Long beginTime = System.currentTimeMillis();

        while(beginTime + tryTime  >= System.currentTimeMillis()){
        if(true == redisTemplate.opsForValue().setIfAbsent(lockName,value,30,TimeUnit.SECONDS))
            return true;
        }
        return false;
    }


  public boolean unlock(String lockName,String value){
        Long result = redisTemplate.execute(defaultRedisScript, Arrays.asList(lockName, value));
        return Objects.equals(1L,result);
    }
```

以上实现在 Redis 正常运行情况下是没问题的，但如果存储锁对应key的那个节点挂了的话，就可能存在丢失锁的风险，导致出现多个客户端持有锁的情况，这样就不能实现资源的独享了，只是由于Redis的主从同步结构是异步的。



# 三、RedissonLock

Redisson中的RedissonLock是实现好了的Redis锁

相较于我自己实现的分布式锁，其主要增加了以下功能

- 可重入：增加一个数据记录锁的进入次数，实现可重入功能
- 看门狗机制：Redisson内部提供了一个监控锁的看门狗，它的作用是在Redisson实例被关闭前，不断的延长锁的有效期

