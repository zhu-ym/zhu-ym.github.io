---
title: LRU算法
typora-root-url: ../
date: 2021-05-08 10:58:20
tags:
- LRU
categories:
- [算法,经典算法]
---

自己实现一个LRU以及利用JDK实现LRU

<!--more-->

# **思路**

LRU即**最近最少使用**，是一个经典的页面调度算法，在一定容量的缓存中也被经常使用，保证缓存的数据是热点数据，其大致思路如下：

LRU会删除最近最少使用的项目。缓存应该从键映射到值(允许你插入和检索特定键对应的值)，并在初始化时指定最大容量。当缓存被填满时，它应该删除最近最少使用的项目。

首先LRU算法涉及到频繁的插入和删除，可以考虑用内部用双向链表存储这些节点（方便查找前驱）

其次为了尽快得到缓存的键值映射，而不是遍历整个链表，有可以考虑用一个Map维护key和对应的链表节点，这样也便于得到链表节点对双向链表进行操作

# 方案一：链表+Map

首先定义一个内部链表节点类，注意定义为`static`、减少持有外部类的引用消耗

其次思考对链表涉及到的操作

- 插入最新节点，那应该有addFirst API
- 删除最老节点，那应该有removeLast API
- 当一个缓存更新时，涉及到删除其旧位置并更新到最新的操作，那应该还要增加一个remove功能

最后LRU应该维护一个size变量，便于判断缓存是否溢出

设计完这些功能操作细节后，在来编写LRU的核心get、put功能，注意逻辑即可

```java
class LRUCache {

    private Map<Integer,Node> map;
    private Node first;
    private Node last;
    private int size;
    private int capacity;


    public LRUCache(int capacity) {
        map = new HashMap<>(capacity);
        first = new Node();
        last = new Node();
        first.next = last;
        last.pre = first;
        size = 0;
        this.capacity = capacity;
    }
    
    public int get(int key) {
        if(!map.containsKey(key))
            return -1;
        Node cur = map.get(key);
        remove(cur);
        addFirst(cur);
        return cur.value;

    }
    
    public void put(int key, int value) {
            Node cur = new Node(key,value);
            if(map.containsKey(key)){
                cur = map.get(key);
                remove(cur);
            }
            cur.value = value;
            // 删除缓存和老节点
            if(size == capacity){       
                Node temp = removeLast();
                map.remove(temp.key);
                }
            addFirst(cur);
            map.put(key,cur);
    }

    
    public Node remove(Node cur){
        cur.next.pre = cur.pre;
        cur.pre.next = cur.next;
        size--;
        return cur;
    }
     
    public void addFirst(Node cur){
        cur.next = first.next;
        first.next.pre = cur;
        first.next = cur;
        cur.pre = first;
        size++;
    }

    public  Node removeLast(){
           return remove(last.pre);
    }

    //  链表节点类
    private static class Node{
        int key;
        int value;
        Node pre;
        Node next;

        Node(){}
        Node(int key ,int value){
            this.key = key;
            this.value = value;
        }     
        Node(int key ,int value, Node pre ,Node next){
            this.key = key;
            this.value = value;
            this.pre = pre;
            this.next = next;
        } 
    }
}
```

# 方案二：LinkedHashMap

LinkedHashMap有这样一个方法

```java
//Returns:true if the eldest entry should be removed from the map; false if it should be retained.
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }
```

该方法可以继承，在调用map的put或putAll方法时，其调用该方法判断是否为true来决定是否删除最老的节点，一般默认是不删除，所以表现得像正常的map

于是我们可以重写该方法从而实现LRU

```java
class LRUCache extends LinkedHashMap<Integer, Integer>{
    private int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75F, true);
        this.capacity = capacity;
    }

    public int get(int key) {
        return super.getOrDefault(key, -1);
    }

    public void put(int key, int value) {
        super.put(key, value);
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity; 
    }
}
```



