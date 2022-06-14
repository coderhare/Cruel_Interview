## 一、写在前面

sql常见优化

## 二、sql

1、用group by 代替select distinct

2、多用in 而不是 not in

```sql
一种替代方式
用left outer join 再 filter by x column is null
另一种替代方式：
用exist语句
```

 

3、万不得已再order by

​	尽量最后一步操作了再order by

4、用temp table临时表，不要nested query子查询

```sql
临时表样例：
create table #tempPersonTable (
	personid int primary key identity(1,1),
	lastname varchar(255),
	firstname varchar(255)
)

select * from #tempPersonTable
```

5、学会巧用partition by
