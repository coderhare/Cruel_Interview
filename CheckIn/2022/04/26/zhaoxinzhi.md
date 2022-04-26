## 一、写在前面

java的foreach中不允许对元素进行add和remove（尽量还是fori的方式吧）

> 参考链接：https://www.jb51.net/article/226347.htm

## 二、java

### 1、原理剖析

其实就是用modCount和expectedModCount实现了一个类似版本号机制的乐观锁。

next和remove方法中都调用了一个`checkForComodification()`方法，他会检查`expectedModCount==modCount`

，如果不相等就抛出一个ConcurrentModificationException并发修改异常。

也就是说，expectedModCount 初始化为 modCount 了，但是后面 expectedModCount 没有修改，而在 remove 和 add 的过程中修改了modCount ，这就导致了执行的时候，通过 checkForComodification 方法来判断两个值是否相等，如果相等了，那么没问题，如果不相等，那就给你抛出一个异常来

fail-fast 机制，也就是快速检测失败机制（其实类似乐观锁？）

### 2、解决方法

如果非要想类似foreach的方式还要remove元素，可以考虑使用listIterator或iterator

