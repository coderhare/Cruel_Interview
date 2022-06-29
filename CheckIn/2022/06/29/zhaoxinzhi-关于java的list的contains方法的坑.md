## 一、写在前面

关于Java的list的contains方法的坑

https://baijiahao.baidu.com/s?id=1679623586178323643&wfr=spider&for=pc

## 二、Java



1、对这种情况

![img](https://pics2.baidu.com/feed/503d269759ee3d6d263f914c7edd0c254d4ade8a.jpeg?token=cf52426ea135e3416b76df3f3d8d2ba1)

第17行是会返回true的，因为底层是调用了.equals()方法，而对于String，这个方法比较的就是字符串内容，而不是对象的地址，所以是会返回true的。
