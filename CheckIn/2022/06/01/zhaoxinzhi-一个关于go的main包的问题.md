## 一、写在前面

go run main.go undefined这个报错。

> 参考链接：https://blog.csdn.net/weixin_33910759/article/details/88737395



## 二、goalng

错误的触发方式如下：

```bash
错误：
>>> go run main.go 
# command-line-arguments
./main.go:33:28: undefined: ThumbServiceImpl

正确：
>>> go run main.go handler.go 
是没问题的
```

其实是在main包下有多个go文件，此时需要都加在参数里才可以。

主要还是main包的加载的特性决定的。

### 1、关于main包

golang main包推荐只有一个main.go文件，这样大家就能按照习惯的方式，`go run main.go 或 go build main.go`来运行编译项目。

如果main包下有多个go文件，应该使用`go run a.go b.go c.go 或 go run *.go`来运行，编译同理。

因为mian包里，使用go run main.go，编译器只会加载main.go这个文件，不会加载main包里的其他文件，只有非main包里的文件才会通过依赖去自动加载。所以你需要输入多个文件作为参数。

### 2、golang推荐项目结构

```go
.
├── .gitignore
├── README.md
├── main.go
└── src
    ├── pkg1
    │   └── a.go
    ├── pkg2
    │   └── b.go
    └── pkg3
        └── c.go
```

如果需要编译为多个文件，可以加入cmd文件夹：

```go
.
├── .gitignore
├── README.md
├── cmd
│   ├── cmd1
│   │   └── main.go
│   └── cmd2
│       └── main.go
└── src
    ├── pkg1
    │   └── a.go
    ├── pkg2
    │   └── b.go
    └── pkg3
        └── c.go
```



但这应该是GOPATH模式的写法，GOMODULE的写法应该略有区别，但是可以作为参考。

