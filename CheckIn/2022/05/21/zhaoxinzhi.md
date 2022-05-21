## 一、写在前面

关于golang的断言

>参考链接：https://blog.csdn.net/weixin_44014995/article/details/109531850



## 二、Golang

类型断言就是将接口类型的值(x)，转换成类型(T)。格式为：x.(T)；
类型断言的必要条件就x是接口类型，非接口类型的x不能做类型断言；
T可以是非接口类型，如果想断言合法，则T必须实现x的接口；
T也可以是接口，则x的动态类型也应该是接口T；
类型断言如果非法，运行时会导致错误，为了避免这种错误，应该总是使用下面的方式来进行类型断言

```go
package main

import (
	"fmt"
)

func main() {

	var x interface{}
	x =100
	value1,ok :=x.(int)
	if ok {
		fmt.Println(value1)
	}
	value2,ok :=x.(string)
	if ok {
		fmt.Println(value2)
	}
}

```

需要注意的如果不接收第二个参数也就是`ok`，这里失败的话则会直接`panic`这里还存在一种情况就是`x`为`nil`同样会`panic`



注意下接口的继承：（知识点写到代码里了）

```go
//定义一个类型AA
type AA struct {
	x string
	y int
}

//定义一堆接口，比如Ix,Iy,Ixy,Ixyz
type Ix interface {
	printX() string
}
type Iy interface {
	printY() string
}
type Ixy interface {
	Ix
	Iy
}

type Iz interface {
	printZ() string
}
type Ixyz interface {
	Ixy
	Iz
}

//让AA类型实现Ixy接口及其父接口（如何定义父接口？越少越简陋越父接口，比如interface{}这个空接口）
func (a AA) printX() string {
	fmt.Println("in printX: ", a.x)
	return "printX over"
}

func (a AA) printY() string {
	fmt.Println("in printY: ", a.y)
	return "printY over"
}


//a Iz 会报错 AA does not implement Iz (missing printZ method)
//a Ixyz 会报错 AA does not implement Ixyz (missing printZ method)
//这里函数传参是向上兼容的，可以interface{}也可以Ix也可以Iy也可以Ixy
func mytest(a Ix) {
	aa, ok := a.(Ixy) //这里既可以是a.(Ix)，也可以是a.(AA)
	//也就是说，虽然是个interface{}类型，但其实还是有真实类型的，然后通过断言成不同的类型，来使用他的一些接口。可以断言成接口或者类型，都可以。

	if ok {
		aa.printX()
		aa.printY()
	}
}

func TestName(t *testing.T) {
	var a AA
	a.x = "abc"
	a.y = 1
	mytest(a)
}
```



