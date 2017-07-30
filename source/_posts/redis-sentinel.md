---
title: redis Sentinel
date: 2017-04-30 23:19:55
tags: Redis
categories: Redis
---

> Sentinel是Redis的高可用性解决方案：由一个或多个Sentinel实例组成的Sentinel系统可以监视任意多个主服务器，以及这些主服务器属下的所有从服务器，并在监视的主服务器进入下线状态时，自动将下线主服务器属下的某台从服务器升级为新的主服务器，然后由新的主服务器代替已下线的主服务器继续处理命令请求；  

所以Sentinel的功能肯定有**Master选举**，**故障转移**等；  

当监视到server1的下线时长超过用户设定的下线时长上线时，Sentinel系统就会对server1执行故障转移操作：  
- 首先，Sentinel系统会挑选server1属下的其中一种从服务器，并将这个被选中的从服务器升级为新的主服务器； 
- 之后，Sentinel系统会向server1属下的所有从服务器发送新的复制指令，让它们成为新的主服务器的从服务器，当所有的从服务器都开始复制新的主服务器时，故障转移操作执行完毕；  
- 另外，Sentinel还会继续监视已下线的server1，并在它重新上线时，将它设置为新的主服务器的从服务器。  

#### 1. 启动并初始化Sentinel  
使用redis-sentinel /path/to/your/sentinel.conf或redis-server /path/to/your/sentinel.conf --sentinel启动Sentinel

启动Sentinel时会执行以下步骤：  
- 初始化服务器；  
- 将普通Redis服务器使用的代码替换成Sentinel专用代码；  
- 初始化Sentinel状态；  
- 根据给定的配置文件，初始化Sentinel的监视主服务器列表；  
- 创建连向主服务器的网络连接。  

##### 1. 初始化服务器  
因为Sentinel本质上只是一个运行在特殊模式下的Redis服务器，所以第一步是初始化一个普通的Redis服务器；但不同的是Sentinel并不是使用数据库，所以不会载入RBD或AOF文件；  

##### 2. 使用Sentinel专用代码  

将一部分普通Redis服务器使用的专用代码替换成Sentinel专用代码。
Sentinel只使用PING, SENTINEL, INFO, SUBSCRIBE, UNSUBSCRIBE, PSUBSCRIBE和PUNSUBSCRIBE七个命令；  

##### 3. 初始化Sentinel状态  
接下来Sentinel会初始化一个sentinelState结构，这个结构保存了服务器中所有和Sentinel功能有关的状态。  

##### 4. 初始化Sentinel状态的masters属性  

Sentinel状态中的masters字典记录了所有被Sentinel监视的主服务器的相关信息，其中：  
- 字典的键是被监视主服务器的名字；  
- 字典的值则是被监视主服务器对应的sentinel.c/SentinelRedisInstance结构。  

##### 5. 创建连向主服务器的网络连接  
初始化最后一步时创建连向被监视主服务器的网络连接，Sentinel将成为主服务器的客户端，它可以向主服务器发送命令，并从命令回复中获取相关的信息。  
对于每个被Sentinel监视的主服务器来说，Sentinel会创建两个连向主服务器的网络连接：  
- 一个是命令连接，这个连接专门用于向主服务器发送命令，并接收命令回复；  
- 另一个是订阅连接，这个连接专门用于订阅主服务器的__sentinel__:hello频道。  


#### 2. 获取主服务器信息   
Sentinel默认会以每十秒一次的频率，通过命令连接向被监视的主服务器发送INFO命令，并通过分析INFO命令的回复来获取主服务器的当前信息。  

INFO信息中包含了以下两个方面的内容：  
- 一方面是关于主服务器本身的信息，包括run_id域记录的服务器运行ID，以及role域记录的服务器角色； 
- ；另一方面是关于主服务器属下所有从服务器信息，每个从服务器都由一个“slave”字符串开头的行记录，每行的ip=域记录了从服务器的IP地址，而port=域记录端口号。根据这些信息，Sentinel无需 用户提供从服务器的地址信息，就可以自动发现从服务器。   

Sentinel在分析INFO命令中包含的从服务器信息时，会检查从服务器对应的实例是否已经存在于slaves字典：  
- 如果从服务器对应的实例已经存在，那么Sentinel对从服务器的实例结构进行更新；  
- 如果从服务器对应的实例结构不存在，那么说明这个从服务器是新发现的，Sentinel会在slaves字典中为这个从服务器新创建一个实例结构；  

#### 3. 获取从服务器信息  

当Sentinel发现主服务器有新的从服务器出现时，Sentinel除了会为这个从服务器创建相应的实例结构外，Sentinel还会**创建连接到从服务器的命令连接和订阅连接**；  

