

## 一、写在前面

今天开始学Redis。

## 二、Redis

### 1、nosql四大分类

![image-20220406152020515](/Users/bytedance/Library/Application Support/typora-user-images/image-20220406152020515.png)

### 2、Redis是什么？

redis默认端口号是6379

> 为什么默认端口号是6379？
>
> 科普一下：因为作者觉得有个女明星叫merz的很蠢，所以就用9键merz对应6379作为端口号

### 3、Redis一般用来做什么？

- 可以存储多种数据类型
- 可以持久化：rdb和aof
- 可以用于缓存
- 发布订阅系统  可以做一些消息队列的事情
- 地图信息分析
- 计时器计数器

### 4、官方自带的压测工具benchmark

如何使用：

```shell
>>> redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```

该工具会对每个指令都去测试，并且输出对应的测试指标。

如何解读：

![image-20220409175502285](/Users/bytedance/Library/Application Support/typora-user-images/image-20220409175502285.png)



### 5、Redis是单线程的

主要是因为Redis是基于内存操作的，CPU不是Redis的性能瓶颈，机器的内存和网络带宽才是性能瓶颈，所以是单线程。（避免了多线程的线程切换调度，浪费时间）

> 现在引入了多线程？
>
> redis6.0引入了对多线程为了提高网络IO读写性能，但是默认关闭。并且执行命令等核心模块还是单线程的。（即多线程是对应io操作搬运数据的时候多线程，工作线程还是就一个）

> 为什么单线程还这么快？
>
> 首先高性能的服务器不一定是多线程的，其次多线程也不一定比单线程快。
>
> （CPU 进行一次上下文切换时间是1500ns-2000ns之间）
>
> 多线程应用目的，为了提高资源利用率。在涉及到硬盘数据时，CPU在执行时会有很多时间阻塞到IO,采用多线程可以利用阻塞这段时间



### 6、Redis基本操作

redis不区分大小写命令。

一共有16个数据库（可以在conf中查看）。默认使用是0号数据库。

切换到2号数据库

```bash
127.0.0.1:6379> select 2 # 切换数据库
OK
127.0.0.1:6379[2]> desize  # 查看db大小（即该库中有结果值）
(integer) 0

127.0.0.1:6379[2]> set name zxz  # set和get常规操作

127.0.0.1:6379[2]> keys *  # 查看所有的key，注意有s！！！
1) "name"

127.0.0.1:6379[2]> flushdb # 清空当前库
127.0.0.1:6379[2]> flushall # 清空所有库
127.0.0.1:6379[2]> 
127.0.0.1:6379[2]> 


```

**exist key**：key是否存在于当前库中

**expire key x** ：给key设置x秒的有效期

**move key x**： 移动key 到指定x库里（目标库不能是当前库），通常用来做删除数据用。但是实际的数据不会随便删，比如0号库的数据不用了，移动到1号库中，当做逻辑删除。

**ttl key**：查看key剩余时间

**type key**：用于返回 key 所储存的值的类型

### 7、String类型

**incr key**：自增1

**decr key：**自减1（可以变为负数）

**incrby key x：**自增，步长为x

**decr key x：**自减，步长为x

getrange key L R ：类似subString。  [L, R]，左闭右闭！！，R可以是-1

setrange

setex key x value：  set expire。在分布式锁中常用。



setnx：set if not exist。如果set成功返回1，已经存在返回0，不覆盖原值！直接set的话是会覆盖原值的。



mset k1 v1 k2 v2：同时设置两个键值对[k1, v1] , [k2, v2] 

mget k1 k2：同时获取两个恶

msetnx  原子操作，要么一起成功，要么一起失败。（Redis的事务是不保证原子性的）

getset key value：将给定 key 的值设为 value ，并返回 key 的旧值(old value)。

如果要保存对象，可以把前缀都设置在key中，只有value当value。类似命名空间的概念，用:或其他字符分隔不同的层次。就不用字典的形式了（json的形式）



### 8、List类型

所有的list类型的命令都是以 “ L ” 或者 “ R ” 开头的。

list中每个元素可以重复

Redis里面的list可以被当做栈、队列、阻塞队列。

LIST底层是快速链表，快速链表相对于普通链表使用的是连续地址空间块，当数据过多时候，通过指针连接地址空间块。

**LPUSH key value：**从左边放入key这个队列。

**RPUSH key value**：从右边放入key这个队列。

**LRANGE key L R：**从左侧开始输出key队列的[L,R]

**LPOP key**：从左侧移除key队列的第一个值，并且返回这个值。

**Lindex key x**：从左侧获取key队列的第x个值

**Llen key**：返回key队列的长度

**Lrem key num value**：从左到右移除num个key队列的value值。

**Ltrim key start end**：对一个列表进行修剪(trim)，就是说，通过下标截取key这个队列让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。会修改key队列。

**rpoplpush key1 key2**：从key1移除最右边的元素，并且从key2最左边放入。

Lset key idx value：从key队列中下标idx的位置替换为value。（前提需要保证存在下标idx）

可以用队列做生产着消费者模式，这种阻塞队列。

消息队列这些都可以做 但是ACK 和消息可靠性投递就不能保证了。

### 9、Set类型

无需不重复集合

sadd key velue：往key这个set里添加value

smembers key：查看set中所有成员。

sismember key value：查看value是否是key中的成员。

scard key： 获取key这个set中的元素个数

srem key value：移除集合中的指定元素。

set的应用场景：

把A用户关注的人放在一个set中，把A用户的粉丝也放在一个set中，然后两个set取交集。

共同关注，公共爱好，二度好友（可以做推荐好友使用）

### 10、Hash类型

本质和string类型没有太大区别，只是把value换成了一个map  

hget

hset

hmset

hmget

hlen

hdel

hexist 判断hash中的指定字段是否存在

hincrby hash key x：给has的这个key自增x

注意：没有hdecrby方法，但是可以通过hincrby实现。

hash更适合对象信息的存储，比如用户信息，经常变动的信息。

### 11、Zset类型

ZRANGEBYSCORE SALARY -INF +INF

### 12、Geo

底层本质是Zset
