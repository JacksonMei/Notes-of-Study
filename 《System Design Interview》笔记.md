

[TOC]

# 《System Design Interview》

# chapter1

## Single server setup

## Database

关系型数据库： MySQL、Oracle database

非关系型数据库： redis、Amazon DynamoDB， 当如下场景下使用非关系型数据库

+ 低时延（super-low latency）
+ 不需要关系数据
+ 只需要序列化和反序列化数据（js\xml\yaml等）
+ 需要保存大量数据

## Load balancer

负载均衡器给服务器集合中平均分配流量。

+ 为了更好的安全性，用户通过公共IP访问负载均衡器，而server与server间、server与负载均衡器间通过私有IP通信
+ 为了更好的可用性，负载均衡器可以防止单点server故障引起的系统宕机、流量峰值时候扩容

## Database replication

主从数据库设计， 主数据库负责写请求，多个从数据库负责读请求（一般读请求更多），从而提高数据库可靠性、可用性、读写性能。

## Cache tier

#### 功能

+ 更好的系统性能
+ 减少数据库负载
+ 缓存层可以独立扩容

<img src="C:\Users\m00416131\AppData\Roaming\Typora\typora-user-images\image-20210315190136722.png" alt="image-20210315190136722" style="zoom:150%;" />



#### 设计

+ 在频繁读的场景中使用，缓存数据不会持久化，因此不能保存重要数据
+ 过期策略，TTL时间太短会导致缓存系统从数据库中频繁读取
+ 数据一致性，缓存和database之间
+ 降低故障， 通过多服务器缓存来避免SPOF，同时超额配给voerpriovision 设置内存配给
+ 驱逐策略: 缓存满后新请求将导致部分数据被驱逐，Least-recently-used(LRU) 驱逐策略最常用，其他的像Least Frequently Used (LFU) or First in First Out (FIFO)

## CDN（content delivery network）

CDN是一种用于传输静态内容的地理上分布式服务器网络，传输内容如图片、视频、CSS、JS文件等。 离用户距离更近的CDN服务器会响应更快

+ 成本，不频繁使用的内容应该移出CDN
+ 合理设置缓存TTL， 太长会导致内容失鲜
+ CDN容错，CDN临时停机时，客户端应能检测到故障并从origin中请求资源
+ 使文件失效， 通过CDN供应商的CDN失效API，或使用内容对象版本管理

![image-20210315192412893](C:\Users\m00416131\AppData\Roaming\Typora\typora-user-images\image-20210315192412893.png)



## 服务器水平扩展

#### 有状态服务器

在服务器中保存中状态数据如用户session数据（用户只会被路由到有其session数据的server），此时虽然很好地实现了负载均衡，但是不利于放服务器扩缩容以及容错。

#### 无服务架构

将状态数据如session数据移出服务器到database如redis或关系数据库等持久化存储中，此时就可以通过增删server实现基于负载的web层自动扩缩容

![image-20210315200833026](C:\Users\m00416131\AppData\Roaming\Typora\typora-user-images\image-20210315200833026.png)

## 数据中心

当用户数量庞大且有地理分布广泛，需要在地理上就近建立数据中心（一组服务器和数据库）。

geoDNS可以实现域名解析为基于用户位置的ip地址，以便于分摊流量。 同时在某个数据中心停机期，流量会定向到其他数据中心

关键技术

+ 流量从重定向，GeoDNS可以用于重定向到用户就近数据中心
+ 数据同步，为解决故障场景数据丢失的问题，一个通用策略是多数据中心间备份数据
+ 测试和部署，通过自动化部署工具保证web和应用在各个数据中心的服务是一致的

## 消息队列

为了系统更加便利的独立扩容，各个组件间需要做到解耦，消息队列被很多分布式系统用于解决这个问题。



## 日志、指标、自动化

+ 日志分级，使用日志汇聚
+ 指标，收集不同类型指标有助于获取业务视野和理解系统健康状态

如下图，实际应该有多个数据中心

![image-20210315210231606](C:\Users\m00416131\AppData\Roaming\Typora\typora-user-images\image-20210315210231606.png)





## 数据库扩容

#### 纵向扩容

add more power(cpu、ram、disk等)

+ 有硬件限制，超大用户数据时，单服务不够用
+ SPOF风险更大
+ 纵向扩容总开销大，powerful server更贵

#### 水平扩容

+ 数据分片 shard database，每个分片共享数据定义方案
+ 分片策略（其实就是hash）最重要的因数是sharding key，决定数据保存在哪个分片中，需要保证均匀分配（evenly distributed data）
+ 分片策略并不完美，会带来如下问题
  + 重分片（单个shard不够用了）、分布不均导致shard耗尽
  + 热点关键问题， 可能需要为每个热点问题分片专用切片甚至更多分区
  + 连接和反规范化， 跨分片的连接问题可能需要 反规范化 才能在一张表上执行jion操作

![image-20210315204948851](C:\Users\m00416131\AppData\Roaming\Typora\typora-user-images\image-20210315204948851.png)

