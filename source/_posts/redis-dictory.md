---
title: redis字典
date: 2017-04-24 23:14:59
tags: Redis
categories: Redis
---

### 1. 字典实现  
- hash表结构  
```C
/*
 * 哈希表
 *
 * 每个字典都使用两个哈希表，从而实现渐进式 rehash 。
 */
typedef struct dictht {
    
    // 哈希表数组
    dictEntry **table;

    // 哈希表大小
    unsigned long size;
    
    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;

    // 该哈希表已有节点的数量
    unsigned long used;

} dictht;
```

- hash表节点：保存了key/value对  
```C
/*
 * 哈希表节点
 */
typedef struct dictEntry {
    
    // 键
    void *key;

    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;

    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;

} dictEntry;
```

- 字典  
```C
/*
 * 字典
 */
typedef struct dict {

    // 类型特定函数
    dictType *type;

    // 私有数据
    void *privdata;

    // 哈希表
    dictht ht[2];

    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */

    // 目前正在运行的安全迭代器的数量
    int iterators; /* number of iterators currently running */

} dict;
```
其中dict结构中，使用了两个hash表，这个是用于扩容时进行替换的表结构，rehashidx用于渐进式扩容，记录了目前的进度。  


### 2. 部分细节问题  
#### 1. hash算法采用了MurmurHash算法；  
#### 2. 解决键冲突的方式是使用链地址法，即dictEntry结构中有一个next指针用于链接；  
#### 3. rehash步骤如下：  
- 1. 为字典的ht[1]哈希表分配空间，这个哈希表的空间大小取决于要执行的操作，以及ht[0]当前包含的键值对数量(used)： 
    
    - 如果执行的是扩展操作，那么ht[1]的大小为**第一个大于等于ht[0].used*2的2的n次幂**；  
    - 如果执行的是收缩操作，那么ht[1]的大小为**第一个大于等于ht[0].used的2的n次幂**。  
- 2. 将保存在ht[0]中的所有键值对rehash到ht[1]上面：rehash指的是重新计算键的哈希值得索引值，然后将键值对放置到ht[1]哈希表的指定位置上。  
- 3. 当ht[0]包含的所有键值对都迁移到了ht[1]之后，释放ht[0]，将ht[1]设置成ht[0]，并在ht[1]新创建一个空白哈希表，为下一次rehash做准备。  
    
#### 4. 哈希表的扩展和收缩  
当以下条件中任意一个满足时，程序会自动开始对哈希表执行扩展操作：  
- 服务器目前**没有**在执行**BGSAVE命令或者BGREWRITEAOF**命令，并且哈希表的负载因子大于等于1.  
- 服务器目前**正在**执行**BGSAVE命令或者BGREWRITEAOF**命令，并且哈希表的负载因子大于等于5.  

#### 5. 渐进式rehash  
为避免rehash对服务器性能造成影响，服务器不是一次性将ht[0]里面的所有键值对全部rehash到ht[1]，而是渐进式的方式：  
- 为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两个哈希表；  
- 在字典中维持一个索引计数器变量rehashidx，并将它的值设置为0，表示rehash工作正式开始；  
- 在rehash进行期间，每次对字典执行添加、删除、查找或者更新操作时，程序处理执行指定的操作之外，还会顺带将ht[0]哈希表在rehashidx索引上的所有键值对rehash到ht[1]，当rehash工作完成后，程序将rehashidx属性的值增一；  
- 随着字典操作的不断执行，最终在某个时间点上，ht[0]的所有键值对都会被rehash至ht[1]，这时程序将rehashidx属性的值设为-1，表示rehash操作已完成。  

> 渐进式rehash的好处在于它采取分而治之的方式，将rehash键值对所需的计算工作均摊到对字典的每个添加、删除、查找和更新操作上，从而避免了集中式rehash而带来的庞大计算量。  

