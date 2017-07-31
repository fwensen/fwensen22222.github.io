---
title: redis对象
date: 2017-04-24 23:15:05
tags: Redis
categories: Redis
---

Redis的对象包括：字符串对象、列表对象、哈希对象、集合对象以及有序集合对象；
Redis的对象系统实现了基于引用计数技术的内存内存回收机制。此外使用了引用计数实现了对象共享。    
### 1. 定义     
```Java  
typedef struct redisObject {
	// 类型
	unsigned type:4;

	// 编码
	unsigned encoding:4;

	// 对象最后一次被访问的时间
	unsigned lru:REDIS_LRU_BITS; /* lru time (relative to server.lruclock) */

	// 引用计数
	int refcount;

	// 指向实际值的指针
	void *ptr;
} robj;  
```  
  
#### 1. 类型     
键总是一个字符串对象，而值可以是任何一种对象，使用**TYPE命令**可以查查看给定键的值对应的值对象类型。  
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/224EBB316DFA4F65B2AD366BE6DEE9BF/7139)  
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/8130E3E151934C98A569F56880CE6180/7144)  

#### 2. encoding属性  
该属性记录了对象所使用的编码技术。  
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/A33C382A00EA483190DF28F437DFF7A9/7151)  

使用**OBJECT ENCODING**命令可以查看一个数据键的值对象所使用的编码。  
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/4243F2F5458D41829BD3D3EF00B01209/7160)  

### 2. 字符串对象  

字符串对象的编码可以是int、raw或embstr；  
- 如果字符串对象保存的是一个字符串值，并且这个字符串值的长度大于32字节，那么字符串对象将使用一个简单动态字符串对象（SDS）来保存，编码设置为raw;  
- 如果字符串对象保存的是一个字符串值，并且长度小于等于32字节，那么将使用embstr编码方式来保存这个字符串值；  
- int编码的字符串对象和embstr编码的字符串在条件满足的情况下，会被转换成raw编码的字符串对象，例如int 字符串执行一些命令后该对象不再是整数值，而是字符串值，那么将变为字符串对象；  

![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/F7267FBE66E84385A9BEDF178CC24BBC/7194)  

### 3. 列表对象  
列表对象的编码可以是ziplist或者linkedlist;  
如执行RPUSH numbers 1 "three" 5后，，则使用ziplist和linkedlist编码分别如下图示：  
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/710AC207C51843398B0CC1C9BF629287/7208)  

- 当列表对象同时满足以下两个条件时使用ziplist编码:  
    1. 列表对象保存的所有字符串元素长度都小于64字节；  
    2. 列表对象保存的元素数量小于512个。  
不能满足上面两个条件都将使用linkedlist编码。但不过默认值可以修改，具体见配置文件。  
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/688DC1A85ECB4B60A445BC92B2ACC8F8/7224)  

### 4. 哈希对象  

哈希对象编码可以是ziplist或者hashtable;  

1. 使用ziplist作为编码方式时，会将键和值放在一起存储，键在前值在后，以FIFO的方式放入；  
2. 使用hashtable作为编码方式时，每个键都是一个字符串对象，每个值都是一个字符串对象； 
举例如下：    

```Java  
HSET profile name "Tom" 
HSET profile age 25  
HSET profile career "Programmer"
```   

![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/88DB715E43EB4A59A353E7733CBE5E6E/7244)  
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/A1FF84E8F94443AE801EC2DE4764A313/7246)  

3. 当哈希对象同时满足以下两个条件时，哈希对象使用ziplist:  
    - 保存的所有键值对的键和值的字符串长度都小于64字节；  
    - 键值对数量小于512个；不能满足以上两个中任一条件都将使用hashtable编码。同样也可以更改配置文件的方式更改默认值。  

![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/0FC7714CB1EF4867817CDE76601097F1/7267)  

### 5. 集合对象  
1. 集合对象可以使用intset或者hashtable；  
2. intset编码的集合对象使用整数集合作为底层实现，集合对象包含的都将存在整数集合里面；  
3. 另一方面，hashtable编码则是每一个键值都是一个字符串对象，每个字符串对象包含了一个集合元素，而字典的值全为NULL;   
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/C02B4C0C37B6457A8BEB8D9D4B48DEF5/7286)  

4. 当集合对象同时满足以下两个条件，对象使用intset编码。否则使用hashtable编码：  
    - 集合对象保存的所有元素都是整数值； 
    - 集合对象保存的元素数量不超过512个。当然可在配置文件中更改默认值。  
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/881EC036D273434D89409AE263EABAD0/7296)      
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/AF8D4DEB24EA4C08811AE700FE0EA547/7298)  

### 6. 有序集合  
1. 有序集合的编码可以是ziplist或者skiplist；
    - 使用ziplist作为底层编码，则会在每个集合元素中使用两个紧挨在一起的压缩列表节点来保存，第一个节点保存元素的成员，第二个保存元素的分值，压缩列表内的集合元素按分值从小到大进行排序，分值较小的元素靠前；  
    - 而若是使用skiplist作为底层编码，则内部会使用zset作为编码实现，每个zset中包含两个指针：一个是zskiplist指针另一个是dict指针，其中zskiplist跳跃表按分值从小到大保存了所有元素，每个跳跃节点都保存了一个集合元素，object属性保存了元素成员，score成员保存了元素分值。另外dict指针部分为有序集合创建了一个从成员到分值的映射。
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/895B6C8A994744BCBD3EE2E31C030266/7304)  

举例：  
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/73ADD98A5A0941729A639BB7FF2B1A5D/7310)  
![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/FFC8C1B1A3784EEB8FFE6FCE9050BC9A/7312)  

2. 当有序集合同时满足以下两个条件时，使用ziplist，否则使用跳跃表实现：  
    - 有序集合保存的元素数量小于128个；  
    - 有序集合保存的所有元素成员的长度都小于64字节。
    - 以上两个默认值可在配置文件中更改。  

![](http://note.youdao.com/yws/public/resource/ec373eb8fb08bab5885484b5dcf7aea9/xmlnote/0866CB807DE24C8F9E069C43AB56C52B/7318)  

### 6. 对象共享  
Redis会共享值为0到9999的字符串对象。 
