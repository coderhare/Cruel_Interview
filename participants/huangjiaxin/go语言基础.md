# Go总结
## Go
**1. slice, map，slice是怎么实现的。**
> 参考[go语言设计与实现](https://draveness.me/golang/docs/)
- go传参时永远是值传递
- array实现
    - array数据结构
    ```go
    type Array struct {
        Elem  *Type // element type
        Bound int64 // number of elements; <0 if unknown yet
    }
    ```
- slice
    - 三种初始化方式
        - 下标
            ```go
            arr := [3]int{1, 2, 3}
            slice := arr[0:1]
            ```
        - 字面量
            ```
            slice := []int{1, 2, 3}
            ```
        - 
    - slice的数据结构
    ```go
    type slice struct {
        array unsafe.Pointer
        len   int
        cap   int
    }
    ```
    - slice的扩容。
        - 如果期望容量大于当前容量的两倍就会使用期望容量；
        - 如果当前切片的长度小于 1024 就会将容量翻倍；
        - 如果当前切片的长度大于 1024 就会每次增加 25% 的容量，直到新容量大于期望容量；
    - slice append元素
        - 切片容量不足时需扩容，否则直接赋值即可
        ![go_slice_append](images/go-slice-append.png)
- 哈希表
    - 

- 字符串
    - 数据结构
        ```go
        type StringHeader struct {
            Data uintptr
            Len  int
        }
        ```
    

2. map和slice并发访问的安全性问题


3. go里面的几种锁和原子变量怎么用，怎么实现的。
4. channel
5. 协程
6. GMP调度模型要讲清楚原理。
7. 锁

**8. golang GC**
- GC的意义
    - 解决因忘记清理不使用的内存区域造成的内存泄漏，避免业务无关的内存申请释放造成的开发负担
    - C/C++存由程序员管理， Java/Python/Go等语言均有垃圾回收机制
    - 无GC的语言一般采用内存泄漏检测工具来辅助检测。C++使用智能指针来解决程序员管理内存的问题
- GC常用算法
    - 引用计数
        - 对每个对象维护一个引用计数。当引用计数减为0时，
    - 标记清除
    - 复制整理
    - 分代回收

- Golang GC

9. epoll原理

