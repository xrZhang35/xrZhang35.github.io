# ZooKeeper笔记

基础：ZooKeeper的数据结点、Watch机制、ACL权限控制、Jute序列化

进阶：了解ZooKeeper服务器从创建到对外提供服务的整个过程，清楚会话在不同时期的不同状态，掌握ZooKeeper的会话管理策略和底层实现原理

高级：ZooKeeper集群的工作方式以及内部实现原理，重点介绍Leader群首选举算法，集群中的不同角色及功能

实战：分布式事务，Paxos等算法

## 一、基础

### 1. 数据节点

ZooKeeper数据模型最根本的功能就像是一个数据库。

我们可以利用`create`创建节点，节点看起来来就像树形目录一样

#### 1. znode节点类型与特性

-   持久节点

    即使创建该节点的客户端与服务端会话关闭了，该数据结点还是会存储在ZooKeeper服务器上，需要显式调用delete函数进行删除

-   临时节点

    顾名思义，会自动删除，也可以手动删除。

    该节点可以用来做服务集群内机器运行情况的统计

-   有序节点

    像自增id一样，可以用来看顺序

>   共性：每个结点都会维护
>
>   -   一个二进制数组，用来存储节点的数据、ACL访问控制信息、子节点数据
>   -   记录自身状态信息的字段stat

#### 2. 节点的状态结构

![](http://learn.lianglianglee.com/专栏/ZooKeeper源码分析与实战-完/assets/Ciqc1F6zbwWAVkt5AAC_yMQVCFo712.png)

#### 3. 数据节点的版本

在ZooKeeper中每个数据节点有3中类型的版本信息，为节点数据内容、子节点信息、ACL修改次数

#### 4. 使用ZooKeeper实现锁

-   悲观锁

    用户客户端争抢着来实现`/locks`节点

    但是会存在死锁的问题，所以可以将该节点设置为临时节点

-   乐观锁CAS

    ZooKeeper中的version值就可以用来实现乐观锁

    在 ZooKeeper 的底层实现中，当服务端处理 `setDataRequest` 请求时，首先会调用 `checkAndIncVersion` 方法进行数据版本校验。ZooKeeper 会从 `setDataRequest `请求中获取当前请求的版本 version，同时通过 `getRecordForPath` 方法获取服务器数据记录 nodeRecord， 从中得到当前服务器上的版本信息 currentversion。如果 version 为 -1，表示该请求操作不使用乐观锁，可以忽略版本对比；如果 version 不是 -1，那么就对比 version 和 currentversion，如果相等，则进行更新操作，否则就会抛出 BadVersionException 异常中断操作。

    ![](http://learn.lianglianglee.com/专栏/ZooKeeper源码分析与实战-完/assets/CgqCHl6yMBKAZzwGAABPrrtajyI575.png)

#### 5. 为什么ZooKeeper不采用相对路径查找

这是因为 ZooKeeper 大多是应用场景是定位数据模型上的节点，并在相关节点上进行操作。像这种查找与给定值相等的记录问题最适合用散列来解决。因此 ZooKeeper 在底层实现的时候，使用了一个 hashtable，即 `hashtableConcurrentHashMap<String, DataNode> nodes` ，用节点的完整路径来作为 key 存储节点数据。这样就大大提高了 ZooKeeper 的性能。

### 2. Watch机制

