## 一、写在前面

binlog redolog undolog区别



## 二、mysql log

### 1、按照作用的区别

binlog：二进制日志是mysql-server层的。用于数据库的主从复制及数据的增量恢复ROW (行模式)、时间点恢复使用。

redo log：重做日志是InnoDB存储引擎层特有的，用来保证事务安全。

undo log：回滚日志保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读。undo log有两个作用：提供回滚和多个行版本控制(MVCC)。

### 2、按照定义的区别



redo log在事务没有提交前，每一个修改操作都会记录变更后的数据，保存的是物理日志->数据。

作用：确保事务的持久性。防止在发生故障的时间点，尚有脏页未写入磁盘，在重启mysql服务的时候，根据redo log进行重做，从而达到事务的持久性这一特性。内容：物理格式的日志，记录的是物理数据页面的修改的信息，其redo log是顺序写入redo log file的物理文件中去的。



binlog是一个二进制格式的文件，是mysql-server层的。用于记录用户对数据库更新的SQL语句信息，但对库表等内容的查询不会记录。由于是二进制文件，需使用mysql binlog解析查看。

记录哪条数据修改了，**记录的是修改的那条记录的全部数据**。（即使只更新了1个字段， binlog里也会记录所有字段的数据Statement (语句模式) ）每一条会修改数据的sql都会记录在binlog中。

作用：用于复制，在主从复制中，从库利用主库上的binlog进行重播，实现主从同步。



在数据修改的时候，不仅记录了redo，还记录了相对应的undo，如果因为某些原因导致事务失败或回滚了，可以借助该undo进行回滚。undo log是采用段(segment)的方式来记录的，每个undo操作在记录的时候占用一个undo log segment。

作用：保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读



### 3、redolog和binlog区别

> https://blog.csdn.net/xiangjai/article/details/117288603

1. **redo log 是 InnoDB 引擎特有的**；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。
2. redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给D=2这一行的C字段加1”。。
3. redo log 是循环写的，空间固定会用完（4个文件，每个文件1Ｇ）；binlog 是可以追加写入的。“追加写”是指 binlog文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。
4. binlog 日志没有 crash-safe 的能力，只能用于归档。而 redo log 来实现 crash-safe 能力。
5. **redo log 用于保证 crash-safe 能力。innodb_flush_log_at_trx_commit 这个参数设置成 1 的时候，表示每次事务的 redo log 都直接持久化到磁盘**。
6. **sync_binlog 这个参数设置成 1 的时候，表示每次事务的 binlog 都持久化到磁盘**。
7. binlog有两种模式，statement 格式是记sql语句， row格式是记录行的内容，记两条，更新前和更新后都有。



> https://blog.csdn.net/damokelisijian866/article/details/108551224
>
> 因此，由binlog和redo log的概念和区别可知：binlog日志只用于归档，只依靠binlog是没有crash-safe能力的。但只有redo log也不行，因为redo log是InnoDB特有的，且日志上的记录落盘后会被覆盖掉。因此需要binlog和redo log二者同时记录，才能保证当数据库发生宕机重启时，数据不会丢失。



### 4、一条update语句，redolog和binlog如何联动，保证持久性

> https://blog.csdn.net/xiangjai/article/details/117288603

执行：

```sql
update t set n = n+1 where id = 1
```

1. 执行器先找引擎取 id=1 这一行。id 是主键，引擎直接用树搜索找到这一行。如果
2. id=1这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。
3. 执行器拿到引擎给的行数据，把这个值加上 1，比如原来是 n，现在就是n+1，得到新的一行数据，再调用引擎接口写入这行新数据。
4. 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，此时 redo log 处于 prepare状态。然后告知执行器执行完成了，随时可以提交事务。
5. 执行器生成这个操作的 binlog，并把 binlog 写入磁盘。
6. 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成。

所以记住，先更新redolog，再更新binlog，再更新redolog，二阶段提交！

### 5、binlog使用场景

> https://blog.csdn.net/damokelisijian866/article/details/108551224

binlog的主要使用场景有两个，分别是主从复制和数据恢复。

主从复制：在Master端开启binlog，然后将binlog发送到各个Slave端，Slave端重放binlog从而达到主从数据一致。
数据恢复：通过使用mysqlbinlog工具来恢复数据。

### 6、binlog刷盘时机

> https://blog.csdn.net/damokelisijian866/article/details/108551224

对于InnoDB存储引擎而言，只有在事务提交时才会记录biglog，此时记录还在内存中，那么biglog是什么时候刷到磁盘中的呢？mysql通过sync_binlog参数控制biglog的刷盘时机，取值范围是0-N：

