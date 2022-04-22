## Redis 是什么？

- Redis 是一个开源（BSD 许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件；
- Redis 支持多种类型的数据结构，如 字符串（strings），散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） ，范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询；
- Redis 内置了复制（replication），LUA 脚本（Lua scripting），LRU 驱动事件（LRU eviction），事务（transactions）和不同级别的 磁盘持久化（persistence）；
- Redis 通过 哨兵（Sentinel） 和自动分区（Cluster）提供高可用性（high availability）。

## Redis 特性

- 速度快
  单节点读110000次/s，写81000次/s

- 数据存放内存中
  用 C 语言实现，离操作系统更近
  单线程架构，6.0 开始支持多线程（CPU、IO 读写负荷）

- 持久化
  数据的更新将异步地保存到硬盘（RDB 和 AOF）
  多种数据结构 - 不仅仅支持简单的 key-value 类型数据，还支持：字符串、hash、列表、集合、有序集合，
  支持多种编程语言

- 功能丰富
  HyperLogLog、GEO、发布订阅、Lua脚本、事务、Pipeline、Bitmaps，key 过期

- 简单稳定
  源码少、单线程模型

- 主从复制
  Redis 支持数据的备份（master-slave）与集群（分片存储），以及拥有哨兵监控机制。
  Redis 的所有操作都是原子性的，同时 Redis 还支持对几个操作合并后的原子性执行。

## Redis 典型使用场景

1. 缓存
2. 计数器
3. 消息队列
4. 排行榜
5. 社交网络

## Redis高并发原理

1. Redis 是纯内存数据库，一般都是简单的存取操作，线程占用的时间很多，时间的花费主要集中在 IO 上，所以读取速度快
2. Redis 使用的是非阻塞 IO，IO 多路复用，使用了单线程来轮询描述符，将数据库的开、关、读、写都转换成了事件，减少了线程切换时上下文的切换和竞争。
3. Redis 采用了单线程的模型，保证了每个操作的原子性，也减少了线程的上下文切换和竞争。
4. Redis 存储结构多样化，不同的数据结构对数据存储进行了优化，如压缩表，对短数据进行压缩存储，再如，跳表，使用有序的数据结构加快读取的速度。
5. Redis 采用自己实现的事件分离器，效率比较高，内部采用非阻塞的执行方式，吞吐能力比较大。