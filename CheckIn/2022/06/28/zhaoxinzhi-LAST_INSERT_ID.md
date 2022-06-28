## 一、写在前面

关于select LAST_INSERT_ID()的使用场景和作用

https://blog.csdn.net/czd3355/article/details/71302441

https://blog.csdn.net/weixin_42271651/article/details/98784653



## 二、sql常用函数

### 1、使用场景

常常用于跟在insert语句后，作用是返回刚刚那条语句的第一个插入的元素。

常用组合拳：在mybatis中经常可以见到他。（就拿博客中的例子举例了）

```xml
<insert id="insertStudent" parameterType="com.czd.mybatis01.bean.Student">
    INSERT stu(name)VALUES (#{name})
    <selectKey keyProperty="id" order="AFTER" resultType="java.lang.Integer">
        SELECT LAST_INSERT_ID()
    </selectKey>
</insert>
```

### 2、为什么要用这个语句？

因为实际业务中，insert后往往想要知道自己插入的数据的主键id是多少（==提问：如果是多个字段做联合主键呢？==）所以sql的话，一般insert完还需要select一下id。

难点在于，主表的id自增以后，如何获取它对应的id，一般都是使用“select max(id) from table”语句，但是这种语句并没有考虑到并发的情况，需要在事务中添加“X锁”，待获取max(id)值以后再解锁，这种做法需要的步骤有些麻烦，而且也不能完全解决并发性问题。因此还有另外一种方法“select last_insert_id()”这个操作，它和“select max(id)”很像，但是它是线程安全的。也就是说它是具体于数据库连接的。

### 3、线程安全

last_insert_id()函数基于客户端独立原则。这意味着特定客户端last_insert_id()函数返回的值是该客户端生成的值。这样就确保每个客户端可以获得自己的唯一id。

> 参考链接：https://codingdict.com/questions/22791
>
> https://dev.mysql.com/doc/refman/5.5/zh-CN/information-
> functions.html#function_last-insert-
> id：
>
> > 生成的ID在 *每个连接* 的服务器中维护。这意味着函数返回给定客户端的值 *是为该客户端* 影响AUTO_INCREMENT列 *的*
> > 最新语句生成的第一个AUTO_INCREMENT值。即使其他客户端生成自己的AUTO_INCREMENT值，该值也不会受到其他客户端的影响。此行为可确保每个客户端都可以检索自己的ID，而不必担心其他客户端的活动，也不需要锁或事务。
>
> 因此，除非您为多个用户执行的插入操作恰巧是在同一数据库连接上完成的，否则您不必担心。

### 4、注意事项

> https://blog.csdn.net/q279838089/article/details/41080677

LAST_INSERT_ID()自动返回最后一个INSERT或UPDATE查询中AUTO_INCREMENT列设置的第一个表发生的值。

MySQL的LAST_INSERT_ID的注意事项：
 

第一、查询和插入所使用的Connection对象必须是同一个才可以，否则返回值是不可预料的。

第二、LAST_INSERT_ID是与表无关的，如果向表a插入数据后再向表b插入数据，LAST_INSERT_ID返回表b的Id值。  

第三、假如你使用一条INSERT语句插入多个行，LAST_INSERT_ID() 只返回插入的第一行数据时产生的值。

（==假如你使用多条INSERT语句插入多个行，LAST_INSERT_ID() 只返回最后一条插入的第一行数据时产生的值？==）

第四、假如你使用 INSERT IGNORE而记录被忽略，则AUTO_INCREMENT 计数器不会增量，而 LAST_INSERT_ID() 返回0, 这反映出没有插入任何记录。

 第五、在insert的时候必须不能显式的指明id，不然`LAST_INSERT_ID()`就会返回0。

> 即对于insert的字段，只有以下两种情况，才会符合第五条： 
>
> 1. ID字段为null，显式 插入。
> 2. 未指定ID字段， 隐式插入。

根据这五条原则，我们讨论的高并发网站访问时的插入后取自增长值其实主要是跟第一条规则和第二条规则有关。即要保证LAST_INSERT_ID的正确性，必须同一个connection，并且LAST_INSERT_ID要紧跟在insert中执行。所以如果是数据库缓存池公用connection可能会出问题，多线程操作在insert后面由执行了别的insert时也会出问题。



```sql
-- 注意点1：last_insert_id 是基于connection的，所有的操作都要在这个connection上进行才能取得Id
-- 注意点2：必须不能显式的指明id。（LAST_INSERT_ID仅适用于在auto_increment字段上创建的自动生成的主键。在下面这行的情况下，显式提供了id，因此未设置最后一个插入ID。 这是符合预期的。）
-- 错误：
INSERT INTO `zxz_test`.`task` (`id`, `tenant_id`, `table_code_id`) VALUES (4, 40, 4);
select LAST_INSERT_ID() as id  -- 输出的是0

-- 正确：
INSERT INTO `zxz_test`.`task` (`tenant_id`, `table_code_id`) VALUES (60, 6);
select LAST_INSERT_ID() as id
```

