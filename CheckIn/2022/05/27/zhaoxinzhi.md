## 一、写在前面

关于go,接口和struct比较,的坑



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

1、type 和 data 都相等（**注意两个相同的指针类型，也可能是不相等的！！**）

```go
type Profile struct {
    Name string
}

type ProfileInt interface {}

func main()  {
    var p1, p2 ProfileInt = Profile{"iswbm"}, Profile{"iswbm"}
    var p3, p4 ProfileInt = &Profile{"iswbm"}, &Profile{"iswbm"}

    fmt.Printf("p1 --> type: %T, data: %v \n", p1, p1)
    fmt.Printf("p2 --> type: %T, data: %v \n", p2, p2)
    fmt.Println(p1 == p2)  // true
//而 p3 和 p4 虽然类型都是 *Profile，但由于 data 存储的是结构体的地址，而两个地址和不相同，因此 p3 与 p4 不相等
    fmt.Printf("p3 --> type: %T, data: %p \n", p3, p3)
    fmt.Printf("p4 --> type: %T, data: %p \n", p4, p4)
    fmt.Println(p3 == p4)  // false
}
/*
输出：
p1 --> type: main.Profile, data: {iswbm} 
p2 --> type: main.Profile, data: {iswbm} 
true
p3 --> type: *main.Profile, data: 0xc00008e200 
p4 --> type: *main.Profile, data: 0xc00008e210 
false
*/
```



2、两个 interface 都是 nil

```go
func main() {
    var p1, p2 interface{}
    fmt.Println(p1 == p2)  // true
    fmt.Println(p1 == nil) // true
}
```



### 5、interface 与 非 interface 比较

当 interface 与非 interface 比较时，会将 非interface 转换成 interface（即转化成type和data） ，然后再按照 **两个 interface 比较** 的规则进行比较。

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

### 6、判断接口为nil的坑点

当把某一个类型的nil赋值给interface时，这个interface其实不是nil，看下面这个例子：

```go
package main

import (
    "fmt"
)

func main()  {
    var a *string = nil
    var b interface{} = a

    fmt.Println(b==nil) // false
}
```

解释：当你使用 `b==nil` 进行判断时，其实右边的 nil 并非单纯的是我们所理解的值为nil，而正确的理解应该是 类型为 nil 且 值也为 nil。即Go 会先将 nil 转换为interface  `(type=nil, data=nil)` ，这与 b `(type=*string, data=nil)` 虽然 data 是一样的，但 type 不相等，因此他们并不相等。

### 7、如何解？

即有没有办法判断一个 interface{} 是不是 nil 呢？

有，借助反射

```go
func IsNil(i interface{}) bool {
    vi := reflect.ValueOf(i)
    if vi.Kind() == reflect.Ptr {
        return vi.IsNil()
    }
    return false
}

func main() {
    var a *string = nil
    var b interface{} = a
    fmt.Println(IsNil(b))
}
```

