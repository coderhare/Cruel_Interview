## 一、写在前面

go get 和 go get -u的区别

go get 和go install 的区别

> 参考链接：https://moelove.info/2020/12/19/Go-1.16-%E4%B8%AD%E5%85%B3%E4%BA%8E-go-get-%E5%92%8C-go-install-%E4%BD%A0%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E7%9A%84%E5%9C%B0%E6%96%B9/



## 二、go

### 1、go get 和 go get -u

一句话，都加-u就行了。

-u 是update 的首字母，更新的意思。

**这个包如果有，就更新为最新版本，如果没有就下载。**

> 例子：pkg/mod里有那个包的文件夹，但是里面是空的，如果不用-u，它也默认我已经安装过这个包了，即使这个包是空的，所以我必须用-u就安装才能安装上。

仔细来说：加上它可以利用网络来更新已有的代码包及其依赖包。如果已经下载过一个代码包，但是这个代码包又有更新了，那么这时候可以直接用 -u 标记来更新本地的对应的代码包。如果不加这个 -u 标记，执行 go get 一个已有的代码包，会发现命令什么都不执行。只有加了 -u 标记，命令会去执行 git pull 命令拉取最新的代码包的最新版本，下载并安装。



### 2、go get 和go install

一句话，如果要安装到`$GOPATH/bin`目录下（即安装二进制），则go install。如果只是在go.mod中更新依赖，则go get



在将来，`go install` 被设计为“用于构建和安装二进制文件”， `go get` 则被设计为 “用于编辑 go.mod 变更依赖”



```bash
mktemp是一个创建临时目录的命令：

(base) C02GD5Q8MD6R:aaaaaaaa bytedance$ pwd
/Users/bytedance/Desktop/zxzstudy/aaaaaaaa
(base) C02GD5Q8MD6R:aaaaaaaa bytedance$ cd $(mktemp -d)
(base) C02GD5Q8MD6R:tmp.qD1YQJV6 bytedance$ ls
(base) C02GD5Q8MD6R:tmp.qD1YQJV6 bytedance$ pwd
/var/folders/30/cmv9c_5j3mq_kthx63sb1t5c0000gn/T/tmp.qD1YQJV6
```

