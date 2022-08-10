## 一、写在前面

关于mysql如何查看版本的四种方法

> 参考链接：https://www.jb51.net/article/36370.htm

## 二、mysql

### 1、查看mysql版本

方法1：从终端输入`mysql -u root -p`登陆

方法2：在mysql中输入status

方法3：使用mysql命令`select version();`查看

方法4：终端输入 `mysql --help | grep Distrib`

### 2、下面是实战：

方法1：

```Java
(base) C02GD5Q8MD6R:~ bytedance$ mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 26
Server version: 5.7.23 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.


mysql>
```

方法2：在mysql中输入status

```Java
mysql> status
--------------
mysql  Ver 14.14 Distrib 5.7.23, for macos10.13 (x86_64) using  EditLine wrapper

Connection id:    26
Current database:  
Current user:    root@localhost
SSL:      Not in use
Current pager:    stdout
Using outfile:    ''
Using delimiter:  ;
Server version:    5.7.23 MySQL Community Server (GPL)
Protocol version:  10
Connection:    Localhost via UNIX socket
Server characterset:  latin1
Db     characterset:  latin1
Client characterset:  utf8
Conn.  characterset:  utf8
UNIX socket:    /tmp/mysql.sock
Uptime:      34 days 8 hours 13 min 38 sec

Threads: 2  Questions: 786  Slow queries: 0  Opens: 152  Flush tables: 1  Open tables: 140  Queries per second avg: 0.000
--------------
```

方法3：使用mysql命令查看

使用mysql命令查看

```Java
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.23    |
+-----------+
1 row in set (0.00 sec)
```

方法4：终端输入 `mysql --help | grep Distrib`

```Java
(base) C02GD5Q8MD6R:~ bytedance$ mysql --help | grep Distrib
mysql  Ver 14.14 Distrib 5.7.23, for macos10.13 (x86_64) using  EditLine wrapper
```
