## 一、写在前面

今天总结下shell常见的一些问题。

源自：https://mp.weixin.qq.com/s/SB1_wqkFIQz-4iEL_LZnow

## 二、Shell

### 1、关于echo

`echo`将`argument`送到`标准输出(stdout)`，通常显示在屏幕

- stdin 标准输入
- stdout 标准输出
- stderr 标准错误输出

### 2、""(双引号)与''(单引号)有什么区别?

- hard quote:`''`(单引号),关闭所有引用
- soft quote:`""`(双引号),保留`$`引用

### 3、定义变量与export

变量定义：`name=value`，等号左右两边不能使用分隔符。
变量替换：`echo ${name}`
export变量：`export name=value`，使变量成为环境变量

```shell
这些都可以在bash里面输入
# 本地变量
A=B
# 取消变量
unset A
# 环境变量
export A=B
```

### 4、exec跟source的区别

环境变量只能从父进程到子进程单向传递。换句话说：在子进程中环境如何变更，均不会影响父进程的环境。
当我们执行一个shell script时，其实是先产生一个sub-shell的子进程， 然后sub-shell再去产生命令行的子进程。

三种执行方式：

```shell
总结来说：
# 创建子shell执行sh脚本
./xx.sh 或 sh xx.sh
# 当前shell执行sh脚本
source xx.sh 或 . xx.sh
# 当前shell执行sh脚本，并且执行后退出
exec xx.sh

详细来说：
$ sh script.sh
--使用该语句执行脚本时，当前shell是父进程，生成一个子shell进程，在子shell中执行脚本。脚本执行完毕，退出子shell，回到当前shell。
$ ./script.sh与$ sh script.sh等效。

$ echo $$  
--会输出当前shell的进程号，可以用来测试用。

$ source script.sh
--使用该语句执行脚本时，在当前上下文中执行脚本，不会生成新的进程。脚本执行完毕，回到当前shell。
source方式也叫点命令，$ . script.sh与$ source script.sh等效。
注意在点命令中，.与script.sh之间有一个空格。

$ exec script.sh
使用exec command方式，会用command进程替换当前shell进程，并且保持PID不变。执行完毕，直接退出，不回到之前的shell环境。即：替代当前shell，执行脚本。
例子：
riversec@rcs:~$ exec echo $$
2364
Connection to 192.168.2.123 closed.
```



#### 使用sh和source方式对上下文的影响

在`sh`和`source`方式下，脚本执行完毕，都会回到之前的shell中。但是两种方式对上下文的影响不同呢。

此例中，`jump.sh`脚本执行如下操作：1）跳到`/`，2）打印当前工作目录，3）打印`Hello`。

```shell
$ vi jump.sh
#!/bin/sh
cd /
pwd
echo Hello
```

通过`sh`执行脚本时，修改的上下文不会影响当前shell。`jump.sh`退出以后，工作目录保持不变。

```shell
$ pwd
/home/riversec
$ ./jump.sh 
/
Hello
$ pwd
/home/riversec
```

通过`source`执行脚本时，修改的上下文会影响当前shell。`jump.sh`退出以后，当前工作目录变成了`/`。

```shell
$ pwd
/home/riversec
$ source jump.sh 
/
Hello
$ pwd
/
```

#### 所以注意事项

所以如果要启动一个项目，或者给他启动一些配置的时候，最好是用sh而不是source，因为可能会改变当前shell的一些上下文。

### 5、( ) 与 { }的区别？

`( )` 将`command group`置于`sub-shell`执行
`{ }` 则是在同一个`shell`内完成

### 6、@与* 区别在哪？

- `"$@"` 则可得到 "p1" "p2 p3" "p4" 这三个不同的词段
- `"$*"` 则可得到 "p1 p2 p3 p4" 这一整串单一的词段

### 7、> 与 <的区别？

- 0: Standard Input (STDIN)
- 1: Standard Output (STDOUT)
- 2: Standard Error Output (STDERR)

> 我们可用 < 来改变读进的数据信道(stdin)，使之从指定的档案读进。
> 我们可用`>` 来改变送出的数据信道(stdout, stderr)，使之输出到指定的档案。

```
ls my.file no.such.file 1> file.out 2>file.err
# 2>&1 就是将stderr并进stdout做输出
ls my.file no.such.file 1> file.out 2>&1
# /dev/null 空
ls my.file no.such.file >/dev/null 2>&1

cat < file > file
# 在 IO Redirection 中，stdout 与 stderr 的管道会先准备好，才会从 stdin 读进资料。 
# 也就是说，在上例中，> file 会先将 file 清空，然后才读进 < file ， 
# 但这时候档案已经被清空了，因此就变成读不进任何数据了
```

