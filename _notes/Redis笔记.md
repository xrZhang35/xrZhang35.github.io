# Redis笔记

# Redis的基本特性

上线了一个Web应用，想做数据统计工作，意大利🇮🇹人创建了统计服务的项目，一开始用的是MySQL数据库，但是当数据量大起来之后，限制系统性能的是IO，于是它自己在内存中实现了这么一个列表结构的数据库并添加了持久化的功能。

Redis = remote dictionary service

可以看出关系型数据库不适合某些场景下的存储功能

|       | SQL              | NoSQL                            |
| ----- | ---------------- | -------------------------------- |
| SQL   | 行存储，二维     | 非结构化数据                     |
| NoSQL | 结构化，Schema   | 数据与数据之间没有关联           |
|       | 表与表之间的关联 | BASE，最终一致性                 |
|       | SQL语法          | 海量数据存储、高并发读写         |
|       | 事务，ACID       | 支持分布式：数据分片、扩缩容简单 |
|       |                  |                                  |

存在的问题

-   扩容，堆硬件，水平扩容需要分库分表等等
-   表结构修改困难
-   分布式大数据场景下读写压力大

Nosql常见的形式

-   KV存储
-   文档存储 MongoDB
-   列存储 HBase
-   图存储 
-   对象存储
-   XML存储

Redis的优点

-   跨进程、分布式
-   丰富的数据类型
-   功能丰富
-   编程语言支持多

## Redis安装

[青山老师的安装教程](https://gper.club/articles/7e7e7f7ff3g5bgccg69)

[Redis指令学习](http://redisdoc.com/)

## Redis客户端连接

平常我们是拿不到生产环境的Redis

客户端和服务端是需要进行序列化和反序列化的

例子写出来了，可以看idea项目

## [Redis的客户端](https://redis.io/docs/clients/)

-   Redission

    >   提供更高级的功能，提供分布式的容器，例如分布式的Map

-   Jedis

    >   简介轻量
    >
    >   多线程情况下可能出现干扰的情况，可以引入连接池来实现
    >
    >   Apache Common Pool来实现的连接池

-   lettuce

    >   解决了线程不安全的问题
    >
    >   利用netty来实现，可以选择同步异步

手写Socket --> Jedis连接 --> lettuce --> redission

这三个客户端有什么区别

## Redis数据类型

-   基本数据类型
    -   string
        -   string
        -   int
        -   float
    -   hash
    -   list
    -   set
    -   zset
-   高级数据类型
    -   bitmmap
    -   hyperloglogs
    -   geospatial
    -   streams

位图在redis中是怎么实现的呢？

>   字符在Redis里边是用8位二进制去存储的
>
>   `set k1 a`
>
>   `setbit k1 7 0`将第8位改为0
>
>   来学习一下位操作的其他值
>
>   `getbit k1 0`
>
>   `bitpos k1 0`第一个0出现的下标
>
>   `bitcount k1 0`计算到底有多少个0
>
>   位操作`bitop and or xor not`

>   String的应用场景
>
>   -   缓存
>
>       >   热点数据，读操作为主的数据可以放到缓存里边
>
>   -   分布式Session
>
>   -   set NX EX 分布式锁
>
>   -   incr 全局ID
>
>       -   分库分表，中间件来实现，这里就是redis
>       -   同时可以批量审批
>   -   incr 计数器
>
>   -   incr 限流
>   -   位操作 统计
>       -   例如要统计活跃度或者最后访问时间的时候，就可以用到未操作来实现统计

>   分布式锁的简单介绍
>
>   NX是能够实现竞争的，所以可以实现加锁
>
>   NX
>
>   del key-name
>
>   如果没有释放这把锁会造成死锁，所以需要加上过期时间
>
>   expire EX



### Hash



