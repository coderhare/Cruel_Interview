## 一、写在前面

今天了解了下关于DDNS服务和花生壳等软件，主要是了解了DDNS用于什么场景下。

（其实就是提供了一种可以动态解析域名的能力，即用户无需感知ip的变化，只需要通过域名就可以解析成动态的ip）

> 参考链接：https://baike.baidu.com/item/ddns/670146?fr=aladdin

## 二、DDNS

### 1、DDNS是什么

DDNS（Dynamic Domain Name Server，动态域名服务）是将用户的动态IP地址映射到一个固定的域名解析服务上，用户每次连接网络的时候客户端程序就会通过信息传递把该主机的动态IP地址传送给位于服务商主机上的服务器程序，服务器程序负责提供DNS服务并实现动态域名解析。

### 2、和DNS的区别

动态域名解析（Dynamic DNS，简称DDNS）是把[互联网域名指向可变IP地址的系统。DNS只是提供了域名和IP地址之间的静态对应关系，当IP地址发生变化时，DNS无法动态的更新域名和IP地址之间的对应关系，从而导致访问失败。（注意这里主要不是指在阿里云公网ip搭建web服务的那种DNS解析，因为公网ip是不变的）

但是DDNS系统是将用户的动态IP地址映射到一个固定的域名解析服务上，用户每次连接网络时，客户端程序通过信息传递把该主机的动态IP地址传送给位于服务商主机上的服务器程序，实现动态域名解析。（注意这里用户的IP地址不是指的客户端的，其实也是指需要DNS解析的服务端的ip！即针对的的情况是针对拨号上网或者用花生壳等工具想把自己的网域(比如自己笔记本部署web服务想让外网访问)公布在互联网上的时候，而不是阿里云那种固定的公网ip）

### 3、为什么要DDNS？具体怎么使用？

#### 3.1 为什么要使用DDNS？

目前大部分家庭使用PPPOE拨号方式上网，每次上网获得的IP都是随机变换的，但是家里的网络监控、智能设备需要通过网络访问，每次使用都需要先知道IP非常麻烦。
有了DDNS动态域名解析，我们只要到花生壳之类的域名提供商申请一个动态域名，以后只要记住域名就可以访问家里的网络，非常方便。

#### 3.2 怎么使用？

> 即注册后用路由登录就可以用了。可以用域名访问路由或设置跳转至自己的主页。

在域名提供商处申请一个域名（会提供一个账号和密码），在路由器的动态DDNS里登陆就可以获得这个域名的解析，以后只要记住这个域名就能访问这个家庭的网络了。
动态DDNS需要与路由器的规则转发配合使用，如家里建立一个WEB站点需要使用80端口，但是动态DDNS只能把域名解析到这个网络，不能指定到哪个机器，这个是有就需要规则转发里的虚拟服务器，将80端口映射到指定机器的80端口，这样就可以通过这个域名访问WEB站点了。

### 4、原理

DDNS用来动态更新DNS服务器上域名和IP地址之间的对应关系，从而保证通过域名访问到正确的IP地址。很多机构都提供了DDNS服务，在后台运行并且每隔数分钟来检查电脑的IP地址，如果IP发生变更，就会向DNS服务器发送更新IP地址的请求。

### 5、作用

1、ISP大多提供动态IP（如拨号上网），我们若想在网际网络上以自己的网域公布，DDNS提供了解决方案，它可以自动更新用户每次变化的浮动IP，然后将其与网域相对应，这样其他上网用户就可以透过网域来交流了。
2、DDNS可以让**我们在自己的或家里架设WEB**\MAIL\FTP等服务器，而不用花钱去付虚拟主机租金。(前提是你可以承受ADSL上传的速率)
3、**主机是自己的**，空间可根据自己的需求来扩充，维护也比较方便。有了网域与空间架设网站，FTP 服务器、EMAIL服务器都不成问题。
4、如果有对VPN的需求，有了DDNS就可以用普通上网方式方便地建立Tunnel。透过网域的方式连结，实现远端管理、远端存取、远端打印等功能。（即比如访问`http://mynetcenter.com:7776`先通过DDNS解析到本地网域，然后再通过控制访问的端口来做转发和映射到具体的网域中的机器和服务？比如7776就可以转发到家里的打印机上去打印。这样在公司我也可以控制家里的打印机了）



### 6、使用场景和方案

ISP大多为我们提供动态IP（如ADSL拨号上网），而很多网络视频服务器和网络摄像机通过远程访问时需要一个固定的IP，而固定IP的费用很难让客户接受。所以DDNS为大家提出了一种全新的解决方案，它可以捕获用户每次变化的IP，然后将其与域名相对应，这样客户就可以通过域名来进行远程监控了。DDNS的解决方案有如下几种：

> 其实不管是路由器还是运行ddns客户端，都是当一个网关用（一个网络中可以有多个网关），先传到网关，然后再靠端口映射做转发和分流。

#### 5.1 路由器外挂

路由器外挂就是采用集成DDNS的路由器，通过申请其域名和服务，把申请所得用户名密码填入路由器DDNS模块相关项，再由路由器上作端口映射指向所需访问的监控设备即可，远程监控端通过访问域名即可访问到当前路由器，根据不同的端口来判断并指向所需访问的监控设备。

![image-20220515170513958](/Users/bytedance/Library/Application Support/typora-user-images/image-20220515170513958.png)

> 服务提供商有好多，先去申请域名和一组用户名密码，然后登陆成功后，外网就可以通过访问你这个域名来访问到你这个路由器的网域了。（当然前提路由器是可连接到因特网的！！即相当于是在保证路由器可联网的情况下，提供了一种外部访问到路由器内部网域的方式）
>
> ![image-20220515171944948](/Users/bytedance/Library/Application Support/typora-user-images/image-20220515171944948.png)

#### 6.2 集成DDNS的监控设备

对于无人值守或不方便外挂路由器的状况下，视频监控也可采用集成DDNS的网络摄像机，同样把申请DDNS服务得到的用户名密码填入相关项，通过一条ADSL等宽带线路直接相连。远程监控端通过域名直接访问。

#### 6.3 运行DDNS客户端软件报文交互方式

在局域网内部的任一PC或服务器上运行到DDNS客户端，此时域名解析到的IP地址是局域网网关出口处的公网IP地址，再在网关处作端口映射指向监控设备即可。 

### 7、组网原理

DDNS的典型组网环境如下图所示，DDNS采用客户端/服务器模式。

#### 7.1 DDNS客户端

（客户端收敛在用户的路由器这边）

DDNS客户端需要动态更新域名和IP地址对应关系的设备。Internet用户通常通过域名访问提供应用层服务的服务器，如HTTP、FTP服务器。为了保证IP地址变化时，仍然可以通过域名访问这些服务器，当服务器的IP地址发生变化时，它们将作为DDNS客户端，向DDNS服务器发送更新域名和IP地址对应关系的DDNS更新请求。

#### 7.2 DDNS服务器

（服务端收敛在花生壳等服务提供商那里）

DDNS服务器负责通知DNS服务器动态更新域名和IP地址之间的对应关系。

接收到DDNS客户端的更新请求后，DDNS服务器通知DNS服务器重新建立域名和IP地址之间的对应关系。从而保证即使DDNS客户端的IP地址改变，Internet用户仍然可以通过同样的域名访问DDNS客户端。

DDNS客户端向 DDNS服务器发送TCP连接请求，如果连接建立成功，则向DDNS服务器发送 DDNS更新请求，并统计发送 DDNS 更新请求报文的次数；
DDNS服务器收到DDNS 客户端发送过来的DDNS更新请求后，通知DDNS服务器进行域名更新，并且向 DDNS客户端发送应答报文。 



![图1 DDNS组网原理](https://bkimg.cdn.bcebos.com/pic/9345d688d43f879460c6716ed21b0ef41bd53ab0?x-bce-process=image/resize,m_lfit,w_440,limit_1/format,f_auto)