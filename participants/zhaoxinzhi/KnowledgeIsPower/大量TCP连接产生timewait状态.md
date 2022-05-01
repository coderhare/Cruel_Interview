## 一、写在前面

大量的TIME_WAIT 状态 TCP连接对业务有什么影响怎么处理。

有空也可以整理下CLOSE_WAIT的可能原因。

## 二、TCP

先把结论提上来讲：

> **结论**：
>
> 1、 **time_wait 状态的影响**：
>
> TCP 连接中，「主动发起关闭连接」的一端，会进入 time_wait 状态time_wait 状态，默认会持续 2 MSL（报文的最大生存时间），一般是 2x2 minstime_wait 状态下，TCP 连接占用的端口，无法被再次使用TCP 端口数量，上限是 6.5w（65535，16 bit）大量 time_wait 状态存在，会导致新建 TCP 连接会出错，**address already in use : connect** 异常
>
> 2、 **现实场景**：
>
> 服务器端，一般设置：**不允许**「主动关闭连接」但 HTTP 请求中，http 头部 connection 参数，可能设置为 close，则，服务端处理完请求会主动关闭 TCP 连接现在浏览器中， HTTP 请求 connection 参数，一般都设置为 keep-alive。Nginx 反向代理场景中，可能出现大量短链接，服务器端，可能存在
>
> 3、 **解决办法：服务器端**，
>
> 允许 time_wait 状态的 socket 被重用缩减 time_wait 时间，设置为 1 MSL（即，2 mins）



### 1、问题描述：什么现象？什么影响？

**TIME_WAIT状态：**

TCP 连接中，**主动关闭连接**的一方出现的状态；（收到 FIN 命令，进入 TIME_WAIT 状态，并返回 ACK 命令）保持 2 个 MSL 时间，即，4 分钟；（MSL 为 2 分钟）

![image-20220502003502475](D:\mystudy\internship\Cruel_Interview\participants\zhaoxinzhi\assets\大量TCP连接产生timewait状态\image-20220502003502475.png)



模拟高并发的场景，会出现批量的TIME_WAIT的 TCP 连接：

短时间后，所有的TIME_WAIT全都消失，被回收，端口包括服务，均正常。

即，在高并发的场景下，一部分 TIME_WAIT 连接被回收，但新的 TIME_WAIT 连接产生。一些极端情况下，会出现**大量**的 TIME_WAIT 连接。



> Q：上述大量的 TIME_WAIT状态 TCP 连接，有什么业务上的影响吗？
>
> A：在 TIME_WAIT 状态时，两端的端口不能使用，要等到2MSL时间结束，才可继续使用。（IP 层）
>
> 当连接处于2MSL等待阶段时，任何迟到的报文段都将被丢弃。
>
> > 不过在实际应用中，可以通过设置 「**SO_REUSEADDR选项**」，达到不必等待2MSL时间结束，即可使用被占用的端口。



Nginx 作为反向代理时，大量的短链接，可能导致 Nginx 上的 TCP 连接处于time_wait

状态。

每一个 time_wait 状态，都会占用一个「本地端口」，上限为 65535；当大量的连接处于 time_wait 时，新建立 TCP 连接会出错，**address already in use : connect** 异常



> TCP端口上限65535：
>
> TCP 本地端口数量，上限为 65535（6.5w），这是因为 TCP 头部使用 16 bit，存储「**端口号**」，因此约束上限为 65535。
>
> 统计 TCP 连接的状态：
>
> 1. `netstat -nat |grep TIME_WAIT`
> 2. `netstat -nat | grep -E "TIME_WAIT|Local Address"`
> 3. `$ netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'`
>
> ![image-20220502003918782](D:\mystudy\internship\Cruel_Interview\participants\zhaoxinzhi\assets\大量TCP连接产生timewait状态\image-20220502003918782.png)

### 2、问题分析

大量的TIME_WAIT状态 TCP 连接存在，其本质原因是什么？

大量的**短连接**存在。特别是 HTTP 请求中，如果 connection 头部取值被设置为 close 时，基本都由「**服务端**」发起**主动关闭连接**而，TCP 四次挥手关闭连接机制中，为了保证 ACK 重发和丢弃延迟数据，设置 time_wait 为 2 倍的 MSL（报文最大存活时间）



### 3、解决方案

1、**客户端**，HTTP 请求的头部，connection 设置为 keep-alive，保持存活一段时间：现在的浏览器，一般都这么进行了 

2、**服务器端**，允许 time_wait 状态的 socket 被**重用**缩减 time_wait 时间，设置为 1 MSL（即，2 mins）

> 更多细节，参考：https://www.cnblogs.com/yjf512/p/5327886.html

### 



### 4、Q&A

1、**time_wait 是「服务器端」的状态？or 「客户端」的状态？**

RE：time_wait 是「主动关闭 TCP 连接」一方的状态，可能是「客服端」的，也可能是「服务器端」的一般情况下，都是「客户端」所处的状态；「服务器端」一般设置「不主动关闭连接」

2、**服务器在对外服务时，是「客户端」发起的断开连接？还是「服务器」发起的断开连接？**

正常情况下，都是「客户端」发起的断开连接「服务器」一般设置为「不主动关闭连接」，服务器通常执行「被动关闭」但 HTTP 请求中，http 头部 connection 参数，可能设置为 close，则，服务端处理完请求会主动关闭 TCP 连接关于 Apache httpd 服务器的关联配置，参考：https://elf8848.iteye.com/blog/1739571

3、**关于 HTTP 请求中，设置的主动关闭 TCP 连接的机制：TIME_WAIT的是主动断开方才会出现的，所以主动断开方是服务端？**

答案是是的。在HTTP1.1协议中，有个 Connection 头，Connection有两个值，close和keep-alive，这个头就相当于客户端告诉服务端，服务端你执行完成请求之后，是关闭连接还是保持连接，保持连接就意味着在保持连接期间，只能由客户端主动断开连接。还有一个keep-alive的头，设置的值就代表了服务端保持连接保持多久。HTTP默认的Connection值为close，那么就意味着关闭请求的一方几乎都会是由服务端这边发起的。那么这个服务端产生TIME_WAIT过多的情况就很正常了。虽然HTTP默认Connection值为close，但是，现在的浏览器发送请求的时候一般都会设置Connection为keep-alive了。所以，也有人说，现在没有必要通过调整参数来使TIME_WAIT降低了。

4、**time_wait 状态**存在的必要性？（老生常谈了。。两方面原因）

**可靠的实现 TCP 全双工连接的终止**：四次挥手关闭 TCP 连接过程中，最后的 ACK 是由「主动关闭连接」的一端发出的，如果这个 ACK 丢失，则，对方会重发 FIN 请求，因此，在「主动关闭连接」的一段，需要维护一个 time_wait 状态，处理对方重发的 FIN 请求；**处理延迟到达的报文**：由于路由器可能抖动，TCP 报文会延迟到达，为了避免「延迟到达的 TCP 报文」被误认为是「新 TCP 连接」的数据，则，需要在允许新创建 TCP 连接之前，保持一个不可用的状态，等待所有延迟报文的消失，一般设置为 2 倍的 MSL（报文的最大生存时间），解决「延迟达到的 TCP 报文」问题；
