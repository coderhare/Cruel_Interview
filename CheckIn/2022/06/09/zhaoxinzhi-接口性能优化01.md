

## 一、写在前面

关于接口性能优化

> 参考链接：https://mp.weixin.qq.com/s/40uXyuJG0nuMNBDmFZmnbg



## 二、性能优化

### 1、首先查询慢接口有哪些

因为过早优化是万恶之源，所以一定要知道有哪些接口是值得优化的。

> 比如：有20多个**慢接口**，5个接口**响应时间**超过5s，1个超过10s，其余的都在2s以上，**稳定性**不足99.8%。
> 这种数据才应该是性能优化的原动力。

### 2、哪些问题会引起接口性能问题？

不完全的总结：

- 数据库慢查询

- - 深度分页问题
  - 未加索引
  - 索引失效
  - join过多
  - 子查询过多
  - in中的值太多
  - 单纯的数据量过大

- 业务逻辑复杂

- - 循环调用
  - 顺序调用

- 线程池设计不合理

- 锁设计不合理

- 机器问题（fullGC，机器重启，线程打满）

### 3、数据库

#### 1、深度分页

在很多不同的场景中都需要使用到分页，比如前端页面展示的分页功能；从数据库中分批取全量数据，都需要分页。所以sql可以优化如下：

```sql
-- 优化前
select name,code from student limit 1000000,20
-- 优化后
select name,code from student where id>1000000  limit 20
```

注意1：如果是场景是前端页面展示分页功能，则这个方式需要接口的调用方配合改造，把上次查询出来的最大id以参数的方式传给接口提供方。

注意2：如果是数据库中分批取全量数据，则需要保证id是连续的，中间如果删掉了一条数据，则不能用这种方式。

#### 2、未加索引

注意1：先思考有没有必要加索引。

注意2：加索引的alter操作，可能引起锁表，执行sql的时候一定要在低峰期

#### 3、索引失效