在创建命令连接后，Sentinel在默认情况下，会以每十秒一次的频率通过命令连接向从服务器发送INFO命令，获得提取以下内容：  
- 从服务器的运行ID run_id.  
- 从服务器的角色role;  
- 主服务器的IP地址master_host，以及主服务器的端口号master_port；  
- 主从服务器的连接状态master_ling_status；  
- 从服务器的优先级slave_priority；  
- 从服务器的复制偏移量slave_repl_offset。 

#### 4. 向主服务器和从服务器发送信息  

默认情况下，Sentinel会以每两秒一次的频率，通过命令连接向所有的主从服务器发送以下格式的命令：  
PUBLISH __sentinel__:hello "<s_ip>, <s_port>, <s_runid>, <s_epoch>, <m_name>, <m_ip>, <m_port>, <m_epoch>"  
其中以s_开头的参数记录了Sentinel本身的信息，以m_开头的参数记录的则是主服务器的信息。  

#### 5. 接收来自主服务器和从服务器的频道信息  

对于每个与Sentinei连接的服务器，Sentinel即通过命令连接向sentinel__:hello频道发送信息，又通过订阅连接从服务器的__sentinel__:hello频道接收信息。  

对于监视同一服务器的多个Sentinel来说，一个Sentinel发送的信息会被其他Sentinel接收到，这些信息会被用于更新其他Sentinel队发送信息Sentinel的认知，也会被用于更新其他Sentinel对被监视服务器的认知。  


##### 1. 更新sentinels字典  
Sentinel为主服务器创建的实例结构中的sentinels字典保存了除Sentinel本身之外，所有同样监视这个主服务器的其他Sentinel的资料；  

当一个Sentinel接收到其他Sentinel发来的信息时，目标Sentinel会从信息中分析并提取以下两方面的参数：  
- 与Sentinel有关的参数：源Sentinel的IP地址、端口号、运行ID和配置纪元；  
- 与主服务器有关的参数：源Sentinel正在监视的主服务器名字，IP，PORT和配置纪元。  

目标Sentinel根据这些信息来更新自己的sentinels字典，若存在源Sentinel实例，则更新，否则创建相应的实例。  

##### 2. 创建连向其他Sentinel的命令连接  
当Sentinel通过频道订阅信息发现一个新的Sentinel时，他不仅会为新的Sentinel在sentinels字典中创建相应的实例结构，还会创建一个连向新Sentinel的命令连接，而新的Sentinel也同样会创建连向这个Sentinel的命令连接，最终形成相互连接的网络。  


#### 6. 检测主管下线状态  
在默认情况下，Sentinel会以每秒一次的频率向所有与它创建命令连接的主服务器、从服务器、其他Sentinel发送PING命令，并通过实例返回的PING命令回复来判断实例是否在线。  

有效回复有：实例返回-PONG、-LOADING、-MASTERDOWN回复的其中一种；  
无效回复为以上三种回复除外的任一种回复。  

如果在down-after-milliseconds毫秒内，连续向Sentinel返回无效回复，则对应的实例为主观下线状态。  

#### 7. 检测客观下线状态  

当Sentinel将一个主服务器判断为主观下线后，为确认这个主服务器是否真的下线了，它会向同样监视这一主服务器的其他Sentinel进行询问，看他们是否也认为主服务器已经进入了下线状态，当Sentinel从其他Sentinel那里接收到足够量(quorum值)的已下线判断后，Sentinel将从服务器判断为客观下线，并对主服务器进行故障转移操作。  

#### 8. 选举领头Sentinel(RAFT算法)  
![](http://note.youdao.com/yws/public/resource/12eeca1b84ec5008b7c369d478db936a/xmlnote/06D431B802B44BBAB77374F79FF59C7B/8146)  
![](http://note.youdao.com/yws/public/resource/12eeca1b84ec5008b7c369d478db936a/xmlnote/B20E3BC843C84259A92CF2EDD7D109A3/8148)  

#### 9. 故障转移  
当选出领头Sentinel后，领头Sentinel将对已下线的主服务器执行故障转移操作，包括以下三个步骤：  
- 在已下线主服务器属下的所有从服务器里面，挑选出一个从服务器，并将其转移为主服务器；  
- 让已下线主服务器属下的所有从服务器改为复制新的主服务器；  
- 将已下线主服务器设置为新主服务器的从服务器，当这个旧主服务器重新上线时，它就会成为新主服务器的从服务器。  

#### 重点回顾  

![](http://note.youdao.com/yws/public/resource/12eeca1b84ec5008b7c369d478db936a/xmlnote/7216576C93D4470898DF2C3206CBEF34/8169)  
![](http://note.youdao.com/yws/public/resource/12eeca1b84ec5008b7c369d478db936a/xmlnote/498EEA3057124D85B2B09220132B854F/8171)  

