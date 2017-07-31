---
title: MyBatis缓存
date: 2017-07-31 22:47:35
tags: MyBatis  
categories: Database  
---

### MyBatis缓存  
#### 一级缓存  
默认开启一级缓存，一级缓存只是相对于同一个SqlSession而言的。在所有参数和SQL完全一样的情况下，使用同一个SqlSession对象调用同一个Mapper的方法时，只执行一次SQL，这是由于在第一次查询后，MyBatis会将其放入缓存中，以后再查询时，若没有显示声明刷新，并且缓存时间未超时时，SqlSession都只会取出当前缓存的数据，而不会再次发送SQL语句到数据库。  

#### 二级缓存  
二级缓存为克服一级缓存中出现同一条SQL语句但不同SqlSession时，会再次发送SQL语句到数据库的情况。它会使得缓存在SqlSessionFactory层面上能够提供给各个SqlSession对象共享。  

MyBatis默认不开启二级缓存，要开启二级缓存需在映射XML文件中加入<cache/>，cache有几个参数，默认未加入参数表示：  
- 映射语句文件中的所有select语句将会被缓存；  
- 映射语句文件中的所有insert, update和deete语句会刷新缓存；  
- 缓存会使用默认的LRU算法回收；  
- 缓存会存储列表集合或对象的1024个引用；  
- 缓存会被视为read/write的缓存，以为着对象检索不仅是共享的，而且是可以安全地被调用者修改，不干扰其它调用者或县城所做的潜在修改；  

**在添加cache后，还要做的事是POJO对象要实现Serializable接口，否则会抛出异常**  

它有以下属性：  
1. eviction: 代表缓存回收策略。  
	- LRU：最少使用，移除最长时间不用对象；  
	- FIFO: 先进先出；  
	- SOFT：软引用，移除基于垃圾回收期状态和软引用规则的对象；  
	- WEAK：弱引用，基于弱引用规则。  

2. flushInterval: 刷新间隔时间，单位毫秒；  
3. size: 引用数目，一个正整数，代表缓存最多可以存储多少个对象；  
4. readOnly：只读，缓存数据只能读取不能修改。  

#### 自定义缓存  
自定义缓存可使用现在流行的Redis缓存，自定义需要实现org.apache.ibatis.cache.Cache接口。  
该接口有如下方法：  

```Java
String getId(); // 获取缓存编号  
void putObject(Object key, Object value); //保存key/val缓存对象  
Object getObject(Object key); //获取缓存  
Object removeObject(Object key); //删除  
void clear(); //清空  
int getSize(); //获取缓存对象大小  
ReadWriteLock getReadWriteLock(); //获取缓存的读写锁
```  

则在使用时，将缓存类作为cache的type参数即可，如:  

```Java
<cache type="xx.yy.MyCache">
	<properti name="xxx", value="ggg">
</cache>
```  
