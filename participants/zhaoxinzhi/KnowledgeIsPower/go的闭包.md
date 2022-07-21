## 一、写在前面

关于go的闭包，以及逃逸分析等一些概念。

## 二、GoLang

### 1、闭包例子

先来看一段例子

```go
func main() {
    f := test()
    fmt.Println(f())
    fmt.Println(f())
}
func test() func() int32 {
    var x int32
    return func() int32 {
        x ++
        return x*x
    }
}
输出：
1
4
```

虽然x是定义在test函数中的一个局部变量，但是由于闭包的关系（子函数可以使用父函数的值），导致x被分配到了堆上（逃逸了）。

### 2、
