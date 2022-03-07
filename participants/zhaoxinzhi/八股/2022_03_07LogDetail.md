## 一、写在前面

MySQL的各种log做详细的学习和整理。

## 二、日志

### 1、redo log

**redo log通常是物理日志**，记录的是数据页的物理修改，而不是某一行或某几行修改成怎样怎样，它用来恢复提交后的物理数据页(恢复数据页，且只能恢复到最后一次提交的位置)。

redo log是server层的（对比binlog是存储引擎层的）。用来实现ACID的持久性。

redo log包括两部分：

1、内存中的日志缓冲(redo log buffer)，该部分日志是易失性的；

2、磁盘上的重做日志文件(redo log file)，该部分日志是持久的。

在概念上，innodb通过force log at commit机制实现事务的持久性，即在事务提交的时候，必须先将该事务的所有事务日志写入到磁盘上的redo log file和undo log file中进行持久化。

为了确保每次日志都能写入到事务日志文件中，在每次将log buffer中的日志写入日志文件的过程中都会调用一次操作系统的fsync操作(即fsync()系统调用)。因为MariaDB/MySQL是工作在用户空间的，MariaDB/MySQL的log buffer处于用户空间的内存中。要写入到磁盘上的log file中(redo:ib_logfileN文件,undo:share tablespace或.ibd文件)，中间还要经过操作系统内核空间的os buffer，调用fsync()的作用就是将OS buffer中的日志刷到磁盘上的log file中。也就是说，从redo log buffer写日志到磁盘的redo log file中，过程如下：

![image-20220306144700878](D:\mystudy\internship\Cruel_Interview\participants\zhaoxinzhi\assets\2022_03_07LogDetail\image-20220306144700878.png)

> MySQL支持用户自定义在commit时如何将log buffer中的日志刷log file中。这种控制通过变量 innodb_flush_log_at_trx_commit 的值来决定。该变量有3种值：0、1、2，默认为1。但注意，这个变量只是控制commit动作是否刷新log buffer到磁盘。
>
> 当设置为1的时候，事务每次提交都会将log buffer中的日志写入os buffer并调用fsync()刷到log file on disk中。这种方式即使系统崩溃也不会丢失任何数据，但是因为每次提交都写入磁盘，IO的性能较差。
>
> 当设置为0的时候，事务提交时不会将log buffer中日志写入到os buffer，而是每秒写入os buffer并调用fsync()写入到log file on disk中。也就是说设置为0时是(大约)每秒刷新写入到磁盘中的，当系统崩溃，会丢失1秒钟的数据。
>
> 当设置为2的时候，每次提交都仅写入到os buffer，然后是每秒调用fsync()将os buffer中的日志写入到log file on disk。
>
> ![image-20220306144832173](D:\mystudy\internship\Cruel_Interview\participants\zhaoxinzhi\assets\2022_03_07LogDetail\image-20220306144832173.png)

#### 日志块：

innodb存储引擎中，redo log以块为单位进行存储的，每个块占512字节，这称为redo log block。所以不管是log buffer中还是os buffer中以及redo log file on disk中，都是这样以512字节的块存储的。

每个redo log block由3部分组成：日志块头、日志块尾和日志主体。其中日志块头占用12字节，日志块尾占用8字节，所以每个redo log block的日志主体部分只有512-12-8=492字节。

因为innodb存储引擎存储数据的单元是页，所以redo log也是基于页的格式来记录的。默认情况下，innodb的页大小是16KB(由 innodb_page_size 变量控制)，一个页内可以存放非常多的log block(每个512字节)，而log block中记录的又是数据页的变化。

### 2、undo log

undo log有两个作用：提供回滚和多个行版本控制(MVCC)。

**undo log记录的是逻辑日志。**可以认为当delete一条记录时，undo log中会记录一条对应的insert记录，反之亦然，当update一条记录时，它记录一条对应相反的update记录。

> 具体来说，
>
> ~~因为`insert`操作的记录，只对事务本身可见，对其他事务不可见。故该`undo log`可以在事务提交后直接删除，不需要进行`purge`操作~~（==这一点有待商榷？==）
>
> 而`Delete`操作在事务中实际上并不是真正的删除掉数据行，而是一种Delete Mark操作，在记录上标识`Delete_Bit`，而不删除记录。是一种"假删除",只是做了个标记，真正的删除工作需要后台`purge`线程去完成。（因为后续还可能会用到`undo log`，如隔离级别为`repeatable read`时，事务读取的都是开启事务时的最新提交行版本，只要该事务不结束，该行版本就不能删除（即`undo log`不能删除）,且`undo log`分配的页可重用减少存储空间和提升性能。）
>
> > 关于purge线程：两个主要作用：清理undo页和清除page里面带有Delete_Bit标识的数据行。
>
> `update`分为两种情况：`update`的列是否是主键列。
>
> - 如果不是主键列，在`undo log`中直接反向记录是如何`update`的。即`update`是直接进行的。
> - 如果是主键列，`update`分两部执行：先delete，再insert。

undo log也会产生redo log，因为undo log也要实现持久性保护。



### 3、binlog

> 参考链接：https://www.cnblogs.com/mao3714/p/8734838.html

这是mysql 的server层的，而另外两个是InnoDB存储引擎层的，也就是说不管哪个存储引擎都是有binlog的。

其实binlog主要是做主从复制的时候用的。

其中在每个事物提交的时候要遵从2阶段提交，原因就是避免主备数据不一致的情况。

> MySQL为了保证master和slave的数据一致性，就必须保证binlog和InnoDB redo日志的一致性（因为备库通过二进制日志重放主库提交的事务，而主库binlog写入在commit之前，如果写完binlog主库crash，再次启动时会回滚事务。但此时从库已经执行，则会造成主备数据不一致）。所以在开启Binlog后，如何保证binlog和InnoDB redo日志的一致性呢？为此，MySQL引入二阶段提交（two phase commit or 2pc），MySQL内部会自动将普通事务当做一个XA事务（内部分布式事物）来处理

