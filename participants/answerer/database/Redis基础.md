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

### Redis的skiplist
> 最近的一次被问到了Redis，但是不是很会答
> https://www.zhihu.com/search?type=content&q=redis%E4%B8%BA%E4%BB%80%E4%B9%88%E7%94%A8%E8%B7%B3%E8%A1%A8

skiplist与平衡树、哈希表的比较skiplist和各种平衡树（如AVL、红黑树等）的元素是有序排列的，而哈希表不是有序的。因此，
在哈希表上只能做单个key的查找，不适宜做范围查找。所谓范围查找，指的是查找那些大小在指定的两个值之间的所有节点。在做范围查找的时候，平衡树比skiplist操作要复杂。
在平衡树上，我们找到指定范围的小值之后，还需要以中序遍历的顺序继续寻找其它不超过大值的节点。如果不对平衡树进行一定的改造，这里的中序遍历并不容易实现。而在skiplist上进行范围查找就非常简单，只需要在找到小值之后，对第1层链表进行若干步的遍历就可以实现。平衡树的插入和删除操作可能引发子树的调整，逻辑复杂，而skiplist的插入和删除只需要修改相邻节点的指针，操作简单又快速。从内存占用上来说，skiplist比平衡树更灵活一些。
一般来说，平衡树每个节点包含2个指针（分别指向左右子树），而skiplist每个节点包含的指针数目平均为1/(1-p)，具体取决于参数p的大小。如果像Redis里的实现一样，取p=1/4，那么平均每个节点包含1.33个指针，比平衡树更有优势。查找单个key，skiplist和平衡树的时间复杂度都为O(log n)，大体相当；而哈希表在保持较低的哈希值冲突概率的前提下，查找时间复杂度接近O(1)，性能更高一些。所以我们平常使用的各种Map或dictionary结构，大都是基于哈希表实现的。从算法实现难度上来比较，skiplist比平衡树要简单得多。
Redis中的skiplist实现在这一部分，我们讨论Redis中的skiplist实现。在Redis中，skiplist被用于实现暴露给外部的一个数据结构：sorted set。准确地说，sorted set底层不仅仅使用了skiplist，还使用了ziplist和dict。这几个数据结构的关系，我们下一章再讨论。现在，我们先花点时间把sorted set的关键命令看一下。这些命令对于Redis里skiplist的实现，有重要的影响。sorted set的命令举例sorted set是一个有序的数据集合，对于像类似排行榜这样的应用场景特别适合。现在我们来看一个例子，用sorted set来存储代数课（algebra）的成绩表。原始数据如下：Alice 87.5Bob 89.0Charles 65.5David 78.0Emily 93.5Fred 87.5这份数据给出了每位同学的名字和分数。
![图片](https://pic1.zhimg.com/v2-6b8c4347d32ecb8d2b513a76d2e01884_r.jpg)
下面我们将这份数据存储到sorted set里面去：对于上面的这些命令，我们需要的注意的地方包括：前面的6个zadd命令，将6位同学的名字和分数(score)都输入到一个key值为algebra的sorted set里面了。注意Alice和Fred的分数相同，都是87.5分。zrevrank命令查询Alice的排名（命令中的rev表示按照倒序排列，也就是从大到小），返回3。排在Alice前面的分别是Emily、Bob、Fred，而排名(rank)从0开始计数，所以Alice的排名是3。注意，其实Alice和Fred的分数相同，这种情况下sorted set会把分数相同的元素，按照字典顺序来排列。按照倒序，Fred排在了Alice的前面。zscore命令查询了Charles对应的分数。zrevrange命令查询了从大到小排名为0~3的4位同学。zrevrangebyscore命令查询了分数在80.0和90.0之间的所有同学，并按分数从大到小排列。总结一下，sorted set中的每个元素主要表现出3个属性：数据本身（在前面的例子中我们把名字存成了数据）。每个数据对应一个分数(score)。根据分数大小和数据本身的字典排序，每个数据会产生一个排名(rank)。可以按正序或倒序。