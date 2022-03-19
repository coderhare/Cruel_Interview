# 实战场景case
# 网络相关
**https 访问出错，CA证书问题**
> [一张证书引发的噱案](https://mp.weixin.qq.com/s/WLc8mt964XmHK34XkuN1Ng)

# 操作系统相关
**一次OOM排查经历**
> [一次OOM排查经历](docs/%E4%B8%80%E6%AC%A1OOM%E6%8E%92%E6%9F%A5%E7%BB%8F%E5%8E%86.md)

# redis
**redis大key问题的排查**

**redis热key问题的排查**

# 消息队列
**消息队列消息重复消费排查**

**消息队列丢消息问题排查**

**消息队列消息积压问题排查**
-  现象
    - 在一次使用python语言消费kafka时，明明设置了多个消费者，但程序跑到最后只有一个消费者(甚至一个都没有)可以消费。扩容消费者是没有用的
- 排查过程
    - 通过在kafka侧查看连接的client，发现到最后只有一个client
    - 查看断开连接的消费者日志，发现有"disconnect"字段
    - 怀疑是heartbeat没有及时进行
    - 此问题是一个积累一定数量的消息然后进行多线程消费的commit上线之后导致(这种逻辑也有问题，一旦有线程挂掉或者某些消息消费失败，这些消息就丢了)
    - 尝试修改heartbeat_interval和consume_timeout都没有用
    - 统计积累消费一批处理的时间，发现达到几分钟甚至十分钟，原因是某些IO耗时*多个消息。超过rocketmq broker心跳最长时长要求(若两分钟未有心跳，则断开连接)
    - 定位原因是积累消息然后多线程消费导致消费者处理时间太长，而python由于全局GIL锁的存在，一个进程里只能跑一个线程，导致消费时间太长。多线程用处不大。
- 解决
    - 将原先的积累消息处理改回成一个个处理。
    - 扩容消费者，减少单个消息IO耗时，增加python mq框架的参数worker_num(多进程消费)

# mysql
**一次mysql查询失败的问题排查(加索引解决)**

**一次造成线上mysql服务主备同步延迟的事故**

**一次由长事务引发线上mysql服务压力大的事故**

**mysql的一次查询优化**


# ES
**一次es写入但未读到引起的问题**

# Go
> [Goroutine泄漏](https://mp.weixin.qq.com/s/vawFvXcSRFjgNxDk2NFfvg)

> [Go服务内存暴涨](https://mp.weixin.qq.com/s/U-xUtzD_iObqSn3WFwRLbg)

> [Go服务锁死](https://mp.weixin.qq.com/s/v7B2TQDQ5zDcE2qHeao1gA)

> [Go服务灵异panic](https://mp.weixin.qq.com/s/wmdmYDenmOY2un6ymlO6SA)









