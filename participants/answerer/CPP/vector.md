# vector源码剖析

vector符合RAII思想，离开作用域即释放内存。

`sizeof vector`，可以得到24的结果。
这是因为vector在栈上有三个指针，而vector的数据存放于堆中。
这三个指针分别是：
1. 头指针，指向数组的头部
2. 尾指针，指向数组的尾部
3. 第三个指针，指向的是实际空间大小的结束地址（capacity）

`vector::operator[]`:
`T & operator[](size_t i) noexcept;`
vector的索引不会提供越界直接报错，出于性能的考虑，于是可以使用`at()`来进行下标访问：
`T & t(size_t i) at`，如果`i >= a.size()`则会抛出`std::out_of_range`让程序提前终止。

如果调用了C语言的库，需要与指针打交道，可以使用`data`成员函数，得到一个vector对应的数组的首指针。
`data`是一个弱引用，因此如果需要在作用域外仍然保持`data()`对数组的弱引用有效，可以把语句块内的vector对象移动到
外面的一个vector对象。（延长生命周期）。注意调用`resize`或者`push_back`导致`size > capacity`，则会发生`realloc`，导致弱引用失效

C++ STL容器的迭代器是符合迭代器模式的，通过重载`++`运算符，来使得多个容器的迭代器操作上一致。
但是对于对象而非内置类型而言，前置`++`与后置`++`的区别可能很大，比如说：
对于链表的`++`而言
```c++
Iterator &operator++(){ //后置++
    cur = cur->next;
    return *this;
}
```
```c++
Iterator operator++(int){ //前置++
    Iterator tmp = *this;
    this->operator()++;
    return tmp;
}
```
可以明显地发现，前置`++`会先产生一份拷贝，然后再调用后置`++`运算符




