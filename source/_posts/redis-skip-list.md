---
title: redis跳跃表和整数集合
date: 2017-04-24 23:14:39
tags: Redis
categories: Redis
---

[跳跃表-原理及Java实现](http://www.cnblogs.com/acfox/p/3688607.html)  
[SkipList 跳表](http://kenby.iteye.com/blog/1187303)  

Redis中使用跳跃表作为有序集合键的底层实现之一。  

```C
/* ZSETs use a specialized version of Skiplists */
/*
 * 跳跃表节点
 */
typedef struct zskiplistNode {

    // 成员对象
    robj *obj;

    // 分值，跳跃表中节点按各自所保存的分值从小到大排列
    double score;

    // 后退指针
    struct zskiplistNode *backward;

    // 层,跳跃表中使用层来加快访问其它节点
    struct zskiplistLevel {

        // 前进指针
        struct zskiplistNode *forward;

        // 跨度
        unsigned int span;

    } level[];

} zskiplistNode;

/*
 * 跳跃表
 */
typedef struct zskiplist {

    // 表头节点和表尾节点
    struct zskiplistNode *header, *tail;

    // 表中节点的数量
    unsigned long length;

    // 表中层数最大的节点的层数
    int level;

} zskiplist;
```

### 2. 整数集合  
整数集合是Redis用于保存整数值的集合抽象数据结构，它可以保存类型为int16_t, int32_t或int64_t的整数值，并且保证集合中不会出现重复元素。  

结构定义如下：  
```C
typedef struct intset {
    
    // 编码方式
    uint32_t encoding;

    // 集合包含的元素数量
    uint32_t length;

    // 保存元素的数组
    int8_t contents[];

} intset;
```  
encoding决定了contents内容的类型，如encoding为int16_t，那么contents将保存int16_t类型的值，contents中的内容按值从小到大排序，并且不包含重复数值。  

1. **升级**  
> 每当我们要将一个新元素添加到整数集合里面，并且新元素的类型比整数集合现有所有元素的类型都要长时，整数集合需要先升级，然后才能将新元素添加到整数集合里面。  

升级的好处：  
- 提升灵活性：避免类型错误；  
- 节约内存： 若最开始直接使用int64_t，那么会很浪费内存。  

