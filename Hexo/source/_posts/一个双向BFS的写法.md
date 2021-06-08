---
title: 一个双向BFS的写法
typora-root-url: ../
date: 2021-05-21 10:51:04
tags:
- 双向BFS
categories:
- [算法，BFS]
---

BFS是穷举法的一种，但是天生适合用来求最少次数等最值问题

而双向BFS相等于是对BFS的一种优化，在情况越大时，效果越好

<!--more-->

# 题目

**773. 滑动谜题**

[力扣链接](https://leetcode-cn.com/problems/sliding-puzzle/)

在一个 2 x 3 的板上（`board`）有 5 块砖瓦，用数字 `1~5` 来表示, 以及一块空缺用 `0` 来表示.

一次移动定义为选择 `0` 与一个相邻的数字（上下左右）进行交换.

最终当板 `board` 的结果是 `[[1,2,3],[4,5,0]]` 谜板被解开。

给出一个谜板的初始状态，返回最少可以通过多少次移动解开谜板，如果不能解开谜板，则返回 -1 。

**示例：**

```
输入：board = [[1,2,3],[4,0,5]]
输出：1
解释：交换 0 和 5 ，1 步完成
输入：board = [[1,2,3],[5,4,0]]
输出：-1
解释：没有办法完成谜板
输入：board = [[4,1,2],[5,0,3]]
输出：5
解释：
最少完成谜板的最少移动次数是 5 ，
一种移动路径:
尚未移动: [[4,1,2],[5,0,3]]
移动 1 次: [[4,1,2],[0,5,3]]
移动 2 次: [[0,1,2],[4,5,3]]
移动 3 次: [[1,0,2],[4,5,3]]
移动 4 次: [[1,2,0],[4,5,3]]
移动 5 次: [[1,2,3],[4,5,0]]
输入：board = [[3,2,4],[1,5,0]]
输出：14
```

**提示：**

- `board` 是一个如上所述的 2 x 3 的数组.
- `board[i][j]` 是一个 `[0, 1, 2, 3, 4, 5]` 的排列.

# 思路概述

该题求最小次数，显然采用BFS，大致根据起始状态，按照题目方式变换到相邻状态，再看最后是否能够变换到

目标态

值得注意一点，该题可能无解，这必须要求我们不能回到已经到过的状态，**不然循环一定退不出来**，关于判断状态是否到达，肯定是用**集合**时间高效

但决定采用集合后，本题的难点也就在此：什么样的**状态存储方便放入集合**

有两个思路：

- 将状态转化成**字符串或数字等Java的原生值比较类**

- **自定义一个类**，重写`hashCode`和`equals`方法

# 思路一：利用Java原生类

对于思路一，转成字符串简单，可能消耗比数字大；转化为一个整数，由于存在0在首位的问题，可以尝试将0替换成6-9的一个数字，就更加不用思考特殊情况

这两种方法推荐使用第二种，可以在BFS队列中就放置一个int[] 数组，将board存放为一维，还可以多加一位作为标记0的位置

但是这里我采用了字符串的方式，主要是我想练习一下双向BFS的书写，采用的是Set集合作为队列

near数组是二维化为一维后的标记

```java
class Solution {
    // 记录相邻可变换位置
    int [][] near= {{1,3},{0,4,2},{1,5},{0,4},{3,1,5},{4,2}};

    public int slidingPuzzle(int[][] board) {
        Set<String> q1 = new HashSet<>();
        Set<String> q2 = new HashSet<>();
        Set<String> set1 = new HashSet<>();
    


        StringBuilder sb = new StringBuilder();
        for(int[] arr: board ){
            for(int i: arr){
                sb.append(i);
            }
        }
        q1.add(sb.toString());
        q2.add("123450");
        int step = 0;
        
        while(!q1.isEmpty() && !q2.isEmpty()){
            
            // 每次选取size小的队列做q1，减少遍历广度
            if(q2.size() < q1.size()){
                Set<String> temp = q2;
                q2 = q1;
                q1 = temp;
            }
            
            Set<String> newQueue = new HashSet<>();
  
            for(String str: q1){
                if(q2.contains(str))
                    return  step;
       
                set1.add(str);
                char[] ch = str.toCharArray();
                
                // 字符串的find0消耗了一定时间
                int k = find0(ch);
                
                for(int j: near[k]){
                    swap(k,j,ch);
                    String s1 = new String(ch);
   					// 去重
                    if(!set1.contains(s1))
                        newQueue.add(s1);
                    swap(k,j,ch);
                }
            }
 
            step++;
            q1 = newQueue;
        }
        
      return -1;

    }
    
    public int find0(char[] c){
        for(int i = 0; i < c.length; i++)
            if(c[i] == '0')
                return  i;
            
        return -1;
    }
    public void swap(int i, int j, char[] ch){
        char temp = ch[i];
        ch[i] = ch[j];
        ch[j] = temp;
        
    }
    
}
```

