## 一、写在前面

关于go



## 二、golang

### 1、

首先提问：

如何判断一个 interface{} 的值是否为 nil ？

直接 `v == nil` 进行判断

### 2、猜测输出

这段代码输出是啥？

```go
package main

import (
    "fmt"
)

func main()  {
    var a *string = nil
    var b interface{} = a

    fmt.Println(b==nil) 
}
```

输出的结果的是 `false`



### 3、interface的内部实现

interface 的内部实现包含了两个字段，一个是 type，一个是 data。比如这个变量：var age interface{} = 25

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z9cbLZEggxIxicrNUWmJhvAKNjGmeaMQBVLljthVxY3sm9IAv5xdPugOwTdh1RxTb7pvZDcyR94Nqiclhia7BcX9w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### 4、interface 与 interface 比较

经过验证，只有下面两种情况，两个 interface 才会相等。

1、type 和 data 都相等

2、两个 interface 都是 nil



### 5、interface 与 非 interface 比较

当 interface 与非 interface 比较时，会将 非interface 转换成 interface ，然后再按照 **两个 interface 比较** 的规则进行比较。

```go
package main

import (
    "fmt"
    "reflect"
)

func main()  {
    var a string = "iswbm"
    var b interface{} = "iswbm"
    fmt.Println(a==b) // true
}
```

