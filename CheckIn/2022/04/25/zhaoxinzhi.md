## 一、写在前面

关于Hashmap死循环问题。

## 二、Hashmap

只会发生在jdk1.7版本中，1.8就修复了这个问题（头插法改尾插法）



### 1、死循环产生的主要原因

 头插法+链表+多线程并发+ 扩容场景

现有链表A->B->C，两个线程t1和t2，分别取得头结点，并且next都是B。

然后t1扩容至完成，C->B->A

然后t2再执行，这样A->B->A->B.....循环了。

### 2、解决方案

**1、采用ConcurrentHashMap替代Hashmap（推荐）**

2、使用线程安全的容器HashTable，但是性能低。

3、synchronized或Locked加锁。



## 三、进阶问题

ConcurrentHashMap底层是如何实现线程安全难道？

jdk7和8有什么区别？

分段锁有什么优缺点？
