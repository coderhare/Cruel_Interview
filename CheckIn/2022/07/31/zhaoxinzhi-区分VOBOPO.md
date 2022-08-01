# 一、写在前面

VO，BO，PO，DO，DTO的区别？

> 参考链接：https://www.jianshu.com/p/55cb67cd4110
>
> https://www.jianshu.com/p/ccdbef3ec75f

## 二、架构

### 1、DTO（Data Transfer Object）数据传输对象

DTO是一个比较特殊的对象，他有两种存在形式：

- 在后端，他的存在形式是java对象，也就是在controller里面作为参数定义的，通常在后端不需要关心怎么从json转成java对象的，这个都是由一些成熟的框架帮你完成啦，比如spring框架
- 在前端，他的存在形式通常是js里面的对象（也可以简单理解成json），也就是通过ajax请求的那个数据体

### 2、VO（Value Object）值对象

一句话理解：前端用的json

VO就是展示用的数据，不管展示方式是网页，还是客户端，还是APP，只要是这个东西是让人看到的，这就叫VO
 VO主要的存在形式就是js里面的对象

> **VO和DTO的区别**
> 主要有两个区别
> 一个是字段不一样，VO根据需要会删减一些字段
> 另一个是值不一样，VO会根据需要对DTO中的值进行展示业务的解释

### 3、PO（Persistant Object）持久对象

PO比较好理解
 简单说PO就是数据库中的记录，一个PO的数据结构对应着库中表的结构，表中的一条记录就是一个PO对象
 通常PO里面除了get，set之外没有别的方法
 对于PO来说，数量是相对固定的，一定不会超过数据库表的数量
 等同于Entity，这俩概念是一致的

具体到类名里，也可以加PO也可以不加，比如用户类就叫User，从mapper里取（mybatis）

比如命令类就叫Command（解耦的时候可以用，发布一个命令，让mq消费命令）

### 4、BO（Business Object）业务对象

BO就是PO的组合
 简单的例子比如说PO是一条交易记录，BO是一个人全部的交易记录集合对象
 复杂点儿的例子PO1是交易记录，PO2是登录记录，PO3是商品浏览记录，PO4是添加购物车记录，PO5是搜索记录，BO是个人网站行为对象
 BO是一个业务对象，一类业务就会对应一个BO，数量上没有限制，而且BO会有很多业务操作，也就是说除了get，set方法以外，BO会有很多针对自身数据进行计算的方法

（比如这个方法例子：`CommandBO.from(Command command)`，代表把PO转化成BO）

从最后一部分的图示来看，BO也画成横跨两层的。原因是现在很多持久层框架自身就提供了数据组合的功能，因此BO有可能是在业务层由业务来拼装PO而成，也有可能是在数据库访问层由框架直接生成
 很多情况下为了追求查询的效率，框架跳过PO直接生成BO的情况非常普遍，PO只是用来增删改使用

> **BO和DTO的区别**
> 区别主要是就是字段的删减
> BO对内，为了进行业务计算需要辅助数据，或者是一个业务有多个对外的接口，BO可能会含有很多接口对外所不需要的数据，因此DTO需要在BO的基础上，只要自己需要的数据，然后对外提供
> 在这个关系上，通常不会有数据内容的变化，内容变化要么在BO内部业务计算的时候完成，要么在解释VO的时候完成

### 5、DO的两个版本，主要指Domain Object

现在主要有两个版本
 一个是阿里巴巴的开发手册中的定义
 DO（ Data Object）这个等同于上面的PO
 另一个是在DDD（Domain-Driven Design）领域驱动设计中
 **DO（Domain Object）这个等同于上面的BO**

### 6、POJO(plain ordinary java object) 无规则简单java对象

实际就是普通JavaBeans,使用POJO名称是为了避免和EJB混淆起来, 而且简称比较直接. 其中有一些属性及其getter setter方法的类,有时可以作为value object或dto(Data Transform Object)来使用.当然,如果你有一个简单的运算属性也是可以的,但不允许有业务方法,也不能携带有connection之类的方法。

POJO是Plain Ordinary Java Objects的缩写不错，但是它通指没有使用Entity Beans的普通java对象，可以把POJO作为支持业务逻辑的协助类。

### 7、DAO (data access object)数据访问对象

是一个 sun 的一个标准 j2ee 设计模式， 这个模式中有个接口就是 DAO ，它负持久层的操作。为业务层提供接口。此对象用于访问数据库。通常和 PO 结合使用， DAO 中包含了各种数据库的操作方法。通过它的方法 , 结合 PO 对数据库进行相关的操作。夹在业务逻辑与数据库资源中间。配合 VO, 提供数据库的 CRUD 操作。

在mybatis中就是各种mapper那一层

### 8、DAL(Data Access Layer)



> **DAO和DAL的区别：**
>
> 数据访问层(*DAL*)是业务逻辑层和持久性/存储层之间存在的系统层。 *DAL*可能是单个类,也可能由多个数据访问对象(*DAO*)组成。
>
> DAO（Data Access Object）侧重ORM对象关系映射，DAO曾往往提供的是ORM操作。
>
> DAL（Data Access Layer）多用于分布式系统
>
> 分布式系统提供水平伸缩能力，应用层水平扩展后，会使数据库连接资源相对较少成为瓶颈，此时需要对数据库分库、分表，DAL要求支持透明分库分表，DAO层提供的往往是sql

### 9、实例图

![img](https://upload-images.jianshu.io/upload_images/3710706-cd9c3d167ff9905c.png?imageMogr2/auto-orient/strip|imageView2/2/w/603/format/webp)
