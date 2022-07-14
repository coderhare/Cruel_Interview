## 一、写在前面

关于GO1.13的error处理的新方法。

> 参考链接：[go1.13-errors](https://blog.golang.org/go1.13-errors)（官方链接）
>
> https://blog.csdn.net/yuanlaidewo000/article/details/104563680
>
> https://blog.csdn.net/kaolengmian7/article/details/123805932
>
> 推荐阅读：https://mp.weixin.qq.com/s/3gEf-vL_iAIaL_tWCddILQ



## 二、Golang

我们经常会在处理 error 的时候需要添加一些有用的上下文信息，之前常用的做法是使用 `github.com/pkg/errors` 这个第三方包里的 `errors.WithMessage `方法，或者是`errors.WithStack` 方法。如果想从结果 err 中得到原始 error 的话，就调用 err.Cause() 方法。

但是这样会有一个问题，就是如果 error 包裹的太深的话，就只能一层一层的拆开去检验。

不过不用担心，go 1.13 新出的 errors 包就解决了这个问题。它添加了 `errors.Is` 和 `errors.As` 两个方法来处理 error。

### 1、 `errors.Is` 和 `errors.As` 

> 参考链接：https://studygolang.com/articles/23454?fr=sidebar

 `errors.Is` ：严格判断相等,即两个error是否相等。（通过每一个字段来判断，比如msg信息等）

 `errors.As` ：是判断类型是否相同，并提取第一个符合目标类型的错误，用来统一处理某一类错误。

### 2、怎么处理error？

#### 1、优先处理error

当一个函数返回一个非空error时，应该优先处理error，忽略它的其他返回值

#### 2、只处理error一次

**低级别error：通过日志+返回默认值处理**

关于error，要么通过日志处理（比如打warn或者error的日志，然后返回(res,nil)，注意这里res返回值要小心处理），要么直接把err抛给上层，具体要看错误的级别了。如果错误级别很低，就warn就可以吗，如果稍微高一些，但是不影响整体链路，则可以一个warn或者error，并且返回的res是一个默认值。

**高级别error：直接panic**

在 Golang 中`panic`会导致程序直接退出，是一个致命的错误。
建议发生致命的程序错误时才使用 panic，例如索引越界、不可恢复的环境问题、栈溢出等等

#### 3、不要反复包装error

我们应该包装error，但只包装一次
上层业务代码建议Wrap error，但是底层基础库不建议。（底层基础库建议返回预定义错误！！）
如果底层基础库包装了一次，上层业务代码又包装了一次，就重复包装了 error，日志就会打重
比如我们常用的sql库会返回sql.ErrNoRows这种预定义错误，而不是给我们一个包装过的 error

#### 4、简化错误处理

Golang因为代码中无数的`if err != nil`被诟病，可以看看go源码中类似`bufio.Scan()`的逻辑来减少`if err != nil`这种代码。

其实就是整个上下文共用一个err（作为一个字段放到ctx里或者一个obj里）

> 源码这里有提到https://blog.csdn.net/kaolengmian7/article/details/123805932

### 3、Go中处理error的最佳实践

> https://studygolang.com/articles/23454?fr=sidebar

Go 最新版本 1.13 中新增了 errors 的一些特性，，本质是套娃！！！

> 套娃的方式1：主要是使用`fmt.Errorf`这个方法进行套娃！！

> 套娃的方式2：`github.com/pkg/errors` 这个包里的 `errors.WithMessage` 方法，或者是 `errors.WithStack` 方法。
> 							如果想从结果 err 中得到原始 error 的话，就调用 `err.Cause()` 方法。

下面主要展示套娃的方式1。（方式2用到在学吧）

#### 3.1 errors.Unwrap

正确示范

```go
err1 := errors.New("new error")
err2 := fmt.Errorf("err2: [%w]", err1)
err3 := fmt.Errorf("err3: [%w]", err2)
fmt.Println(err3) //err3: [err2: [new error]]
fmt.Println(errors.Unwrap(err3))//err2: [new error]
fmt.Println(errors.Unwrap(errors.Unwrap(err3)))//new error
```

`err2` 就是一个合法的**被包装的 error**，同样地，`err3` 也是一个**被包装的 error**，如此可以一直套下去。

注意这里只能有一个%w才算合法套娃err！！不然第二个打印出来就很怪了，而且没法Unwrap操作！！

错误示范的举例：

```go
err1 := errors.New("new error")
err11 := errors.New("new error11")
err2 := fmt.Errorf("err2: %w,[%w],[%w]", 123, err1, err11)

fmt.Println(errors.Unwrap(err3))//err2: %!w(int=123),[new error],[%!w(*errors.errorString=&{new error11})]
fmt.Println(errors.Unwrap(errors.Unwrap(err3)))//<nil>
```

可以发现，拆出err2来的时候虽然只识别第一个err类型数据，但至少报错信息还能看

最后一行中，拆出err1来的时候，就没法看了。直接nil，因为相当于父节点有两个，他也不知道选哪个。



#### 3.2 errors.Is

当多层调用返回的错误被一次次地包装起来，我们在调用链上游拿到的错误如何判断是否是底层的某个错误呢？
它递归调用 Unwrap 并判断每一层的 err 是否相等，如果有任何一层 err 和传入的目标错误相等，则返回 true。

```go
err1 := errors.New("new error")
err2 := fmt.Errorf("err2: [%w]", err1)
err3 := fmt.Errorf("err3: [%w]", err2)

fmt.Println(errors.Is(err3, err2))//true
fmt.Println(errors.Is(err3, err1))//true
```

#### 3.3 errors.As

这个和上面的 `errors.Is` 大体上是一样的，区别在于 `Is` 是严格判断相等，即两个 `error` 是否相等。
**而 `As` 则是判断类型是否相同，并提取第一个符合目标类型的错误，用来统一处理某一类错误。**

```go
type ErrorString struct {
    s string
}

func (e *ErrorString) Error() string {
    return e.s
}

var targetErr *ErrorString
err := fmt.Errorf("new error:[%w]", &ErrorString{s:"target err"})
fmt.Println(errors.As(err, &targetErr)) // true
```



### 4、项目中处理error的注意事项

#### 





### 5、坑！！易错

> 其实是在用errors.As()的时候，一个是指针类型，一个是结构体类型，导致的无法被识别为As。

踩到的坑，就是 errors.As 方法中的第二个参数，也就是 target。在函数声明中，target 是一个空接口，即可以传任意类型。但是根据 As 函数的定义，我们可以看出来，target 必须是一个 “指向一个实现了 error 接口” 的指针 (如果你使用的是 jetbrains 的 IDE 的话，它也会在编辑器中提示你的)。

正是这里，会导致一个小问题，就是如果有一个 Foo 类型结构体，实现了 Error() 方法，但是该方法的接收者是指针类型，即:

```go
type Foo struct{}

func (f *Foo) Error() string {
    return "this is an foo error"
}
```

这样其实 Foo 与 *Foo 都是 error 接口。所以在如果返回的 err 是 *Foo 类型，而你的代码是这样写的:

```go
func Bar() {
    err := someFuncReturnPointerFoo() // *Foo
    var tErr Foo
    fmt.Println(errors.As(err, &tErr))
    // Output: false
}
```

期望将 err 作为 Foo 类型来处理，可以编译通过，但无法得到想要的结果。
因为 tErr 是 Foo 类型，而 err 实际是 *Foo 类型。
所以我们需要将上边代码第三行，声明 tErr 的位置改为 *Foo 即可。

**其实，Foo没有实现err接口！！！ 只是编译器会帮你把Foo.Error() 转换成 (&Foo).Error()**



### 6、为什么`errors.New()`返回的是`errorString对象`的指针？

其原因是防止字符串产生碰撞，如果发生碰撞，两个 error 对象会相等。

```go
func TestErrString(t *testing.T) {
    var error1 = errors.New("error")
    var error2 = errors.New("error")
    
    if error1 != error2 {
        log.Println("error1 != error2")
    }
}
---------------------代码运行结果--------------------------
=== RUN   TestXXXX
2022/03/25 22:05:40 error1 != error2
```