![图片](https://mmbiz.qpic.cn/mmbiz/YrLz7nDONjFUhGlIwUrUuicVZN5tT3fLsiaOOiciawmr7LfVEWic3JnrBjZfYRjTCibTtZdeQv1Y2j8D2kWqrHLbBOlg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

注意1：善于使用explain关键字分析

#### 4、join过多或子查询过多

一般来说，不建议使用子查询，可以把子查询改成join来优化。同时，join关联的表也不宜过多，一般来说2-3张表还是合适的。

在大多数情况下join是在内存里做的，如果匹配的量比较小，或者join_buffer设置的比较大，速度也不会很慢。但是，当join的数据量比较大的时候，mysql会采用在硬盘上创建临时表的方式进行多张表的关联匹配，这种显然效率就极低，本来磁盘的IO就不快，还要关联。

一般遇到这种情况的时候就建议从代码层面进行拆分，在业务层先查询一张表的数据，然后以关联字段作为条件查询关联表形成map，然后在业务层进行数据的拼装。一般来说，索引建立正确的话，会比join快很多，毕竟内存里拼接数据要比网络传输和硬盘IO快得多。

#### 5、in的元素过多

这种问题，如果只看代码的话不太容易排查，最好结合监控和数据库日志一起分析。如果一个查询有in，in的条件加了合适的索引，这个时候的sql还是比较慢就可以高度怀疑是in的元素过多。

解决方法：把元素分个组，每组查一次。想再快的话，可以再引入多线程。

进一步的，为了更好的做到有效的分组，最好还是有个限制

```
select id from student where id in (1,2,3 ...... 1000) limit 200
```

当然了，最好是在代码层面做个限制

```
if (ids.size() > 200) {
    throw new Exception("单次查询数据量不能超过200");
}
```

#### 6、单纯的数据量过大

这种问题，往往需要变动整个的数据存储架构。或者是对底层mysql分表或分库+分表；或者就是直接变更底层数据库，把mysql转换成专门为处理大数据设计的数据库。这种工作是个系统工程，需要严密的调研、方案设计、方案评审、性能评估、开发、测试、联调，同时需要设计严密的数据迁移方案、回滚方案、降级措施、故障处理预案。除了以上团队内部的工作，还可能有跨系统沟通的工作，毕竟做了重大变更，下游系统的调用接口的方式有可能会需要变化。

### 4、业务逻辑复杂

#### 2.1 循环调用

这种情况，一般都循环调用同一段代码，每次循环的逻辑一致，前后不关联。比如说，我们要初始化一个列表，预置12个月的数据给前端：

```java
List<Model> list = new ArrayList<>();
for(int i = 0 ; i < 12 ; i ++) {
    Model model = calOneMonthData(i); // 计算某个月的数据，逻辑比较复杂，难以批量计算，效率也无法很高
    list.add(model);
}
```

这种显然每个月的数据**计算相互都是独立的，可以采用多线程方式进行**：

```java
// 建立一个线程池，注意要放在外面，不要每次执行代码就建立一个，具体线程池的使用就不展开了
public static ExecutorService commonThreadPool = new ThreadPoolExecutor(5, 5, 300L,
        TimeUnit.SECONDS, new LinkedBlockingQueue<>(10), commonThreadFactory, new ThreadPoolExecutor.DiscardPolicy());

// 开始多线程调用
List<Future<Model>> futures = new ArrayList<>();
for(int i = 0 ; i < 12 ; i ++) {
    Future<Model> future = commonThreadPool.submit(() -> calOneMonthData(i););
    futures.add(future);
}

// 获取结果
List<Model> list = new ArrayList<>();
try {
   for (int i = 0 ; i < futures.size() ; i ++) {
      list.add(futures.get(i).get());
   }
} catch (Exception e) {
   LOGGER.error("出现错误：", e);
}
```

#### 2.2 顺序调用

如果不是类似上面循环调用，而是一次次的顺序调用，而且调用之间没有结果上的依赖，那么也可以用多线程的方式进行，例如：



![图片](https://mmbiz.qpic.cn/mmbiz/YrLz7nDONjFUhGlIwUrUuicVZN5tT3fLsRjM3e4RQ1OS90QM72xoibJa1uUIQnJGbvsorRvPLbvGEhgLYDuRh2tw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)



代码上看：

```
A a = doA();
B b = doB();

C c = doC(a, b);

D d = doD(c);
E e = doE(c);

return doResult(d, e);
```

那么可用CompletableFuture解决

```
CompletableFuture<A> futureA = CompletableFuture.supplyAsync(() -> doA());
CompletableFuture<B> futureB = CompletableFuture.supplyAsync(() -> doB());
CompletableFuture.allOf(futureA,futureB) // 等a b 两个任务都执行完成

C c = doC(futureA.join(), futureB.join());

CompletableFuture<D> futureD = CompletableFuture.supplyAsync(() -> doD(c));
CompletableFuture<E> futureE = CompletableFuture.supplyAsync(() -> doE(c));
CompletableFuture.allOf(futureD,futureE) // 等d e两个任务都执行完成

return doResult(futureD.join(),futureE.join());
```

这样A B 两个逻辑可以并行执行，D E两个逻辑可以并行执行，最大执行时间取决于哪个逻辑更慢。

## 3、线程池设计不合理

有的时候，即使我们使用了线程池让任务并行处理，接口的执行效率仍然不够快，这种情况可能是怎么回事呢？

这种情况首先应该怀疑是不是线程池设计的不合理。我觉得这里有必要回顾一下线程池的三个重要参数：核心线程数、最大线程数、等待队列。这三个参数是怎么打配合的呢？当线程池创建的时候，如果不预热线程池，则线程池中线程为0。当有任务提交到线程池，则开始创建核心线程。

<img src="https://mmbiz.qpic.cn/mmbiz/YrLz7nDONjFUhGlIwUrUuicVZN5tT3fLsIIiawKn14zsNkkDZM6WNfvTTMsTNejWGfyYQwYt212dU119fXoByQNw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />

当核心线程全部被占满，如果再有任务到达，则让任务进入等待队列开始等待。

<img src="https://mmbiz.qpic.cn/mmbiz/YrLz7nDONjFUhGlIwUrUuicVZN5tT3fLsFYLBdTWyVRs1iaDDKeztGLgNa1iawh0lghZibbFVMeLeXeIEkwtlibUxDA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />

如果队列也被占满，则开始创建非核心线程运行。

<img src="https://mmbiz.qpic.cn/mmbiz/YrLz7nDONjFUhGlIwUrUuicVZN5tT3fLss0XvO9Zcf6gicB9n2Kt2EHRkDGO1RSeBdw4iag1y7JvOjunFLC3m85pg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />

如果线程总数达到最大线程数，还是有任务到达，则开始根据线程池抛弃规则开始抛弃。



<img src="https://mmbiz.qpic.cn/mmbiz/YrLz7nDONjFUhGlIwUrUuicVZN5tT3fLsGrDowplkIOwgdbpYnJ4dJTic2FTQ4icEOApSN1rVicQjBUib18MslHIRPA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />



那么这个运行原理与接口运行时间有什么关系呢？

- 核心线程设置过小：核心线程设置过小则没有达到并行的效果
- 线程池公用，别的业务的任务执行时间太长，占用了核心线程，另一个业务的任务到达就直接进入了等待队列
- 任务太多，以至于占满了线程池，大量任务在队列中等待

在排查的时候，根据问题出现的原因，确定解决方式。

解决方式无非就是调整线程池参数，按照业务拆分线程池等等。
