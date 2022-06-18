## 一、写在前面

迭代器模式

参考链接：https://blog.csdn.net/Lyon_Nee/article/details/119466202

## 二、golang设计模式

迭代器模式一般不太用，但是在设计框架或者设计一个工具的时候，有的时候是很好用的。

### 1、适用场景

我有一个书架（BookShelf），书架内摆放了很多书籍（Book），我想要不直接暴露BookShelf内部数据的情况下，获取书架上所有的书籍信息。（即不想定义成一个slice然后for range 遍历）

这个场景就很适合用迭代器模式了。

类似的场景还有，比如当设计一个中间件的时候，想用类似filter的方式，类似洋葱模型，一层一层的剥离与拦截，这时候也可以用到迭代器模式。

### 2、迭代器模式的思想

当我们想要获取所有书籍信息时，直接获取BookShelf的迭代器（Iterator），我们通过对迭代器的方法Next（）调用来遍历获取书籍信息。迭代器的职责就是遍历数据集合的。

### 3、举例

#### 3.1 迭代器模式相关工作

首先，我们知道迭代器模式主要适用于数据集合，那么我们就可以定义一个集合（Aggregate）接口，这个接口只有一个方法，就是返回给我们一个迭代器（Iterator）。

```go
// 集合接口
type Aggregate interface {
	// 返回一个迭代器
	GetIterator() Iterator
}
```

然后，我们还需要一个定义一个迭代器（Iterator）接口，这个接口有两个方法，分别是 HasNext（）bool，判断是否还有值；Next（）interface{}，返回下一个值。

```go
// 迭代器接口
type Iterator interface {
	// 判断是否还有值
	HasNext() bool
	// 取下一个值
	Next() interface{}
}
```

#### 3.2 接入业务

接下来我们还需要定义一个Book结构体，并分别定于实现了Aggregate接口书架类型BookShelf，以及实现了 Iterator接口的 BookShelfIterator

书架BookShelf中有一个书的集合Books。并且实现了集合（Aggregate）接口的 返回迭代器Iterator的方法。

书架迭代器BookShelfIterator实现 Iterator接口的两个方法。

```go
// 书
type Book struct {
	// 名字
	Name string
	// 作者
	Author string
}
Book只有两个string类型字段，书名Name，作者Author。

// 书架，一个存放书籍的集合
type BookShelf struct {
	books []Book
}

// 书架迭代器
type BookShelfIterator struct {
	// 当前遍历位置
	index int
	// 所指向书架
	bookShelf *BookShelf
}
```

```go
// 书架实现集合接口
func (bs *BookShelf) GetIterator() Iterator {
	// 每次都会创建一个新的迭代器，从 index=0 位置开始遍历
	return &BookShelfIterator{bookShelf: bs}
}

// 书架迭代器实现迭代器接口
func (i *BookShelfIterator) HasNext() bool {
	return i.index < len(i.bookShelf.books)
}

// 书架迭代器实现迭代器接口
func (i *BookShelfIterator) Next() interface{} {
	book := i.bookShelf.books[i.index]
	i.index++
	return book
}

```

#### 3.3 测试

接下来我们测试一下代码：

```go
func main() {
	// 声明一个书架对象，并添加一些书籍
	bookShelf := &BookShelf{
		books: []Book{
			{Name: "以太坊技术详解与实战", Author: "闫莺，郑恺，郭众鑫"},
			{Name: "大话代码架构", Author: "田伟，郎小娇"},
			{Name: "GO语言公链开发实战", Author: "郑东旭，杨明珠，潘盈瑜，翟萌"},
			{Name: "区块链原理、设计与应用", Author: "杨保华，陈昌"},
			{Name: "精通区块链智能合约开发", Author: "熊丽兵"},
			{Name: "C程序设计", Author: "谭浩强"},
		},
	}
 
	// 获取书架的迭代器
	iterator := bookShelf.GetIterator()
 
	// 遍历，直到没有下一本书（HasNext == flase）
	for iterator.HasNext() {
		book := iterator.Next().(Book)
		fmt.Printf("书名：%s, 作者：%s \n", book.Name, book.Author)
	}
}
```

### 4、总结

事实上，对于上述例子，我们是在book这个业务model中包了一层bookshelf，但是实际上可以根据具体业务抽象出更高层的一些实体，并且在这些实体上应用迭代器模式。

但是个人认为，迭代器模式还是比较繁琐的，除非在设计开源框架等工作时，对一些高频接口和高频实体可以这么使用，来降低用户使用的成本，简化代码，提高易用性。其他时候没必要。
