---
title: 缓冲行填充
date: 2017-06-30 22:49:00
tags: Java, Disruptor 
categories: Disruptor
---

[缓冲行填充](http://ifeve.com/disruptor-cacheline-padding/)

### 1. 计算机缓存分为L1， L2， L3，越靠近CPU越快，然而越小；
### 2. 缓存由缓存行组成，一个缓存行一般是64字节；  
### 3. CPU实现中以一个缓存行作为单位处理， 所以如果两个long l1, l2放在一起，那么一般会被CPU同时读取；  
### 4. 如果线程1对l1变量(volatile long)写入，那么它首先会加载l1所在的缓存行，这时该缓冲行中也包含l2，然后CPU会使其它加载了同一缓存行的线程中的缓存行无效，这样若线程2想写入l2变量，那么不得不再次从主存中取得该缓存行，相互之间造成了很大的冲突，所以使用了缓冲行填充的技巧：
``` Java
public long p1, p2, p3, p4, p5, p6, p7; // cache line padding
private volatile long cursor = INITIAL_CURSOR_VALUE;
public long p8, p9, p10, p11, p12, p13, p14; // cache line padding

```
