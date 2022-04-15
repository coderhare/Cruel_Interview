## 一、写在前面

总结一下工作中常用的sql函数和操作。

## 二、sql



#### 1、获取一个date类型的 年月日时分秒

```sql
select day('2022-04-14 09:45:37')

select day(getdate())  -- 执行这一句的前提必须有getdate权限
```



#### 2、获取当前时间

```sql
SELECT NOW()


```

#### 3、日期转为unix时间戳

```sql
SELECT UNIX_TIMESTAMP(NOW());  

SELECT FROM_UNIXTIME(1649929518);

组合使用：
SELECT FROM_UNIXTIME(UNIX_TIMESTAMP('2022-04-14 09:45:37'));
```



#### 4、对日期进行自定义的格式化

```sql
SELECT DATE_FORMAT('2022-04-14 09:45:37','%Y%m%d') -- 这样是只提取出date类型的年月日，忽略时分秒
-- 注意YMD的大小写，输出的是不同的形式，比如如果是%M的话输出的是英文的月份
其他：
select date_format(now(),’%Y-%m-%d %H:%i:%s’)
对时间进行加减运算
加运算 1
负运算 -1
SELECT date_format(date_add(CURRENT_TIMESTAMP(),interval 1 YEAR),’%Y-%m-%d %H:%i:%s’) a,
date_format(date_add(CURRENT_TIMESTAMP(),interval 1 MONTH),’%Y-%m-%d %H:%i:%s’) b,
date_format(date_add(CURRENT_TIMESTAMP(),interval 1 DAY),’%Y-%m-%d %H:%i:%s’) c,
date_format(date_add(CURRENT_TIMESTAMP(),interval 1 HOUR),’%Y-%m-%d %H:%i:%s’) d,
date_format(date_add(CURRENT_TIMESTAMP(),interval 1 MINUTE),’%Y-%m-%d %H:%i:%s’) e,
date_format(date_add(CURRENT_TIMESTAMP(),interval 1 SECOND),’%Y-%m-%d %H:%i:%s’) f,
date_format(date_add(CURRENT_TIMESTAMP(),interval 1 WEEK),’%Y-%m-%d %H:%i:%s’) g

-- 原文链接：https://blog.csdn.net/yy1209357299/article/details/109488661
```

#### 5、常用常量

| datepart | 缩写     |
| -------- | -------- |
| 年       | yy, yyyy |
| 季度     | qq, q    |
| 月       | mm, m    |
| 年中的日 | dy, y    |
| 日       | dd, d    |
| 周       | wk, ww   |
| 星期     | dw, w    |
| 小时     | hh       |
| 分钟     | mi, n    |
| 秒       | ss, s    |
| 毫秒     | ms       |
| 微妙     | mcs      |
| 纳秒     | ns       |





#### 6、验证子查询之间的隔离性

注意内查询和外查询是隔离的！！包括表名和字段名！！

```sql
SELECT `day`, tenantCode, 
FROM_UNIXTIME(AVG(UNIX_TIMESTAMP(execute_time))) avg, 
country_code countryCode		-- 注意这里不加result.  即不加内层查询的表的别名
from (
SELECT DATE_FORMAT(result.execute_time,'%Y%m%d') `day`, 
result.tenant_id tenantCode, result.execute_time,
task.country_code
from kunlun_result result, task task
WHERE result.task_id=task.id 
) a -- 这里不加a会报错，编译器会优化，把内层查询的结果表当做a了
GROUP BY `day`, tenantCode

或者这样写：
SELECT `day`, tenantCode, 
FROM_UNIXTIME(AVG(UNIX_TIMESTAMP(`ee`))) avg, -- 那这里注意，别名必须要加``飘号才可以！！
country_code countryCode		
from (
SELECT DATE_FORMAT(result.execute_time,'%Y%m%d') `day`, 
result.tenant_id tenantCode, result.execute_time ee,
task.country_code
from kunlun_result result, task task
WHERE result.task_id=task.id 
) a
GROUP BY `day`, tenantCode
```



#### 7、想得到一个sql语句产生的结果集的条数

```sql
-- 只需要把原sql放到子查询中即可。
SELECT COUNT(*) from (原sql) a -- 记得要给这个子查询得到的表一个别名 比如这里的a

例子：
SELECT COUNT(*) from (
SELECT DATE_FORMAT(result.execute_time,'%Y%m%d') `day`, result.tenant_id tenantCode, FROM_UNIXTIME(AVG(UNIX_TIMESTAMP(result.execute_time))) avg, task.country_code countryCode
from kunlun_result result, task task
WHERE result.task_id=task.id GROUP BY `day`, tenantCode, task.country_code
) a
```



#### 8、比较两个sql查询语句产生的结果集是否相同

比如我在写了sql语句，一周之后我想优化一下这个语句，那我怎么知道我新写的sql语句是正确的呢？

最简便的方法是和原来的sql语句对拍下。所以可能会有这么一个需求。

```sql
(A minus B) union (B minus A)
```

https://wenku.baidu.com/view/7c72d353757f5acfa1c7aa00b52acfc789eb9fd3.html



## 常用套路

1、group by A 通常用来确保A字段的唯一性

2、不管是hive还是mysql，可以通过建视图的方式来避免多层嵌套查询。

3、ifelse来新增字段

```sql
CASE WHEN id > 5 THEN 1 ELSE 0 END AS new_field_name
```





## 待测命令

from_unixtime(unix_timestamp(cast(day as string),'yyyyMMdd'),'yyyy-MM-dd') as `日期`,



convert函数



sql中怎样知道Select查询出的结果一共有多少行



## 常见报错

1、

```sql
1064 - You have an error in your SQL syntax; check the manual that corresponds to your MySQL
可能原因：
1、字段名称与表名称不能使用单引号，要使用飘号
2、该加逗号的地方没加，不该加的地方加了。
3、select xxxfrom table 这种，from前面忘加空格了

```

2、

```sql
1582 - Incorrect parameter count in the call to native function '某函数'
报错原因：
1、因为版本不同，导致函数的参数不一致。解决方法查官方文档看对应版本是否对参数个数有做删减，改正就好。
```

