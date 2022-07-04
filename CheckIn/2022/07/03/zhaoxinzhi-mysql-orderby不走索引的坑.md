## 一、写在前面

mysql-orderby导致使用错索引以及解决办法

> 参考链接：https://mp.weixin.qq.com/s/x2du0S2k0q6PvhVRd1_tSA
>
> 通过这篇文章，可以学习到
>
> 1、mysql中，orderby的坑，所以要慎用！
>
> 2、对于这个坑，有什么解决办法
>
> 3、mysql5.6以后的版本中，可以使用optimizer_trace功能来查看优化器生成执行计划的整个过程
>
> 4、走主键索引不意味着就不进行全表扫描，即执行计划中，type=index，key=PRIMARY就是全表扫描！根据这一点进一步理解为什么需要索引。
>
> 5、结合具体实例如何分析explain执行计划

## 二、mysql

### 1、事件描述以及确定原因

事件就是，一个sql的查询超时，原因就是，没有走预定的索引，而是直接走的主键（即全表查询！）

超时的sql如下：

```sql
select * from order_info where uid = 5837661 order by id asc limit 1
```

执行`show create table order_info` 发现这个表其实是有加索引的

```sql
CREATE TABLE `order_info` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `uid` int(11) unsigned,
  `order_status` tinyint(3) DEFAULT NULL,
  ... 省略其它字段和索引
  PRIMARY KEY (`id`),
  KEY `idx_uid_stat` (`uid`,`order_status`),
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

理论上执行上述 SQL 会命中 idx_uid_stat 这个索引，即不会很慢，但实际执行 explain 查看

```sql
explain select * from order_info where uid = 5837661 order by id asc limit 1
```

可以看到它的 possible_keys（此 SQL 可能涉及到的索引） 是 idx_uid_stat，但实际上（key）用的却是全表扫描

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/OyweysCSeLWPpdPHibhkib1QA0pJwNiagjbPfGYxOa65u1y3ax0ssCHwJlYnnibdFtJ8eHVDEsElxYnFicuEsSciaibRg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

我们知道 MySQL 是基于成本来选择是基于全表扫描还是选择某个索引来执行最终的执行计划的，所以看起来是全表扫描的成本小于基于 idx_uid_stat 索引执行的成本，不过我的第一感觉很奇怪，这条 SQL 虽然是回表，但它的 limit 是 1，也就是说只选择了满足 uid = 5837661 中的其中一条语句，就算回表也只回一条记录，这种成本几乎可以忽略不计，优化器怎么会选择全表扫描呢。



### 2、使用optimizer_trace进一步分析没走索引的原因

为了查看 MySQL 优化器为啥选择了全表扫描，我打开了 optimizer_trace 来一探究竟

> 在MySQL 5.6 及之后的版本中，我们可以使用 optimizer trace 功能查看优化器生成执行计划的整个过程



```sql
SET optimizer_trace="enabled=on";        // 打开 optimizer_trace
SELECT * FROM order_info where uid = 5837661 order by id asc limit 1
SELECT * FROM information_schema.OPTIMIZER_TRACE;    // 查看执行计划表
SET optimizer_trace="enabled=off"; // 关闭 optimizer_trace
```

MySQL 优化器首先会计算出全表扫描的成本，然后选出该 SQL 可能涉及到的所有索引并且计算索引的成本，然后选出所有成本最小的那个来执行，来看下 optimizer trace 给出的关键信息

```json
{
  "rows_estimation": [
    {
      "table": "`rebate_order_info`",
      "range_analysis": {
        "table_scan": {
          "rows": 21155996,
          "cost": 4.45e6    // 全表扫描成本
        }
      },
      ...
      "analyzing_range_alternatives": {
          "range_scan_alternatives": [
          {
            "index": "idx_uid_stat",
            "ranges": [
            "5837661 <= uid <= 5837661"
            ],
            "index_dives_for_eq_ranges": true,
            "rowid_ordered": false,
            "using_mrr": false,
            "index_only": false,
            "rows": 255918,
            "cost": 307103,            // 使用idx_uid_stat索引的成本
            "chosen": true
            }
          ],
       "chosen_range_access_summary": {    // 经过上面的各个成本比较后选择的最终结果
         "range_access_plan": {
             "type": "range_scan",
             "index": "idx_uid_stat",  // 可以看到最终选择了idx_uid_stat这个索引来执行
             "rows": 255918,
             "ranges": [
             "58376617 <= uid <= 58376617"
             ]
         },
         "rows_for_plan": 255918,
         "cost_for_plan": 307103,
         "chosen": true
         }
         }  
    ...
```

可以看到全表扫描的成本是 4.45e6，而选择索引 idx_uid_stat 的成本是 307103，远小于全表扫描的成本，而且从最终的选择结果（chosen_range_access_summary）来看，确实也是选择了 idx_uid_stat 这个索引，但为啥从 explain 看到的选择是执行 PRIMARY 也就是全表扫描呢，难道这个执行计划有误？其实并不是的，看继续分析。

### 3、继续分析执行计划

执行计划中有一个 reconsidering_access_paths_for_index_ordering

```json
{
    "reconsidering_access_paths_for_index_ordering": {
    "clause": "ORDER BY",
    "index_order_summary": {
      "table": "`rebate_order_info`",
      "index_provides_order": true,
      "order_direction": "asc",
      "index": "PRIMARY",    // 可以看到选择了主键索引
      "plan_changed": true,
      "access_type": "index_scan"
        }
    }
}
```

这个选择表示**由于排序的原因**再进行了一次索引选择优化，由于我们的 SQL 使用了 id 排序（order by id asc limit 1）,优化器最终选择了 PRIMARY 也就是全表扫描来执行，也就是说这个选择会无视之前的基于索引成本的选择，为什么会有这样的一个选项呢，主要原因如下：

```
The short explanation is that the optimizer thinks — or should I say hopes — that scanning the whole table (which is already sorted by the id field) will find the limited rows quick enough, and that this will avoid a sort operation. So by trying to avoid a sort, the optimizer ends-up losing time scanning the table.
```

从这段解释可以看出主要原因是由于我们使用了 order by id asc 这种基于 id 的排序写法，优化器认为排序是个昂贵的操作，所以为了避免排序，并且它**认为** limit n 的 n 如果很小的话即使使用全表扫描也能很快执行完，所以它选择了全表扫描，也就避免了 id 的排序（全表扫描其实就是基于 id 主键的聚簇索引的扫描，本身就是基于 id 排好序的）

然而实际上这里优化器误判了！实际上，选择 idx_uid_stat 索引，执行会快得多（只要 28 ms）！网上有不少人反馈这个问题，而且出现这个问题基本只与 SQL 中出现 `order by id asc limit n`这种写法有关，**如果 n 比较小很大概率会走全表扫描，如果 n 比较大则会选择正确的索引。**

这个 bug 最早追溯到 2014 年，不少人都呼吁官方及时修正这个bug、这其实也不算个bug，可以算是优化器会稳定出现的一种误判。

不过可能是实现比较困难，直到 MySQL 5.7，8.0 都还没解决。所以在官方修复前我们要尽量避免这种写法，如果一定要用这种写法，怎么办呢，主要有两种方案，看下节。

### 4、解决方案

1、使用 force index 来强制使用指定的索引，如下：

```sql
select * from order_info force index(idx_uid_stat) where uid = 5837661 order by id asc limit 1
```

这种写法虽然可以，但不够优雅，如果这个索引被废弃了咋办？于是有了第二种比较优雅的方案

2、使用 order by (id+0) 方案，如下

```sql
select * from order_info where uid = 5837661 order by (id+0) asc limit 1
```

这种方案也可以让优化器选择正确的索引，更推荐！为什么这个 trick 可以呢，因为此 SQL 虽然是按 id 排序的，但在 id 上作了加法这样耗时的操作(虽然只是加个无用的 0，但足以骗过优化器)，优化器认为此时基于全表扫描会更耗性能，于是会选择基于成本大小的方式来选择索引。



### 5、进一步理解索引

虽然看explain的结果上看，type=index，但是其实这不算使用到了索引！！或者说，其实并没有节省时间，**因为索引的优点比如排序，会加速查找，这些性能并没有被使用到！**所以虽然是用到了主键索引，但其实为了找`uid = 5837661`的那条记录，需要遍历所有的记录。（又因为是innodb引擎，聚簇索引，所以相当于遍历主键索引了）。

所以这个查询不能算作是走索引了！！

只有走到了：

```
KEY `idx_uid_stat` (`uid`,`order_status`),
```

这个索引上，因为满足最左前缀匹配，所以会很快的找到`uid = 5837661`这条记录，此时索引才起到了加速查询的效果！！
