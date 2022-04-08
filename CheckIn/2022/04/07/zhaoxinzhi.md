## 一、写在前面

今天主要是学习一下关于Java的threadlocal，也是在工作中用到了，所以记录一下。

参考链接：https://www.cnblogs.com/fsmly/p/11020641.html

## 二、Java

### 1、ThreadLocal是什么

多线程访问（读或写）同一个共享变量的时候容易出现并发问题

为了保证线程安全，一般使用者在访问共享变量的时候需要进行额外的同步措施才能保证线程安全性。（一般是加锁）

ThreadLocal是除了加锁这种同步方式之外的一种保证一种规避多线程访问出现线程不安全的方法，当我们在创建一个变量后，如果每个线程对其进行访问的时候访问的都是线程自己的变量这样就不会存在线程不安全问题。（C++也有类似的机制`thread_local_`）

ThreadLocal是JDK包提供的，它提供**线程本地变量**，如果创建一个ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的一个副本，在实际多线程操作的时候，操作的是自己本地内存中的变量，从而规避了线程安全问题，如下图所示![img](https://img2018.cnblogs.com/blog/1368768/201906/1368768-20190613220434628-1803630402.png)

### 2、简单使用threadlocal

```java
package test;

public class ThreadLocalTest {

    //如果去掉static？
    static ThreadLocal<String> localVar = new ThreadLocal<>();

    static void print(String str) {
        //打印当前线程中本地内存中本地变量的值
        System.out.println(str + " :" + localVar.get());
        //清除本地内存中的本地变量
        localVar.remove();
    }

    public static void main(String[] args) {
        Thread t1  = new Thread(new Runnable() {
            @Override
            public void run() {
                //设置线程1中本地变量的值
                localVar.set("localVar1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                //调用打印方法
                print("thread1");
                //打印本地变量
                System.out.println("after remove : " + localVar.get());
            }
        });

        Thread t2  = new Thread(new Runnable() {
            @Override
            public void run() {
                localVar.set("localVar2");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                print("thread2");
                System.out.println("after remove : " + localVar.get());
            }
        });
        t1.start();
        t2.start();
    }
}

输出结果：
thread1 :localVar1
after remove : null
thread2 :localVar2
after remove : null
```

所以可以发现，对于static类型的类变量，他是提升到了线程级别的变量。



### 3、原理

首先要明确，Thread和ThreadLocal两个类都在java.lang包下。Thread中声明了一个default级别修饰符的threadLocals，但也只是在这里和exit()方法中置为null了，其他时候啥用没干。

![image-20220405163214560](/Users/bytedance/Library/Application Support/typora-user-images/image-20220405163214560.png)

threadLocals这个map的具体操作是在ThreadLocal类中实现的，当你传入一个thread对象的时候，会调取当前中的value

> **进一步理解面向对象：**
>
> 其实，一个类中包含另一个类对象，与线程啥的没有直接关系，因为类只是代表了一种数据(在内存中)的定义和组织方式，对象是在堆内存中真正按照这个组织方式生成的一个实例而已。
>
> 所以这里其实比如`Father类中有一个ThreadLocal类型的变量local`，看似是ThreaLlocal对象local是在某一个包含在某一个对象father中的，看起来他应该是该father对象的一部分，但是其实只是说我father中有一个组织格式为ThreadLocal的一部分内存空间，我可以通过local这个引用来去按照ThreadLocal格式调取对应内存中的数据而已。这也是为什么ThreadLocal类型的数据必须要用过get和set方法，而不能直接放到变量(或者叫属性)中，因为如果是属性，那只能是按照对应组织格式给取过来，不够灵活；只有通过方法，才可以修改具体拿到的值是什么。（比如getDomain中，我就可以根据需要(其实正是ThreadLocal类型的isSG字段)来给出是va的domain还是sg的domain。）
>
> 对于这里来说，放置在内存某区域的一个对象local，不管被赋值到哪里，引用到别的随便哪个地方多少次，我local.get()的时候都是根据具体这一行代码执行的时候的线程名称去取的。（get()具体逻辑为：获取在执行的线程对象(Thread类型)->获取其threadLocals变量(Map类型)->进行读取）



3.1 set方法源码

```java
//在ThreadLocal.class中
public void set(T value) {
    //(1)获取当前线程（调用者线程）
    Thread t = Thread.currentThread();
    //(2)获取Thread变量中的threadLocals字段、
    ThreadLocalMap map = getMap(t);
    //(3)如果map不为null，就直接添加本地变量，key为当前定义的ThreadLocal变量的this引用，值为添加的本地变量值
    if (map != null)
        map.set(this, value);
    //(4)如果map为null，说明首次添加，需要首先创建出对应的map
    else
        createMap(t, value);
}
```



### 4、ThreadLocal不可继承

同一个ThreadLocal变量在父线程中被设置值后，在子线程中是获取不到的。（threadLocals中为当前调用线程对应的本地变量，所以二者自然是不能共享的）

```java
package test;

public class ThreadLocalTest2 {

    //(1)创建ThreadLocal变量
    public static ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        //在main线程中添加main线程的本地变量
        threadLocal.set("mainVal");
        //新创建一个子线程
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("子线程中的本地变量值:"+threadLocal.get());
            }
        });
        thread.start();
        //输出main线程中的本地变量值
        System.out.println("mainx线程中的本地变量值:"+threadLocal.get());
    }
}

输出结果：
main线程中的本地变量值:mainVal
子线程中的本地变量值:null
```



### 5、可继承的InheritableThreadLocal类

在上面说到的ThreadLocal类是不能提供子线程访问父线程的本地变量的，而InheritableThreadLocal类则可以做到这个功能，下面是该类的源码

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {

    protected T childValue(T parentValue) {
        return parentValue;
    }

    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

InheritableThreadLocals类通过重写getMap和createMap两个方法将本地变量保存到了具体线程的inheritableThreadLocals变量中，当线程通过InheritableThreadLocals实例的set或者get方法设置变量的时候，就会创建当前线程的inheritableThreadLocals变量。而父线程创建子线程的时候，ThreadLocalMap中的构造函数会将父线程的inheritableThreadLocals中的变量复制一份到子线程的inheritableThreadLocals变量中。



### 6、从ThreadLocalMap看ThreadLocal使用不当的内存泄漏问题

一句话总结：ThreadLocalMap是弱引用，大概的意思是；因为这是Entry类型，GC只会回收掉对应的key，而不会回收掉对应的value，所以可能会导致value无迹可寻，造成内存泄漏？