```
0：不去强制要求，由系统自行判断何时写入磁盘；
1：每次commit的时候都要将binlog写入磁盘；
N：每N个事务，才会将binlog写入磁盘。
```

从上面可以看出，sync_binlog最安全的是设置是1，这也是MySQL 5.7.7之后版本的默认值。但是设置一个大一些的值可以提升数据库性能，因此实际情况下也可以将值适当调大，牺牲一定的一致性来获取更好的性能。

### 7、redo log刷盘时机

redo log包括两部分：一个是内存中的日志缓冲(redo log buffer)，另一个是磁盘上的日志文件(redo log file)。mysql每执行一条DML语句，先将记录写入redo log buffer，后续某个时间点再一次性将多个操作记录写到redo log file。这种先写日志，再写磁盘的技术就是MySQL里经常说到的WAL(Write-Ahead Logging) 技术。

在计算机操作系统中，用户空间(user space)下的缓冲区数据一般情况下是无法直接写入磁盘的，中间必须经过操作系统内核空间(kernel space)缓冲区(OS Buffer)。因此，redo log buffer写入redo log file实际上是先写入OS Buffer，然后再通过系统调用fsync()将其刷到redo log file中，过程如下图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914172112886.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RhbW9rZWxpc2lqaWFuODY2,size_16,color_FFFFFF,t_70#pic_center)

==问题1：谁去调用【用户空间->OS Buffer】这一步？fsync函数应该是C语言调用的吧，写在mysql代码里的那种？==



### 8、redo log记录形式

redo log实际上记录数据页的变更，而这种变更记录是没必要全部保存，因此redo log实现上采用了大小固定（4个1G的文件），循环写入的方式，当写到结尾时，会回到开头循环写日志。如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/202009141715408.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RhbW9rZWxpc2lqaWFuODY2,size_16,color_FFFFFF,t_70#pic_center)

==问题2：上图展示的是redo log buffer还是 redo log file？如果是buffer，那file也是循环写入的吗？（我感觉应该是file，那buffer是什么样的结构？）==

==问题3：数据页刷盘是完全参照redo log吗？还是要结合binlog还是啥？具体什么方式？何时触发？==

同时我们很容易得知，在innodb中，既有redo log需要刷盘，还有数据页也需要刷盘，redo log存在的意义主要就是降低对数据页刷盘的要求。在上图中，write pos表示redo log当前记录的LSN(逻辑序列号)位置，check point表示数据页更改记录刷盘后对应redo log所处的LSN(逻辑序列号)位置。write pos到check point之间的部分是redo log空着的部分，用于记录新的记录；check point到write pos之间是redo log待落盘的数据页更改记录。当write pos追上check point时，会先推动check point向前移动，空出位置再记录新的日志。

### 9、redo log何时起作用（启动innondb的时候）

启动innodb的时候，不管上次是正常关闭还是异常关闭，总是会进行恢复操作。因为redo log记录的是数据页的物理变化，因此恢复的时候速度比逻辑日志(如binlog)要快很多。

==问题4：那万一数据页已经变化了呢？还是说redolog记录的数据页其实是逻辑数据页？==

重启innodb时，首先会检查磁盘中数据页的LSN，如果数据页的LSN小于日志中的LSN，则会从checkpoint开始恢复。

还有一种情况，在宕机前正处于checkpoint的刷盘过程，且数据页的刷盘进度超过了日志页的刷盘进度，此时会出现数据页中记录的LSN大于日志中的LSN，这时超出日志进度的部分将不会重做，因为这本身就表示已经做过的事情，无需再重做。

==问题5：checkpoint的刷盘是指的数据页的刷盘吗？也就是数据页的刷盘和redolog的刷盘是同时进行的？那数据页是根据什么刷盘的呢？他的LSN是哪来的？==

==问题6：如果redolog刷盘的时候，宕机了，那没写入的那部分不也是丢失了吗？还是说此时则判断整个事务都失败了。（不是有个二阶段提交吗？）==

==问题7：接问题6：我这个理解是正确的吗：就算redo log 刷盘成功了，但是没有置事务的标志位为commit，则还是认为事务没有提交成功？所以下次此次事务的redolog还是会重做？感觉也只有这种办法（置标志位）来判断事务是否真正写入成功了。==

==问题8：4G的空间，是因为单个事务肯定不会影响超过4G的空间吗？如果超过了咋办？他是循环写入的啊==

==问题9：接问题8：不管有几个事务，万一write pos追上了check point（就是check point写磁盘太慢了），那么会暂停执行当前事务吗？==



### 10、关于刷盘

binlog需要刷盘，redo log需要刷盘，数据页也需要刷盘，那undolog呢？
