## 一、写在前面

mysql使用规范和技巧

> 参考链接：https://blog.csdn.net/weixin_39534873/article/details/111200040

## 二、mysql



### 1、需要遵守的规范

1. 库名、表名、字段名必须使用小写字母，并采用下划线分割。

​	a) MySQL有配置参数lower_case_table_names，不可动态更改，Linux系统默认为 0，即库表名以实际情况存储，大小写敏感。如果是1，以小写存储，大小写不敏感。如果是2，以实际情况存储，但以小写比较。 
​	b) 如果大小写混合使用，可能存在abc，Abc，ABC等多个表共存，容易导致混乱。 
​	c) 字段名显示区分大小写，但实际使⽤不区分，即不可以建立两个名字一样但大小写不一样的字段。 
​	d) 为了统一规范，库名、表名、字段名使用小写字母。

2. 库名、表名、字段名禁止超过32个字符。 
   	库名、表名、字段名支持最多64个字符，但为了统一规范、易于辨识以及减少传输量，禁止超过32个字符。

3. 库名、表名、字段名禁止使用MySQL保留字。 

​		当库名、表名、字段名等属性含有保留字时，SQL语句必须用反引号引用属性名称，这将使得SQL语句书写、SHELL脚本中变量的转		义等变得⾮常复杂。

4. 禁止使用分区表。 

   ​	分区表对分区键有严格要求；分区表在表变大后，执⾏行DDL、SHARDING、单表恢复等都变得更加困难。因此禁止使用分区表，并建议业务端手动SHARDING。

### 2、用INNODB存储引擎

​	INNODB引擎是MySQL5.5版本以后的默认引擘，支持事务、行级锁，有更好的数据恢复能力、更好的并发性能，同时对多核、大内存、SSD等硬件支持更好，支持数据热备份等，因此INNODB相比MyISAM有明显优势。

### 3、用UNSIGNED存储非负数值

同样的字节数，非负存储的数值范围更大。如TINYINT有符号为 -128-127，无符号为0-255。

### 4、用INT UNSIGNED存储IPV4

用UNSINGED INT存储IP地址占用4字节，CHAR(15)则占用15字节。另外，计算机处理整数类型比字符串类型快。使用INT UNSIGNED而不是CHAR(15)来存储IPV4地址，通过MySQL函数inet_ntoa和inet_aton来进行转化。IPv6地址目前没有转化函数，需要使用DECIMAL或两个BIGINT来存储。 例如：

```sql
SELECT INET_ATON('209.207.224.40'); 
3520061480
SELECT INET_NTOA(3520061480);
209.207.224.40
```

### 5、使用TINYINT来代替ENUM类型

ENUM类型在需要修改或增加枚举值时，需要在线DDL，成本较高；ENUM列值如果含有数字类型，可能会引起默认值混淆。

### 6、用varbinary存储大小写敏感的变长字符串或二进制内容

VARBINARY默认区分大小写，没有字符集概念，速度快。

### 7、INT类型固定占用4字节存储

例如INT(4)仅代表显示字符宽度为4位，不代表存储长度。数值类型括号后面的数字只是表示宽度而跟存储范围没有关系，比如INT(3)默认显示3位，空格补齐，超出时正常显示，Python、Java客户端等不具备这个功能。

### 8、区分使用DATETIME和TIMESTAMP 

存储年使用YEAR类型。存储日期使用DATE类型。存储时间(精确到秒)建议使用TIMESTAMP类型。 
DATETIME和TIMESTAMP都是精确到秒，优先选择TIMESTAMP，因为TIMESTAMP只有4个字节，而DATETIME8个字节。同时TIMESTAMP具有自动赋值以及⾃自动更新的特性。

```sql
//进阶：
注意：在5.5和之前的版本中，如果一个表中有多个timestamp列，那么最多只能有一列能具有自动更新功能。如何使用TIMESTAMP的自动赋值属性? 
a)自动初始化，而且自动更新：column1 TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATECURRENT_TIMESTAMP
b)只是自动初始化：column1 TIMESTAMP DEFAULT CURRENT_TIMESTAMP
c)自动更新，初始化的值为0：column1 TIMESTAMP DEFAULT 0 ON UPDATE CURRENT_TIMESTAMP
d)初始化的值为0：column1 TIMESTAMP DEFAULT 0
```

### 9、索引字段均定义为NOT NULL

a) 对表的每一行，每个为NULL的列都需要额外的空间来标识。 

b) B树索引时不会存储NULL值，所以如果索引字段可以为NULL，索引效率会下降。 

c) 建议用0、特殊值或空串代替NULL值。

### 10、实用技巧

**1.分离冷热数据**

将大字段、访问频率低的字段拆分到单独的表中存储，分离冷热数据。有利于有效利用缓存，防⽌止读入无用的冷数据，较少磁盘IO，同时保证热数据常驻内存提⾼高缓存命中率。

**2.禁止在数据库中存储明文密码**

采用加密字符串存储密码，并保证密码不可解密，同时采用随机字符串加盐保证密码安全。

**3.表必须有主键，推荐使用UNSIGNED自增列作为主键。**

 表没有主键，INNODB会默认设置隐藏的主键列；没有主键的表在定位数据行的时候非常困难，也会降低基于行复制的效率。

