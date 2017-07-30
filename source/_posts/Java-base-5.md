---
title: Java基础五
date: 2017-04-04 23:10:52
tags: Java
categories: Java
---

### 1. 容器  
[Java中的容器讲解](http://blog.csdn.net/wwww1988600/article/details/8646191)  
[Java容器详细解析](http://www.cnblogs.com/LipeiNet/p/5888513.html)  
[Java集合源码分析](http://blog.csdn.net/column/details/collection.html?page=1) 

![](http://img.blog.csdn.net/20140628144205625)  


#### 1. ArrayList  

注意几点：  
    
- ArrayList默认大小为10；  
- ArrayList扩容(ensureCapacity)时大小为旧空间的3/2倍加1；  
- 允许null值的查找;  
- 尽量别删除ArrayList中的元素，因为ArrayList是数组实现，若删除则会造成大量数据的复制。  

#### 2. LinkedList

注意几点：

- LinkedList的实现是基于双向循环链表的；  
- 在查找和删除某元素时，源码中都划分为该元素为null和不为null两种情况来处理，LinkedList中允许元素为null；  
- LinkedList是基于链表实现的，因此不存在容量不足的问题，所以这里没有扩容的方法；  
- 注意源码中的Entry<E> entry(int index)方法。该方法返回双向链表中指定位置处的节点，而链表中是没有下标索引的，要指定位置出的元素，就要遍历该链表，从源码的实现中，我们看到这里有一个加速动作。源码中先将index与长度size的一半比较，如果index<size/2，就只从位置0往后遍历到位置index处，而如果index>size/2，就只从位置size往前遍历到位置index处。这样可以减少一部分不必要的遍历，从而提高一定的效率（实际上效率还是很低）；  
- LinkedList是基于链表实现的，因此插入删除效率高，查找效率低  
- 


#### 3. HashMap  

[谈谈HashMap线程不安全的体现](https://my.oschina.net/hosee/blog/673521)    
[疫苗：JAVA HASHMAP的死循环](http://coolshell.cn/articles/9606.html)  

1. HashMap为什么线程不安全？  

在HashMap扩容时，会调用一个transfer函数，该函数中若转移前链表顺序是1->2->3，那么转移后就会变成3->2->1。
这时候就有点头绪了，死锁问题不就是因为1->2的同时2->1造成的吗？所以，HashMap 的死锁问题就出在这个transfer()函数上。  
### 待补充  


#### 4. TreeMap  
1、TreeMap是根据key进行排序的，它的排序和定位需要依赖比较器或覆写Comparable接口，也因此不需要key覆写hashCode方法和equals方法，就可以排除掉重复的key，而HashMap的key则需要通过覆写hashCode方法和equals方法来确保没有重复的key。  
2、TreeMap的查询、插入、删除效率均没有HashMap高，一般只有要对key排序时才使用TreeMap。  
3、TreeMap的key不能为null，而HashMap的key可以为null。  


#### LinkedHashMap  
[【Java集合源码剖析】LinkedHashmap源码剖析](http://blog.csdn.net/ns_code/article/details/37867985)  
