moka面经
1、spring的生命周期
1.实例化一个Bean－－也就是我们常说的new；
2.按照Spring上下文对实例化的Bean进行配置－－也就是IOC注入；
3.如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String)方法，也就是根据就是Spring配置文件中Bean的id和name进行传递
4.如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现setBeanFactory(BeanFactory)也就是Spring配置文件配置的Spring工厂自身进行传递；
5.如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，和4传递的信息一样但是因为ApplicationContext是BeanFactory的子接口，所以更加灵活
6.如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessBeforeInitialization()方法，BeanPostProcessor经常被用作是Bean内容的更改，由于这个是在Bean初始化结束时调用那个的方法，也可以被应用于内存或缓存技术
7.如果Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法。
8.如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessAfterInitialization()，打印日志或者三级缓存技术里面的bean升级
9.以上工作完成以后就可以应用这个Bean了，那这个Bean是一个Singleton的，所以一般情况下我们调用同一个id的Bean会是在内容地址相同的实例，当然在Spring配置文件中也可以配置非Singleton，这里我们不做赘述。
10.当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，或者根据spring配置的destroy-method属性，调用实现的destroy()方法


2、分布式事务二段提交的全部过程
准备阶段的资源锁定，存在性能问题，严重时会造成死锁问题
提交事务请求后，出现网络异常，部分数据收到并执行，会造成一致性问

3、数据库sql的优化的手段
答：主要进行了答了索引的优化，和冷热数据分离，大数据分页，覆盖索引，索引的顺序等
https://javaguide.cn/database/mysql/mysql-high-performance-optimization-specification-recommendations.html#_12-%E7%A6%81%E6%AD%A2%E4%BD%BF%E7%94%A8-order-by-rand-%E8%BF%9B%E8%A1%8C%E9%9A%8F%E6%9C%BA%E6%8E%92%E5%BA%8F

4、volatile,sychornized,AQS,ThreadPoolExecutor的原理说一下

答：挨个分点去答 

5、分布式缓存、分布式锁,

答：讲解了redission的原理，包括进行原子性加锁和解锁，设置超市时间，watchDog的作用，分布式锁的具体实现

6、netty的原理说一下
答：讲了一下netty的基本架构，reactor模型，链试编程
7、你项目中的DDD领域模型是怎么设计的
答：根据业务身份面向接口编程，领域边界进行原子性划分，领域模型提供底层原子性服务，和代码级别的事务回滚，
service层提供业务通用流程的编排和沉淀不同业务线的能力类，防腐层屏蔽外部接口的差异和对象的包装。
8、RocketMQ，Sentinel的原理说下
答： 一个是高可用的分布式消息队列，一个是分布式的限流器