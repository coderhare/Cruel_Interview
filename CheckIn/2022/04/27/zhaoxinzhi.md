## 一、写在前面

关于hivesql 的 VIEW EXPLODE。



## 二、HIVE

## 1、一些概念

⾏转列：将多个列中的数据在⼀列中输出
列转⾏：将某列⼀⾏中的数据拆分成多⾏

### 2、Explode炸裂函数

将hive某列⼀⾏中复杂的 array 或 map 结构拆分成多⾏（**只能输⼊array或map**）

2.1 语法

```sql
explode(col)
select explode(arraycol)as newcol from tablename;
```

 

explode()：函数中的参数传⼊的是arrary数据类型的列名。
newcol：是给转换成的列命名⼀个新的名字，⽤于代表转换之后的列名。
tablename：原表名



``` sql 
-- 注意是hive！！而不是sql！！sql没有array这个函数
select(array('1','2','3'))
select explode(map('A','1','B','2','C','3'))

不能直接这么用，会报错：
解析错误: org.apache.calcite.runtime.CalciteContextException: Sql 1: From line 1, column 8 to line 1, column 44: TABLE Function 'explode' should not use in select list directly, please use lateral view instead
```





### 3、posexplode()函数

对⼀列进⾏列转行可以使⽤ explode()函数，但是如果想实现对两列都进⾏多⾏转换，那么⽤explode()函数就不能实现了，可以⽤
posexplode()函数，因为该函数可以将index和数据都取出来，使⽤两次posexplode并令两次取到的index相等就⾏了。



### 4、UDTF函数

split, explode等函数叫UDTF函数。

### 5、Lateral View

Lateral View配合 split, explode 等UDTF函数⼀起使⽤，它能够将⼀列数据拆成多⾏数据，并且对拆分后结果进⾏聚合，即将多⾏结果组合成⼀个⽀持别名的虚拟表

lateral view在UDTF前使⽤，表⽰连接UDTF所分裂的字段。	

```sql
语法：
lateral view udtf(expression) tableAlias as columnAlias (,columnAlias)*

说明：
UDTF(expression)：使⽤的UDTF函数，例如explode()。
tableAlias：表⽰UDTF函数转换的虚拟表的名称。
columnAlias：表⽰虚拟表的虚拟字段名称，如果分裂之后有⼀个列，则写⼀个即可；如果分裂之后有多个列，按照列的顺序在括号中声明所有虚拟列名，以逗号隔开。
```

Lateral View主要解决在select使⽤UDTF做查询的过程中查询只能包含单个UDTF，不能包含其它字段以及多个UDTF的情况（不能添加额
外的select列的问题）。
Lateral View其实就是⽤来和想类似explode这种UDTF函数联⽤的，lateral view会将UDTF⽣成的结果放到⼀个虚拟表中，然后这个虚拟
表会和输⼊⾏进⾏join来达到连接UDTF外的select字段的⽬的。



注：
1）lateral view的位置是from后where条件前
2）⽣成的虚拟表的表名不可省略
3）from后可带多个lateral view
3）如果要拆分的字段有null值，需要使⽤lateral view outer 替代，避免数据缺失



### 6、lateral view explode()组合拳

例子1：

<img src="/Users/bytedance/Library/Application Support/typora-user-images/image-20220428164239817.png" alt="image-20220428164239817" style="zoom:50%;" />

```sql
select movie,category_name
from  movies 
lateral view explode(category) table_tmp as category_name; -- 不加as会咋样？
```

其实相当于 explode拆分成多行，假设是x，

然后外面套一层 `lateral view UDTF 临时表名 新列名`，

然后放在from后面，where前面。

-----



例子2：

![image-20220428164152909](/Users/bytedance/Library/Application Support/typora-user-images/image-20220428164152909.png)

在例子2中，先用split把一个string给转成array，然后才能用explode函数。



### 7、丢失数据问题

LATERAL VIEW EXPLODE 函数一行拆多行数据异常丢失

https://www.jianshu.com/p/8e68ded82622
