## 一、写在前面

json->go对象的转化时，不能转化成map[string]string类型，因为有可能value是个int。怎么解决这个问题？

今天来整理一下

> 原文链接：https://blog.csdn.net/wxs19970115/article/details/102414651
>
> 样例：https://blog.csdn.net/justinsun221/article/details/107043184/

## 二、go

### 1、方案1: map[string]interface{}

使用map[string]interface{}。

map[string]interface{}类型的map，
在取值的时候，可以使用如下方式避免出现panic：

```go
m := make(map[string]interface{})
x:=m[“notExistsKey”].(int) //若key不存在或者类型不为期待类型则会导致panic
x,ok:=m[“notExistsKey”].(int) //可以通过判断ok，确定是否存在指定类型的值，不会报panic，存在错误时，返回对应类型的默认零值

x,_:=m[“notExistsKey”].(int) //若确定key存在，也可使用该方式
```

但是这有个前提，是你得知道他的value确实是int类型，不然就要用一个switch case去挨个尝试类型。

因此这种方案，适用于那种：

1、整个map里面类型可能鱼龙混杂，但是有一个单独的字段保存具体类型。

2、整个map里面类型可能鱼龙混杂，但是我很明确的知道我想要的字段的具体类型。

### 2、对于struct转map的特殊情况

一个例子：

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	ret := struct{
		Name string `json:"name"`
		Age int `json:"age"`
	}{}
	ret.Name = "justin"
	ret.Age = 18
	m := StructToMap(ret, "json")
	for k,v := range m{
		fmt.Printf("%s:%v \n", k,v)
	}
  /*输出：
name:justin 
age:18  
  */
}
/*
*@brief :struct to map[string]interface{}
*@param :obj interface{}
*@param :tagName tag Name 
*@return :map[string]interface{}
*/
func StructToMap(obj interface{}, tagName string) map[string]interface{} {
	out := make(map[string]interface{})
	t := reflect.TypeOf(obj)
	v := reflect.ValueOf(obj)
	// 取出指针的值
	if v.Kind() == reflect.Ptr{
		v = v.Elem()
	}
	// 判断是否是结构体
	if v.Kind() != reflect.Struct{
		fmt.Println("it is not struct")
		return nil
	}

	for i:= 0 ; i < t.NumField(); i ++ {
		if tagVal := t.Field(i).Tag.Get(tagName); tagVal != ""{
			out[tagVal] = v.Field(i).Interface() 
		}
	}
	return out
}

```



