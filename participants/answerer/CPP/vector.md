# vector源码剖析
`sizeof vector`，可以得到24的结果。
这是因为vector在栈上有三个指针，而vector的数据存放于堆中。
这三个指针分别是：
1. 头指针，指向数组的头部
2. 尾指针，指向数组的尾部

`vector::operator[]`:
`T & operator[](size_t i) noexcept;`
vector的索引不会提供越界直接报错，出于性能的考虑，于是可以使用`at()`来进行下标访问：
`T & t(size_t i) `
