---
title: LFU算法
typora-root-url: ../
date: 2021-05-09 15:01:23
tags:
- LFU
categories:
- [算法,经典算法]
---

在LRU的基础上更近一步实现LFU

<!--more-->

# 题目描述

[力扣链接](https://leetcode-cn.com/problems/lfu-cache/)

请你为 [最不经常使用（LFU）](https://baike.baidu.com/item/缓存算法)缓存算法设计并实现数据结构。

实现 `LFUCache` 类：

- `LFUCache(int capacity)` - 用数据结构的容量 `capacity` 初始化对象
- `int get(int key)` - 如果键存在于缓存中，则获取键的值，否则返回 -1。
- `void put(int key, int value)` - 如果键已存在，则变更其值；如果键不存在，请插入键值对。当缓存达到其容量时，则应该在插入新项之前，使最不经常使用的项无效。在此问题中，当存在平局（即两个或更多个键具有相同使用频率）时，应该去除 **最近最久未使用** 的键。

**注意**「项的使用次数」就是自插入该项以来对其调用 `get` 和 `put` 函数的次数之和。使用次数会在对应项被移除后置为 0 。

为了确定最不常使用的键，可以为缓存中的每个键维护一个 **使用计数器** 。使用计数最小的键是最久未使用的键。

当一个键首次插入到缓存中时，它的使用计数器被设置为 `1` (由于 put 操作)。对缓存中的键执行 `get` 或 `put` 操作，使用计数器的值将会递增。

# 思路分析

显然题意在删除元素时会删除**最低频率最近最少未使用**的键

其中**最近最少未使用**是LRU，之前我们的实现是用一个双向队列+HashMap实现，这里涉及到不同频率，我们可以考虑为不同频率维护不同的双向队列（HashMap），同时维护一个当前最小频率，可以直接找到要删除的双端队列

关键的get、put方法思路如下

```
get(int key){
	缓存中没有返回-1
	有
	首先先更新其出现频率到对于频率的队列
	然后判段当前最短频率是否因此改变
	最后返回对应值
}

put(int key,int value){
	缓存中没有{
		插入到队列
		最小频率更新为1
		判断是否删除最老值
	}
	缓存中有{
		首先先更新其出现频率到对于频率的队列
	    然后判段当前最短频率是否因此改变	
	}
}
```

根据上述思路，我们可以抽象出几个函数，提高代码复用

- 插入值函数
- 更新最小频率函数
- 更新值频率函数
- 删除最老值函数

最低一层的抽象涉及到对双向链表的插入和删除，注意插入时我们默认把最新元素插入到队列头，所以可以有以下两个函数

- 节点插入到队列头函数
- 删除元素操作

# 代码

最终Java代码如下

```java
class LFUCache {
    // key-vaue
    Map<Integer,Node> cache;
    // count-list
    Map<Integer,Node[]> lists;
    int minCount;
    int size;
    int capacity;

    public LFUCache(int capacity) {
        cache = new HashMap<>(capacity);
        lists = new HashMap<>();
        size = 0;
        minCount = 0;
        this.capacity = capacity;
    }
    
    public int get(int key) {
        
        if(!cache.containsKey(key))
            return - 1;
        Node cur = cache.get(key);
        int oldCount = cur.count;
        updateByCount(cur);
        if(oldCount == minCount)
            updateMinCount();

        return cur.value;
    }
    
    public void put(int key, int value) {
        if(capacity == 0)
            return;
        Node cur ;
        
        if(!cache.containsKey(key)){
            if(size == capacity)
                removeOldest();
            minCount = 1;
            cur = new Node(key,value,1);
            insertByCount(cur);
        }
        else{
            cur = cache.get(key);
            cur.value = value;
            int oldCount = cur.count;
            updateByCount(cur);
            if(oldCount == minCount)
                updateMinCount();

        }
    }

    private void updateMinCount(){
        Node[] arr = lists.get(minCount);
        if(arr[0].next == arr[1])
            minCount++;
        
    }

    // 插入
    private void insertByCount(Node cur){
        int newCount = cur.count;
        Node first;
        Node last;
        
        // 双向链表初始化
        if(!lists.containsKey(newCount)){
            first = new Node();
            last = new Node();
            first.next = last;
            last.pre = first;
            lists.put(newCount,new Node[]{first,last});
        }
        else{
            Node[] arr= lists.get(newCount);
            first = arr[0];
        }
        addFirst(cur,first);
        
    }

    private void updateByCount(Node cur){
        remove(cur);
        cur.count++;
        insertByCount(cur);
    }
    
    
    private Node removeOldest(){
        Node[] arr = lists.get(minCount);
        Node cur = arr[1].pre;
        remove(cur);
        return cur;
    }
 

    private static class Node{
        int key;
        int value;
        int count;
        Node pre;
        Node next;
        Node(){};
        Node(int key, int value, int count){
            this.key = key;
            this.value = value;
            this.count = count;
        }
    }

    private Node remove(Node cur){
        cur.next.pre = cur.pre;
        cur.pre.next = cur.next;
        size--;
        cache.remove(cur.key);
        return cur;
    }
     
    private void addFirst(Node cur, Node first){
        cur.next = first.next;
        first.next.pre = cur;
        first.next = cur;
        cur.pre = first;
        size++;
        cache.put(cur.key,cur);
    }

}
```

其中Node[]大小为2包含头尾节点就相当于一个双向队列，这里也可以再做一层封装，成为一个DoubleList类

其次size这个元素可以不用维护，可以通过cache的size来判断，我这里没有修改