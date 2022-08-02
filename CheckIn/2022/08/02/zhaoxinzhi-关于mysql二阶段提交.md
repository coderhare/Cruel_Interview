## 一、写在前面

关于mysql的二阶段提交。分布式事务

> 参考链接：https://blog.51cto.com/u_10679074/4926342

## 二、分布式事务

### 1、场景

如果我们现在有两个mysql实例（连接），在我们要尽量简单地完成分布式事务，怎么处理？

比如我们现在有两个数据库，mysql3306和mysql3307

在mysql3306中，我们有一个user表。

```sql
create table user (
    id int,
    name varchar(10),
    score int
);
insert into user values(1, "foo", 10)
```

在mysql3307中，我们有一个wallet表。

```sql
create table wallet (
    id int,
    money float 
);
insert into wallet values(1, 10.1)
```

id为1的用户初始分数（score）为10，而它的钱，在wallet中初始钱（money）为10.1。

现在假设我们有一个操作，需要对这个用户进行操作：每次操作增加分数2，并且增加钱数1.2。

这个操作需要很强的一致性。这需要分布式事务来解决。



### 2、二阶段提交

我们可以使用2PC的方法进行保证分布式事务的最终一致性。

![使用golang理解mysql的两阶段提交_github](https://s2.51cto.com/images/blog/202112/31101518_61ce67b6c64ce58718.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

2PC的概念如图所示，引入一个资源协调者的概念，由这个资源协调者进行事务协调。

第一阶段，由这个资源协调者对每个mysql实例调用prepare命令，让所有的mysql实例准备好，如果其中由mysql实例没有准备好，协调者就让所有实例调用rollback命令进行回滚。如果所有mysql都prepare完成，那么就进入第二阶段。

第二阶段，资源协调者让每个mysql实例都调用commit方法，进行提交。

mysql里面也提供了分布式事务的语句XA。



### 3、如果只用事务是什么效果

```go
package main

import (
     "database/sql"
     "fmt"

     _ "github.com/go-sql-driver/mysql"
     "github.com/pkg/errors"
)

func main() {
     var err error

     // db1的连接
     db1, err := sql.Open("mysql", "root:123456@tcp(127.0.0.1:3306)/hade1")
     if err != nil {
          panic(err.Error())
     }
     defer db1.Close()

     // db2的连接
     db2, err := sql.Open("mysql", "root:123456@tcp(127.0.0.1:3307)/hade2")
     if err != nil {
          panic(err.Error())
     }
     defer db2.Close()

     // 开始前显示
     var score int
     db1.QueryRow("select score from user where id = 1").Scan(&score)
     fmt.Println("user1 score:", score)
     var money float64
     db2.QueryRow("select money from wallet where id = 1").Scan(&money)
     fmt.Println("wallet1 money:", money)

     tx1, err := db1.Begin()
     if err != nil {
          panic(errors.WithStack(err))
     }
     tx2, err := db2.Begin()
     if err != nil {
          panic(errors.WithStack(err))
     }

     defer func() {
          if err := recover(); err != nil {
               fmt.Printf("%+v\n", err)
               fmt.Println("=== call rollback ====")
               tx1.Rollback()
               tx2.Rollback()
          }

          db1.QueryRow("select score from user where id = 1").Scan(&score)
          fmt.Println("user1 score:", score)
          db2.QueryRow("select money from wallet where id = 1").Scan(&money)
          fmt.Println("wallet1 money:", money)
     }()

     // DML操作
     if _, err = tx1.Exec("update user set score=score+2 where id =1"); err != nil {
          panic(errors.WithStack(err))
     }
     if _, err = tx2.Exec("update wallet set money=money+1.2 where id=1"); err != nil {
          panic(errors.WithStack(err))
     }

    // panic(errors.New("commit before error"))

     // commit
     fmt.Println("=== call commit ====")
     err = tx1.Commit()
     if err != nil {
          panic(errors.WithStack(err))
     }

    // panic(errors.New("commit db2 before error"))  如果在这里panic的话就不一致了。

     err = tx2.Commit()
     if err != nil {
          panic(errors.WithStack(err))
     }

     db1.QueryRow("select score from user where id = 1").Scan(&score)
     fmt.Println("user1 score:", score)
     db2.QueryRow("select money from wallet where id = 1").Scan(&money)
     fmt.Println("wallet1 money:", money)
}
```

执行发现，如果`commit db2 before error`这里panic了，是无法保证一致性的。

![使用golang理解mysql的两阶段提交_Java_03](https://s2.51cto.com/images/blog/202112/31101519_61ce67b788d8f62216.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)





### 4、用XA实现分布式事务

注意defer那里，是不是不能进行rollback？？？待确定

```go
package main

import (
     "database/sql"
     "fmt"
     "log"
     "strconv"
     "time"

     _ "github.com/go-sql-driver/mysql"
     "github.com/pkg/errors"
)

func main() {
     var err error

     // db1的连接
     db1, err := sql.Open("mysql", "root:123456@tcp(127.0.0.1:3306)/hade1")
     if err != nil {
          panic(err.Error())
     }
     defer db1.Close()

     // db2的连接
     db2, err := sql.Open("mysql", "root:123456@tcp(127.0.0.1:3307)/hade2")
     if err != nil {
          panic(err.Error())
     }
     defer db2.Close()

     // 开始前显示
     var score int
     db1.QueryRow("select score from user where id = 1").Scan(&score)
     fmt.Println("user1 score:", score)
     var money float64
     db2.QueryRow("select money from wallet where id = 1").Scan(&money)
     fmt.Println("wallet1 money:", money)

     // 生成xid
     xid := strconv.FormatInt(time.Now().Unix(), 10)
     fmt.Println("=== xid:" + xid + " ====")
     defer func() {
          if err := recover(); err != nil {
               fmt.Printf("%+v\n", err)
               fmt.Println("=== call rollback ====")
               db1.Exec(fmt.Sprintf("XA ROLLBACK '%s'", xid))
               db2.Exec(fmt.Sprintf("XA ROLLBACK '%s'", xid))
          }

          db1.QueryRow("select score from user where id = 1").Scan(&score)
          fmt.Println("user1 score:", score)
          db2.QueryRow("select money from wallet where id = 1").Scan(&money)
          fmt.Println("wallet1 money:", money)
     }()

     // XA 启动
     fmt.Println("=== call start ====")
     if _, err = db1.Exec(fmt.Sprintf("XA START '%s'", xid)); err != nil {
          panic(errors.WithStack(err))
     }
     if _, err = db2.Exec(fmt.Sprintf("XA START '%s'", xid)); err != nil {
          panic(errors.WithStack(err))
     }

     // DML操作
     if _, err = db1.Exec("update user set score=score+2 where id =1"); err != nil {
          panic(errors.WithStack(err))
     }
     if _, err = db2.Exec("update wallet set money=money+1.2 where id=1"); err != nil {
          panic(errors.WithStack(err))
     }

     // XA end
     fmt.Println("=== call end ====")
     if _, err = db1.Exec(fmt.Sprintf("XA END '%s'", xid)); err != nil {
          panic(errors.WithStack(err))
     }
     if _, err = db2.Exec(fmt.Sprintf("XA END '%s'", xid)); err != nil {
          panic(errors.WithStack(err))
     }

     // prepare
     fmt.Println("=== call prepare ====")
     if _, err = db1.Exec(fmt.Sprintf("XA PREPARE '%s'", xid)); err != nil {
          panic(errors.WithStack(err))
     }
     // panic(errors.New("db2 prepare error"))
     if _, err = db2.Exec(fmt.Sprintf("XA PREPARE '%s'", xid)); err != nil {
          panic(errors.WithStack(err))
     }

     // commit
     fmt.Println("=== call commit ====")
     if _, err = db1.Exec(fmt.Sprintf("XA COMMIT '%s'", xid)); err != nil {
          // TODO: 尝试重新提交COMMIT
          // TODO: 如果还失败，记录xid，进入数据恢复逻辑，等待数据库恢复重新提交
          log.Println("xid:" + xid)
     }
     // panic(errors.New("db2 commit error"))
     if _, err = db2.Exec(fmt.Sprintf("XA COMMIT '%s'", xid)); err != nil {
          log.Println("xid:" + xid)
     }

     db1.QueryRow("select score from user where id = 1").Scan(&score)
     fmt.Println("user1 score:", score)
     db2.QueryRow("select money from wallet where id = 1").Scan(&money)
     fmt.Println("wallet1 money:", money)
}
```



### 5、其他

mysql从5.7之后才真正实现了两阶段的xa。当然这个两阶段方式在真实的工程中的使用其实很少的，xa的第一定律是避免使用xa。工程中会有很多方式来避免这种分库的事务情况。
