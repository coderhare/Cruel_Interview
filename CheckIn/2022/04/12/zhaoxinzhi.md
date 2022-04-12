## 一、写在前面

今天学习下java8新增的特性stream流。

## 二、Java

### 1、简介

Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。

Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。

元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

```java
Java流，处理过程流程图：
  先把Collect类型数据转换成stream类型 ----> 	进行操作  ---->		最后在转换回collect类型
+--------------------+       +------+   +------+   +---+   +-------+
| stream of elements +-----> |filter+-> |sorted+-> |map+-> |collect|
+--------------------+       +------+   +------+   +---+   +-------+
```

### 2、什么是流

Stream（流）是一个来自数据源的元素队列并支持聚合操作

- **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。
- **元素**是特定类型的对象，形成一个队列。 Java中的Stream并**不会存储元素**，而是按需计算。
- **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

和以前的Collection操作不同， Stream操作还有两个基础的特征：

- **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
- **内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

### 3、如何使用

在 Java 8 中,通过 **Collection**接口的两个方法来生成流：

- **stream()** − 为集合创建串行流。
- **parallelStream()** − 为集合创建并行流。

流在管道中传递，参数必须是函数，这里常用匿名函数。

> **附lambda表达式语法：**
>
> (parameters)-> expression或 (parameters)->{statements; }
>
> 注意前者是表达式，后者是语句。
>
> 其中形参列表的数据类型可以不写，自动判断



### 4、实例

```
list转map：
        List<String> strList = Arrays.asList("a", "ba", "bb", "abc", "cbb", "bba", "cab");
        Map<Integer, String> strMap = new HashMap<Integer, String>();

        strMap = strList.stream()
                .collect( Collectors.toMap( str -> strList.indexOf(str), str -> str ) );
        
        strMap.forEach((key, value) -> {
            System.out.println(key+"::"+value);
        });
```

#### forEach

Stream 提供了新的方法 'forEach' 来迭代流中的每个数据。以下代码片段使用 forEach 输出了10个随机数：

forEach里面可以穿方法引用（==这样应该是加方法引用吧？==）

```java
Random random = new Random(); random.ints().limit(10).forEach(System.out::println);
```

------

#### map

map 方法用于映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数：

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); // 获取对应的平方数 
List<Integer> li=numbers.stream().map(i->i*i).distinct().collect(Collectors.toList());
```

------

#### filter

filter 方法用于通过设置的条件过滤出元素。以下代码片段使用 filter 方法过滤出空字符串：

```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
// 获取空字符串的数量 
long count = strings.stream().filter(string -> string.isEmpty()).count();
```

------

#### limit

limit 方法用于获取指定数量的流。 以下代码片段使用 limit 方法打印出 10 条数据：

```java
Random random = new Random(); 
random.ints().limit(10).forEach(System.out::println);
```

------

#### sorted

sorted 方法用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序：

```java
Random random = new Random(); 
random.ints().limit(10).sorted().forEach(System.out::println);
```

------

#### 并行（parallel）程序

parallelStream 是流并行处理程序的代替方法。以下实例我们使用 parallelStream 来输出空字符串的数量：

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
// 获取空字符串的数量 
long count = strings.parallelStream().filter(string -> string.isEmpty()).count();
```

#### Collectors

Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串：

```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());  
System.out.println("筛选列表: " + filtered); 
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", ")); System.out.println("合并字符串: " + mergedString);
```

------

#### 统计

另外，一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);  
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();  System.out.println("列表中最大的数 : " + stats.getMax()); 
System.out.println("列表中最小的数 : " + stats.getMin()); 
System.out.println("所有数之和 : " + stats.getSum()); 
System.out.println("平均数 : " + stats.getAverage());
```