## Conclude

+ Keep web tier stateless
+ Build redundancy at every tier 
+ Cache data as much as you can
+ Support multiple data centers 
+ Host static assets in CDN
+ Scale your data tier by sharding 
+ Split tiers into individual services
+  Monitor your system and use automation tools



[TOC]

# System Design Interview 

## chapter5 一致性哈希

当数据太大而无法存储在一个节点机器上时，系统中需要多个这样的节点或机器。同时使用多个缓存中间件，每个节点上存储了一部分数据缓存，需要通过数据key值找到目标节点并获取数据，从而实现一定程度的负载均衡。

#### 传统取模方法

```c++
node_number = hash(key) % N  // 其中 N 为节点数。
```

先对key值进行hash计算后，对N取模得到目标节点标号

###### 缺点

+ 当节点增删时，由于节点数N的变化，大部分的key通过上述公式计算得到了不同的结果，因此之前的缓存数据就出现了大量失效，需要remap

  

#### 一致性哈希算法

算法如下:

1. 一致性哈希环，起点为0，终点为hash函数的最大值如2^32 -1；
2. 将各个key值对象和节点信息（如IP或节点名）经过hash计算后落在环上；
3. 对于每个对象，**在哈希环上顺时针查找距离这个对象的 hash 值最近的节点作为存储节点**，对于节点增删，该规则同样遵循，因此只影响该变化节点上的数据
4. 有时候存在数据在节点上分布不均的情况，此时可以为每个节点引入多个虚拟节点放置在哈希环上。虚拟节点越多，各个节点的负载标准差越小。



![image-20210317145309468](C:\Users\m00416131\AppData\Roaming\Typora\typora-user-images\image-20210317145309468.png)

参考:  [图解一致性哈希算法](https://segmentfault.com/a/1190000021199728)



[TOC]



# System Design Interview 

## chapter8  DESIGN A URL SHORTENER

仿照tinyurl, 网址太长（包含REST API格式的各个字段），需要裁短，同时支持重定向。

**时序图/数据模型图/模块流程图/层级架构图**

#### 需求分解

+ 输入长URL，返回短URL用于显示
+ 输入短URL，得到长URL再跳转到web server

#### 输入要求点

+ 短URL长度、格式（a-z,A-Z,0-9， 即2*26+10=72种/个）、保存时间、URL数量，从而计算出所需内存大小
+ 通过URL产生量、读写比例（如10:1）计算请求量QPS

#### 设计内容API Endpoints

1. URL重定向
   + 301 重定向，短URL和长URL有永久映射关系，短URL会缓存在browser中，后续相同的长URL不会再请求短址服务而是直接用缓存的短URL重定向到目标服务器。 此法有助于缓解短址服务器压力
   + 302重定向，短URL和长URL没有永久映射关系，每次都需要先请求短址服务，再重定向到目标服务器。 此法有助于跟踪点击率和点击来源

![image-20210317152713413](C:\Users\m00416131\AppData\Roaming\Typora\typora-user-images\image-20210317152713413.png)

2. URL缩短

   由于需要双向映射，有没有shortURL-LongURL双向解析的函数，所以需要保存shortURL-LongURL的映射表，总共35T太大，所以需要保存在关系数据库中id-shortURL-longURL的形式

#### 深入设计

shortURL的产生方法，35T数据可以用7位URL表示

##### hash方法

35T数据可以用7位URL表示，而 CRC32, MD5, or SHA-1 返回的hash值都不止7位，所以可能需要再缩短，比如只取前7位，此时可能会有hash冲突。

检测到hash冲突时通过添加pre字符串并再次hash来解决，可以通过布隆过滤器来优化检测冲突

![image-20210317170337961](C:\Users\m00416131\AppData\Roaming\Typora\typora-user-images\image-20210317170337961.png)

###### Base 62方法

由于DB表中ID是惟一产生的十进制数字。而十进制可以通过如下表达式表示,因此十进制数ID=11157（不超过35T）可以转化为2TX。

```
• 11157(10) = 2 x 62^2 + 55 x 62^1 + 59 x 62^0 = [2, 55, 59] -> [2, T, X] in base 62
```

![image-20210317171209434](C:\Users\m00416131\AppData\Roaming\Typora\typora-user-images\image-20210317171209434.png)



###### 综合考虑，hash方法不依赖数据库ID因而也不能获取下一个可用shortURL, 同时有hash冲突问题；而base62方法，依赖唯一ID的同时可以获取下一个可用shortURL（变长）。综上考虑base62.     base62的短码变长问题可以通过从指定ID数字开始递增即可，或者补0？

分布式系统产生唯一ID参见《Chapter 7: Design A Unique ID Generator in Distributed Systems》

![image-20210317171138404](C:\Users\m00416131\AppData\Roaming\Typora\typora-user-images\image-20210317171138404.png)

增加分布式缓存层，提高系统性能

![image-20210317171433721](C:\Users\m00416131\AppData\Roaming\Typora\typora-user-images\image-20210317171433721.png)







