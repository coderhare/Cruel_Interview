## 一、写在前面

Go语言结构体的初始化

> 参考链接：https://www.likecs.com/show-306136287.html

## 二、Golang

Go 通过类型别名（alias types）和结构体的形式支持用户自定义类型。
结构体是复合类型，当需要定义类型，它由一系列属性组成，每个属性都有自己的类型和值的时候，就应该使用结构体，它把数据聚集在一起。

**结构体也是值类型，因此可以通过 new 函数来创建**

组成结构体类型的那些数据成为**字段（fields）**。每个字段都有一个类型和一个名字；在一个结构体中，字段名字必须是唯一的。

### 1、结构体定义

结构体定义的一般方式如下：

```
type identifier struct {
    field type1
    field type2
}
```

type T struct {a, b int} 也是合法的语法，它更适用于简单的结构体
结构体里的字段都有 名字，像 field1、field2 等，如果字段在代码中从来也不会被用到，那么可以命名它为 _。
结构体类型和字段的命名遵循可见性规则，所以可能存在一个结构体类型的某些字段是导出的，而另一些没有导出。
结构体的字段可以是任何类型，甚至是结构体本身，也可以是函数或者接口。可以声明结构体类型的一个变量，然后像下面这样给它的字段赋值：

```
var s T
s.a = 5
s.b = 8
```

数组也可以看作是一种结构体类型，不过它使用下标而不是具名的字段



### 2、结构体的初始化

#### 方式一：通过 var 声明结构体

在 Go 语言中当一个变量被声明的时候，系统会自动初始化它的默认值，比如 int 被初始化为 0，指针为 nil。
var 声明同样也会为结构体类型的数据分配内存，所以我们才能像上一段代码中那样，在声明了 `var s T` 之后就能直接给他的字段进行赋值。

> 即结构体类型，不需要new和make，直接 `var s T` 就分配内存了

#### 方式二：使用 new

使用 new 函数给一个新的结构体变量分配内存，它返回指向已分配内存的指针：var t *T = new(T)。

```
type struct1 struct {
    i1 int
    f1 float32
    str string
}

func main() {
    ms := new(struct1)
    ms.i1 = 10
    ms.f1 = 15.5
    ms.str= "Chris"

    fmt.Printf("The int is: %d\n", ms.i1)
    fmt.Printf("The float is: %f\n", ms.f1)
    fmt.Printf("The string is: %s\n", ms.str)
    fmt.Println(ms)
}
```

与面向对象语言相同，使用点操作符可以给字段赋值：`structname.fieldname = value`。

同样的，使用点操作符可以获取结构体字段的值：`structname.fieldname`。

#### 方式三：使用字面量

```go
type Person struct {
    name string
    age int
    address string
}

func main() {
    var p1 Person
    p1 = Person{"lisi", 30, "shanghai"}   //方式A
    p2 := Person{address:"beijing", age:25, name:"wangwu"} //方式B
    p3 := Person{address:"NewYork"} //方式C
}
```

在（方式A）中，值必须以字段在结构体定义时的顺序给出。（方式B）是在值前面加上了字段名和冒号，这种方式下值的顺序不必一致，并且某些字段还可以被忽略掉，就想（方式C）那样。
除了上面这三种方式外，还有一种初始化结构体实体更简短和常用的方式，如下：

```
ms := &Person{"name", 20, "bj"}
ms2 := &Person{name:"zhangsan"}
```

`&Person{a, b, c}` 是一种简写，底层仍会调用 `new()`，这里值的顺序必须按照字段顺序来写，同样它也可以使用在值前面加上字段名和冒号的写法（见上文的方式B，C）。

表达式 `new(Type)` 和 `&Type{}` 是等价的。

### 3、几种初始化方式的区别

> 具体例子不粘贴过来了

到目前为止，我们已经了解了三种初始化结构体的方式：

```
//第一种，在Go语言中，可以直接以 var 的方式声明结构体即可完成实例化
var t T
t.a = 1
t.b = 2

//第二种，使用 new() 实例化
t := new(T)

//第三种，使用字面量初始化
t := T{a, b}
t := &T{} //等效于 new(T)
```

使用 `var t T` 会给 t 分配内存，并零值化内存，但是这个时候的 t 的类型是 T
使用 new 关键字时 `t := new(T)`，变量 t 则是一个指向 T 的指针
从内存布局上来看，我们就能看出这三种初始化方式的区别：
使用 var 声明：
![Go语言结构体的初始化](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvNjYyMjM2LzIwMTgxMi82NjIyMzYtMjAxODEyMDEyMjU5NDM5NzQtMjExNTM0OTk0MC5wbmc=)

使用 new 初始化：
![Go语言结构体的初始化](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvNjYyMjM2LzIwMTgxMi82NjIyMzYtMjAxODEyMDEyMjU5NTAyNzYtNTg4NDUwOTY0LnBuZw==)

使用结构体字面量初始化：
![Go语言结构体的初始化](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvNjYyMjM2LzIwMTgxMi82NjIyMzYtMjAxODEyMDEyMjU5NTQ5MzMtMTMwMDUwMTIwNC5wbmc=)

### 4、结构体的内存布局

Go 语言中，结构体和它所包含的数据在内存中是以连续块的形式存在的，即使结构体中嵌套有其他的结构体，这在性能上带来了很大的优势。不像 Java 中的引用类型，一个对象和它里面包含的对象可能会在不同的内存空间中，这点和 Go 语言中的指针很像。下面的例子清晰地说明了这些情况：

```
type Rect1 struct {Min, Max Point }
type Rect2 struct {Min, Max *Point }
```

![Go语言结构体的初始化](https://www.likecs.com/default/index/img?u=aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvNjYyMjM2LzIwMTgxMi82NjIyMzYtMjAxODEyMDEyMzAwMDU4MDYtMTc2NjM3ODI5OS5wbmc=)
