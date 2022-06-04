

## 一、写在前面

关于golang的map，需要知道的一些坑

http://www.zzvips.com/article/101153.html



## 二、golang

map (映射)是 Golang 中的方便而强大的内建数据结构，是一个同种类型元素的无序组，元素通过另一类型唯一的键进行索引。其键可以是任何相等性操作符支持的类型， 如整数、浮点数、复数、字符串、指针、接口（只要其动态类型支持相等性判断）、结构以及数组。 切片不能用作映射键，因为它们的相等性还未定义。与切片一样，映射也是引用类型。 若将映射传入函数中，并更改了该映射的内容，则此修改对调用者同样可见。未初始化的映射值为 nil。

### 1、map的元素不可取址

map中的元素并不是一个变量，而是一个值。因此，我们不能对map的元素进行取址操作。

```go
var m = map[int]int {
  0: 0,
  1: 1,
}
func main() {
  fmt.Println(&m[0]) // error: cannot take the address of m[0]
}
```

因此，当 map 的元素为结构体类型的值，那么无法直接修改结构体中的字段值。 



下面这段函数，是判断如果一个人的年龄<50，那么就更改isDead字段为true

报错如下：`cannot assign to struct field personMap[name].isDead in map`

```go
package main
 
import (
    "fmt"
)
 
type person struct {
  name  string
  age  byte
  isDead bool
}
 
func whoIsDead(personMap map[string]person) {
  for name, _ := range personMap {
    if personMap[name].age > 90 {
      personMap[name].isDead = true
    } 
  } 
}
 
func main() {
	p1 := person{name: "abc", age: 100, isDead: false}
	p2 := person{name: "xxx", age: 99, isDead: false}
	p3 := person{name: "zzz", age: 20, isDead: false}
  personMap := map[string]person{
    p1.name: p1,
    p2.name: p2,
    p3.name: p3,
  } 
  whoIsDead(personMap)
  
  for _, v :=range personMap {
    if v.isDead {
      fmt.Printf("%s is dead\n", v.name)
    } 
  } 
}
```

原因是 map 元素是无法取址的，也就**说可以得到 personMap[name]，但是无法对其进行修改**。解决办法有二，一是 map 的 value用 strct 的指针类型，二是使用临时变量，每次取出来后再设置回去。

（1）将map中的元素改为struct的指针。

```go
package main

import (
	"fmt"
)

type person struct {
	name   string
	age    byte
	isDead bool
}
/*
这样也是ok的：
func whoIsDead(personMap map[string]*person) {
	for name, _ := range personMap {
		if personMap[name].age > 90 {
			personMap[name].isDead = true
		}
	}
}
*/
func whoIsDead(personMap interface{}) {
	m := personMap.(map[string]*person)
	for name, _ := range m {
		if m[name].age > 90 {
			m[name].isDead = true
		}
	}
}
func main() {
	p1 := person{name: "abc", age: 100, isDead: false}
	p2 := person{name: "xxx", age: 99, isDead: false}
	p3 := person{name: "zzz", age: 20, isDead: false}
	personMap := map[string]*person{
		p1.name: &p1,
		p2.name: &p2,
		p3.name: &p3,
	}
	whoIsDead(personMap)

	for _, v := range personMap {
		if v.isDead {
			fmt.Printf("%s is dead\n", v.name)
		}
	}
}

```



（2）使用临时变量覆盖原来的元素。

```go
package main
 
import (
    "fmt"
)
 
type person struct {
  name  string
  age  byte
  isDead bool
}
 
func whoIsDead(people map[string]person) {
  for name, _ := range people {
    if people[name].age < 50 {
      tmp := people[name]
      tmp.isDead = true
      people[name] = tmp
    } 
  } 
}
 
func main() {
  p1 := person{name: "zzy", age: 100}
  p2 := person{name: "dj", age: 99}
  p3 := person{name: "px", age: 20}
  personMap := map[string]person {
    p1.name: p1,
    p2.name: p2,
    p3.name: p3,
  } 
  whoIsDead(personMap)
  
    for _, v :=range personMap {
        if v.isDead {
            fmt.Printf("%s is dead\n", v.name)
        } 
    } 
}
```



### 2、map并发读写问题

因为map是线程不安全的。

错误写法：

```go
package main
 
import (
    "fmt"
    "time"
)
 
var m = make(map[int]int)
 
func main() {
    //一个go程写map
    go func(){
        for i := 0; i < 10000; i++ {
            m[i] = i 
        } 
    }()
 
    //一个go程读map
    go func(){
        for i := 0; i < 10000; i++ {
            fmt.Println(m[i]) 
        } 
    }()
    time.Sleep(time.Second*20)
}
```

正确写法：

```go
package main
 
import (
    "fmt"
    "time"
    "sync"
)
 
var m = make(map[int]int)
var rwMutex sync.RWMutex
 
func main() {
    //一个go程写map
    go func(){
        rwMutex.Lock()
        for i := 0; i < 10000; i++ {
            m[i] = i 
        } 
        rwMutex.Unlock()
    }()
 
    //一个go程读map
    go func(){
        rwMutex.RLock()
        for i := 0; i < 10000; i++ {
            fmt.Println(m[i]) 
        } 
        rwMutex.RUnlock()
    }()
    time.Sleep(time.Second*20)
}

```

