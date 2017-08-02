---
title: TCPIP知识一
date: 2017-03-18 00:41:55
tags: TCPIP
categories: 网络
---
### 1.状态转移图
* ### 报文
![报文](http://coolshell.cn//wp-content/uploads/2014/05/TCP-Header-01.jpg) 
Sequence Number是包的序号，用来解决网络包乱序（reordering）问题。
Acknowledgement Number就是ACK——用于确认收到，用来解决不丢包的问题。
Window又叫Advertised-Window，也就是著名的滑动窗口（Sliding Window），用于解决流控的。
TCP Flag ，也就是包的类型，主要是用于操控TCP的状态机的。
* ### 状态转移图
![状态转移图](http://coolshell.cn//wp-content/uploads/2014/05/tcpfsm.png)
* ### TCP三次握手四次挥手
![TCP三次握手四次挥手](http://coolshell.cn//wp-content/uploads/2014/05/tcp_open_close.jpg)
* ### 为什么建链接要3次握手，断链接需要4次挥手？
* 对于建链接的3次握手，主要是要初始化Sequence Number 的初始值。通信的双方要互相通知对方自己的初始化的Sequence Number（缩写为ISN：Inital Sequence Number）——所以叫SYN，全称Synchronize Sequence Numbers。也就上图中的 x 和 y。这个号要作为以后的数据通信的序号，以保证应用层接收到的数据不会因为网络上的传输的问题而乱序（TCP会用这个序号来拼接数据）。
* 对于4次挥手，其实你仔细看是2次，因为TCP是全双工的，所以，发送方和接收方都需要Fin和Ack。只不过，有一方是被动的，所以看上去就成了所谓的4次挥手。如果两边同时断连接，那就会就进入到CLOSING状态，然后到达TIME_WAIT状态。下图是双方同时断连接的示意图（你同样可以对照着TCP状态机看）：
![](http://coolshell.cn//wp-content/uploads/2014/05/tcpclosesimul.png)
 
 































### 参考文章：http://coolshell.cn/articles/11564.html