**4.禁止冗余索引**

索引是双刃剑，会增加维护负担，增⼤大IO压力。(a,b,c)、(a,b)，后者为冗余索引。可以利用前缀索引来达到加速目的，减轻维护负担。

**5.禁止重复索引。**

`primary key a;uniq index a;`重复索引增加维护负担、占用磁盘空间，同时没有任何益处。

**6.不在低基数列上建立索引**

，例如“性别”。大部分场景下，低基数列上建立索引的精确查找，相对于不建立索引的全表扫描没有任何优势，而且增大了IO负担。

**7.合理使用覆盖索引减少IO，避免排序。** 

覆盖索引能从索引中获取需要的索引字段，从⽽而避免回表进行二次查找，节省IO。 INNODB存储引擎中，secondary index(非主键索引，又称为辅助索引、二级索引)没有直接存储行地址，而是存储主键值。 如果用户需要查询secondary index中所不包含的数据列，则需要先通过secondary index查找到主键值，然后再通过主键查询到其他数据列，因此需要查询两次。覆盖索引则可以在⼀一个索引中获取所有需要的数据，因此效率较高。 例如SELECT email，uid FROM user_email WHERE uid=xx，如果uid不是主键，适当时候可以将索引添加为index(uid，email)，以获得性能提升。

**8.用IN代替OR。SQL语句中IN包含的值不应过多，应少于1000个。**

 IN是范围查找，MySQL内部会对IN的列表值进行排序后查找，比OR效率更高。

**9.表字符集使用UTF8，必要时可申请使用UTF8MB4字符集。**

 a)UTF8字符集存储汉字占用3个字节，存储英文字符占用一个字节。 

b)UTF8统一而且通用，不会出现转码出现乱码风险。 

c)如果遇到EMOJ等表情符号的存储需求，可申请使用UTF8MB4字符集。

**10.用UNION ALL代替UNION。** 

UNION ALL不需要对结果集再进行排序。

**11.禁止使用order by rand()。** 

order by rand()会为表增加一个伪列，然后用rand()函数为每一行数据计算出rand()值，然后基于该行排序，这通常都会生成磁盘上的临时表，因此效率非常低。建议先使用rand()函数获得随机的主键值，然后通过主键 获取数据。

**12.建议使用合理的分页方式以提高分页效率。** 

```sql
假如有类似下面分页语句:

SELECT * FROM table ORDER BY TIME DESC LIMIT 10000，10;
这种分页方式会导致大量的io，因为MySQL使用的是提前读取策略。 
推荐分页方式:
SELECT * FROM table WHERE TIME ORDER BY TIME DESC LIMIT 10
SELECT * FROM table inner JOIN (SELECT id FROM table ORDER BY TIME LIMIT 10000，10) as t USING(id)
```

**13.SELECT只获取必要的字段，禁⽌止使用SELECT *** 

减少网络带宽消耗； 能有效利用覆盖索引； 表结构变更对程序基本无影响。

**14.SQL中避免出现now()、rand()、sysdate()、current_user()等不确定结果的函数。** 

语句级复制场景下，引起主从数据不一致；不确定值的函数，产⽣生的SQL语句无法利用QUERY CACHE。

**15.采用合适的分库分表策略。例如千库十表、十库百表等。** 

采用合适的分库分表策略，有利于业务发展后期快速对数据库进行水平拆分，同时分库可以有效利⽤用MySQL 的多线程复制特性。

**16.减少与数据库交互次数，尽量采用批量SQL语句。** 

使用下面的语句来减少和db的交互次数:
a)INSERT ... ON DUPLICATE KEY UPDATE

b)REPLACE INTO

c)INSERT IGNORE 

d)INSERT INTO VALUES()
**17.拆分复杂SQL为多个小SQL，避免大事务。** 

简单的SQL容易使⽤用到MySQL的QUERY CACHE；减少锁表时间特别是MyISAM；可以使用多核 CPU。

**18.对同一个表的多次alter操作必须合并为一次操作。**

 mysql对表的修改绝大部分操作都需要锁表并重建表，而锁表则会对线上业务造成影响。为减少这种影响，必须把对表的多次alter操作合并为一次操作。例如，要给表t增加一个字段b，同时给已有的字段aa建立索引

```sql
通常的做法分为两步：
alter table t add column b varchar(10);
alter table t add index idx_aa(aa);
正确的做法是：
alter table t add column b varchar(10),add index idx_aa(aa);

```



**19.避免使用存储过程、触发器、视图、自定义函数等。** 

这些高级特性有性能问题，以及未知BUG较多。业务逻辑放到数据库会造成数据库的DDL、SCALE OUT、 SHARDING等变得更加困难。

**20.禁止有super权限的应用程序账号存在。** 

安全第一。super权限会导致read only失效，导致较多诡异问题而且很难追踪。

**21.不要在MySQL数据库中存放业务逻辑。** 

数据库是有状态的服务，变更复杂而且速度慢，如果把业务逻辑放到数据库中，将会限制业务的快速发展。建议把业务逻辑提前，放到前端或中间逻辑层，而把数据库作为存储层，实现逻辑与存储的分离。

