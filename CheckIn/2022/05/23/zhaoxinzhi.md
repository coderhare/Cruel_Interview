### 一、写在前面

今天整理下关于Go语言type关键字（类型别名）的相关介绍和用法。

> 参考链接：http://c.biancheng.net/view/25.html

## 二、Golang

### 1、type

众所周知，type可以用来定义类型（定义结构体或者接口类型）

基础语法就不整理了。

### 2、定义类型别名

类型别名是 Go 1.9 版本添加的新功能，主要用于解决代码升级、迁移中存在的类型兼容性问题。在 C/C++ 语言中，代码重构升级可以使用宏快速定义一段新的代码，Go语言中没有选择加入宏，而是解决了重构中最麻烦的类型名变更问题。

（所以最终极的代码应该是，业务代码中全部都是别名类型，包括函数参数和返回值都是别名类型，而不使用原生类型，这样在架构升级或者底层数据结构转换的时候，业务层就可以屏蔽底层实现了，即不需要懂业务代码。）

在 Go 1.9 版本之前定义内建类型的代码是这样写的：

```
type byte uint8type rune int32
```

而在 Go 1.9 版本之后变为：

```
type byte = uint8type rune = int32
```

这个修改就是配合类型别名而进行的修改。



### 3、类型别名与类型定义

定义类型别名的写法为：

`type TypeAlias = Type`

类型别名规定：TypeAlias 只是 Type 的别名，本质上 TypeAlias 与 Type 是同一个类型.

类型别名与类型定义表面上看只有一个等号的差异，那么它们之间实际的区别有哪些呢？下面通过一段代码来理解。

```go
package main
import (
    "fmt"
)
// 将NewInt定义为int类型
type NewInt int
// 将int取一个别名叫IntAlias
type IntAlias = int
func main() {
    // 将a声明为NewInt类型
    var a NewInt
    // 查看a的类型名
    fmt.Printf("a type: %T\n", a)
    // 将a2声明为IntAlias类型
    var a2 IntAlias
    // 查看a2的类型名
    fmt.Printf("a2 type: %T\n", a2)
}
```

代码运行结果：

```go
a type: main.NewInt
a2 type: int
```


结果显示 a 的类型是 main.NewInt，表示 main 包下定义的 NewInt 类型，a2 类型是 int，IntAlias 类型只会在代码中存在，编译完成时，不会有 IntAlias 类型。



### 4、本地类型

在一个包下定义的类型，属于本地类型。如果要在其他包使用，那么这个类型就不是本地类型。

### 5、非本地类型不能定义方法

能够随意地为各种类型起名字，是否意味着可以在自己包里为这些类型任意添加方法呢？参见下面的代码演示：

```go
package main
import (
    "time"
)
// 定义time.Duration的别名为MyDuration
type MyDuration = time.Duration
// 为MyDuration添加一个函数
func (m MyDuration) EasySet(a string) {
}
func main() {
}
```

代码说明如下：

- 第 8 行，为 time.Duration 设定一个类型别名叫 MyDuration。
- 第 11 行，为这个别名添加一个方法。

编译上面代码报错，信息如下：

```go
cannot define new methods on non-local type time.Duration
```

编译器提示：不能在一个非本地的类型 time.Duration 上定义新方法，非本地类型指的就是 time.Duration 不是在 main 包中定义的，而是在 time 包中定义的，与 main 包不在同一个包中，因此不能为不在一个包中的类型定义方法。

解决这个问题有下面两种方法：

- 将第 8 行修改为 type MyDuration time.Duration，也就是将 MyDuration 从别名改为类型；
- 将 MyDuration 的别名定义（~~和具体实现？~~）放在 time 包中。



### 6、证明别名和原类型确实是一样的

```go
// 定义商标结构
type Brand struct {
}

// 为商标结构添加Show()方法
func (t Brand) Show() {
}

// 为Brand定义一个别名FakeBrand
type FakeBrand = Brand

// 定义车辆结构
type Vehicle struct {
   // 嵌入两个结构，直接不起变量名，不报错
   FakeBrand
   Brand
}

func main() {
    // 声明变量a为车辆类型
    var a Vehicle
   
    // 指定调用FakeBrand的Show
    a.FakeBrand.Show()   			//如果是a.Show()会报错：`ambiguous selector a.Show`
    // 取a的类型反射对象
    ta := reflect.TypeOf(a)
    // 遍历a的所有成员
    for i := 0; i < ta.NumField(); i++ {
        // a的成员信息
        f := ta.Field(i)
        // 打印成员的字段名和类型
        fmt.Printf("FieldName: %v, FieldType: %v\n", f.Name, f.Type.
            Name())
    }
}
```

代码输出如下：

```go
FieldName: FakeBrand, FieldType: Brand
FieldName: Brand, FieldType: Brand
```

如果是a.Show()会报错：`ambiguous selector a.Show`

在调用 Show() 方法时，因为两个类型都有 Show() 方法，会发生歧义，证明 FakeBrand 的本质确实是 Brand 类型。
