## 一、写在前面

GO的性能优化--sync.Pool

> 参考链接：https://blog.csdn.net/qq_35423190/article/details/119572015

## 二、性能优化



当遇到频繁进行一些结构体的申请，内存会频繁的进行释放和申请，这时可以尝试一下sync.pool进行优化。

sync.pool，需要初始化 Pool，唯一需要的就是设置好 New 函数。当调用 Get 方法时，如果池子里缓存了对象，就直接返回缓存的对象。如果没有存货，则调用 New 函数创建一个新的对象。

另外，我们发现 Get 方法取出来的对象和上次 Put 进去的对象实际上是同一个，Pool 没有做任何“清空”的处理。但我们不应当对此有任何假设，因为在实际的并发使用场景中，无法保证这种顺序，最好的做法是在 Put 前，将对象清空。



对不同的数据类型，以及go官方推荐的场景尝试了pool：

```go
package main
import (
  "fmt"
  "time"
  "sync"
  "math/rand"
  "strconv"
)
func mapTestNoPool()time.Duration{
        be:=time.Now()
        for i:=0;i<100000;i++{
                var aaa map[int]int=make(map[int]int)
                for j:=0;j<100;j++{
                        aaa[j]=j
                }
        }
        return time.Since(be)
}
func mapTestPool()time.Duration{
        be:=time.Now()
        var pool = sync.Pool{New: func() interface{} { return make(map[int]int) }}
        for i:=0;i<100000;i++{
                aaa:=pool.Get().(map[int]int)
                for j:=0;j<100;j++{
                        aaa[j]=j
                }
                pool.Put(aaa)
        }
        return time.Since(be)
}
func stringTestNoPool()time.Duration{
        be:=time.Now()
        for i:=0;i<100000;i++{
                var aaa string
                aaa=""
                for j:=0;j<100;j++{
                        aaa=aaa+"abcd"
                }
        }
        return time.Since(be)
}
func stringTestPool()time.Duration{
        be:=time.Now()
        var pool = sync.Pool{New: func() interface{} { return "" }}
        for i:=0;i<100000;i++{
                aaa:=pool.Get().(string)
                aaa=""
                for j:=0;j<100;j++{
                        aaa=aaa+"abcd"
                }
                pool.Put(aaa)
        }
        return time.Since(be)
}
func slinceTestNoPool()time.Duration{
        be:=time.Now()
        for i:=0;i<100000;i++{
                var aaa []int = make([]int, 100)
                aaa=aaa[:0]
                for j:=0;j<100;j++{
                        aaa=append(aaa,j)
                }
        }
        return time.Since(be)
}
func slinceTestPool()time.Duration{
        be:=time.Now()
        var pool = sync.Pool{New: func() interface{} { return make([]int, 100) }}
        for i:=0;i<100000;i++{
                aaa:=pool.Get().([]int)
                aaa=aaa[:0]
                for j:=0;j<100;j++{
                        aaa=append(aaa,j)
                }
                pool.Put(aaa)
        }
        return time.Since(be)
}
 
type mystruct struct{
        id int
        name string
        v []int
        str2num map[string]int
        nextpoi *mystruct
}
 
func dowork(a *mystruct){
        n:=rand.Int()
        a.id=n;
        a.name=strconv.Itoa(n)
        a.v=a.v[:0]
        a.str2num=make(map[string]int)
        for i:=0;i<100;i++{
                a.v=append(a.v ,i)
                a.str2num[strconv.Itoa(i)]=i
        }
}
func structTestNoPool()time.Duration{
        be:=time.Now()
        var wg sync.WaitGroup
        thrs:=100
        wg.Add(thrs)
        for i:=0;i<thrs;i++{
                go func(){
                        defer wg.Done()
                        for j:=0;j<100000;j++{
                                aaa :=&mystruct{}
                                dowork(aaa)
                        }
                }()
        }
        wg.Wait()
        return time.Since(be)
}
func structTestPool()time.Duration{
        be:=time.Now()
        var pool = sync.Pool{New: func() interface{} { return  mystruct{}}}
        var wg sync.WaitGroup
        thrs:=100
        wg.Add(thrs)
        for i:=0;i<thrs;i++{
                go func(){
                        defer wg.Done()
                        for j:=0;j<100000;j++{
                                aaa:=pool.Get().(mystruct)
                                dowork(&aaa)
                                pool.Put(aaa)
                        }
                }()
        }
        wg.Wait()
        return time.Since(be)
}
 
func main(){
        fmt.Println("===map===")
        fmt.Println("no pool:",mapTestNoPool())
        fmt.Println("pool:",mapTestPool())
 
        fmt.Println("===string===")
        fmt.Println("no pool:",stringTestNoPool())
        fmt.Println("pool:",stringTestPool())
 
        fmt.Println("===slince===")
        fmt.Println("no pool:",slinceTestNoPool())
        fmt.Println("pool:",slinceTestPool())
 
        fmt.Println("===符合官方pool条件的===")
        fmt.Println("no pool:",structTestNoPool())
        fmt.Println("pool:",structTestPool())
}
输出：
===map===
no pool: 669.209138ms
pool: 150.040187ms
===string===
no pool: 755.579983ms
pool: 766.081325ms
===slince===
no pool: 7.260073ms
pool: 12.101939ms
===符合官方pool条件的结构体===
no pool: 44.480533049s
pool: 37.973930565s
```

结论：对map或者是结构体，可以使用Pool进行优化，但是丢与string或者切片，就算了吧。



疑问：有接口指定Pool的大小吗？默认值是多少？



> 其他学习链接：（还没看，有空看）
>
> https://blog.csdn.net/qcrao/article/details/105630345

