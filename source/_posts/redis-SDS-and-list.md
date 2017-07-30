---
title: 动态字符串SDS和链表
date: 2017-04-24 23:14:29
tags: Redis
categories: Redis
---

### 1、SDS实现  
Redis中使用SDS表示字符串值，这种结构是对c字符串的一种封装，使其支持O(1)时间获取字符串长度、动态扩展、更加安全等功能。定义如下：
```C
/*
 * 保存字符串对象的结构
 */
struct sdshdr {
    
    // buf 中已占用空间的长度
    int len;

    // buf 中剩余可用空间的长度
    int free;

    // 数据空间
    char buf[];
};
```

### 2. 与c字符串的比较  
- **能常数复杂度获取字符串长度**：通过len字段能常数时间获得字符串的长度。  
- **杜绝缓冲区溢出**：SDS字符串中，对于例如c字符串的strcat等操作，SDS能动态的进行空间分配，从而避免源字符串空间不够而引发的缓冲区溢出问题；  
- **减少修改字符串时带来的内存重分配次数**：SDS实现了空间预分配和惰性空间释放两种优化方案：
    - **空间预分配**：  
        1. 如果对SDS进行次改之后，SDS的长度将小于1MB，那么程序分配和len属性同样大小的未使用空间；  
        2. 若大于1MB，那么程序将分配1MB未使用空间。    
    - **惰性空间释放**：在SDS的API要缩短字符串时，程序并不立即使用内存重分配来回收缩短后多出来的字节，而是使用free属性将这些字节的数量记录起来，并等待将来使用。(当然也可以使用相应API真正释放未使用空间)。    
- **二进制安全**：  很多c字符串函数以""\0"作为字符串结束标志，而使用SDS无须担心因为字符串中含有“\0”字符的情况，因为它使用len字段进行操作；  
- **兼容部分c字符串函数**：可直接取得SDS结构体中字符数组。  

![image](http://note.youdao.com/yws/api/personal/file/35B84ABEF3DC455287219A8A2B208D5B?method=download&shareKey=1e9e5404ae7d10614191cd16a60ba935)  


### 2. 链表实现  

```c
*
 * 双端链表节点
 */
typedef struct listNode {

    // 前置节点
    struct listNode *prev;

    // 后置节点
    struct listNode *next;

    // 节点的值
    void *value;

} listNode;

/*
 * 双端链表结构
 */
typedef struct list {

    // 表头节点
    listNode *head;

    // 表尾节点
    listNode *tail;

    // 节点值复制函数
    void *(*dup)(void *ptr);

    // 节点值释放函数
    void (*free)(void *ptr);

    // 节点值对比函数
    int (*match)(void *ptr, void *key);

    // 链表所包含的节点数量
    unsigned long len;

} list;

```
