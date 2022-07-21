## 一、写在前面

关于go的WaitGroup的用法。

> 源码阅读：https://blog.csdn.net/Guzarish/article/details/118626496

## 二、GoLang

### 1、WaitGroup

WaitGroup在go语言中，用于线程同步，指等待一组，等待一个系列执行完成后才会继续向下执行。

正常情况下，goroutine的结束过程是不可控制的，我们可以保证的只有main goroutine的终止。

这时候可以借助sync包的WaitGroup来判断goroutine是否完成。

### 2、使用场景

`WaitGroup`的使用场景是有限的。

`WaitGroup`在需要等待多个任务结束再返回的业务来说还是很有用的，但现实中用的更多的可能是，先等待一个协程组，若所有协程组都正确完成，则一直等到所有协程组结束；若其中有一个协程发生错误，则告诉协程组的其他协程，全部停止运行(本次任务失败)以免浪费系统资源。

该场景`WaitGroup`是无法实现的，那么该场景该如何实现呢，就需要用到通知机制，其实也可以用`channel`来实现，在此先不展开。

### 3、和闭包结合使用

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup

	f := func(x int) {
		defer wg.Done()
		time.Sleep(1)
		fmt.Println(x)
	}
	wg.Add(5)
	for i := 0; i < 5; i++ {
		go f(i)
	}
	fmt.Println("start")
	wg.Wait()
	fmt.Println("over")
}
/*
输出：
start
2   
4   
1   
0   
3   
over
*/
```

### 4、注意事项

通过`WaitGroup`提供的三个函数：`Add`,`Done`,`Wait`，可以轻松实现等待某个协程或协程组完成的同步操作。但在使用时要注意:

- `WaitGroup` 可以用于一个 `goroutine` 等待多个 `goroutine` 干活完成，也可以多个 `goroutine` 等待一个 `goroutine` 干活完成，是一个多对多的关系（多个等待一个的典型案例是 singleflight，==有空分析！==）
- `Add(n>0) `方法应该在启动 `goroutine` 之前调用，然后在 `goroutine` 内部调用 `Done` 方法
- `WaitGroup` 必须在 `Wait` 方法返回之后才能再次使用
- `WaitGroup`不可以复制！！只能在创建一个。
- `Done` 只是 `Add` 的简单封装，所以实际上是可以通过一次加一个比较大的值减少调用，或者达到快速唤醒的目的。
- 协程函数要使用指针类型的`*sync.WaitGroup`作为参数，不能使用值类型的`sync.WaitGroup`作为参数
- Add的数量和Done的调用数量必须相等，否则可能发生死锁。（但是上面代码如果是wg.Add(1)的话可以正确运行，但是只会输出0，不会输出1234了）
