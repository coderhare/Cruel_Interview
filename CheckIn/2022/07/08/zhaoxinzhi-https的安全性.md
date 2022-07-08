## 一、写在前面

讨论https的安全性

> 参考链接：https://mp.weixin.qq.com/s/x4iCxoWmMKyJl1sSWf26Xw

## 二、计算机网络

### 1、前言

![image-20220705145952500](/Users/bytedance/Library/Application Support/typora-user-images/image-20220705145952500.png)

本文主要从一下4个方面进行分析：

- HTTPS 为什么安全。
- **HTTPS 真的安全吗？**
- App 如何保证信息安全，不被爬走？
- 公司可能的监控手段有哪些？我们如何做才能确保自己的隐私泄露？



### 2、https

HTTPS，也称作 HTTP over TLS，TLS 前身是 SSL，会有各个版本。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZMXDhhGnYibuwwViaHhJ65clG1FYP713zJLkVupTc8Rraqg0h1SaoG58CcO5qnnreU5yYw2iapKCx3U612Jq4R91g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图描述了在TCP/IP协议栈中TLS(各子协议）和 HTTP 的关系。HTTP+TLS 也就是 HTTPS，和 HTTP 相比，HTTPS的优势：

- 数据完整性：内容传输经过完整性校验（==这个之前我都不知道==）
- 数据隐私性：内容经过对称加密，每个连接生成一个唯一的加密密钥
- 身份认证：第三方无法伪造服务端（客户端）身份



![图片](https://mmbiz.qpic.cn/mmbiz_png/ZMXDhhGnYibuwwViaHhJ65clG1FYP713zJTdSMTw23hpeZzbvIbVP0OBPAB4qYnhCBNNmrJM4qWA6ZhGOX6vTrSw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图就是大致介绍了 HTTPS 的握手流程，可以用 WireShark 抓包详细看看其中的每一个步骤，有助于理解 HTTPS 的完整流程。

因此，在通过 HTTPS 访问网站的时候，就算流量被截取监听，获取到的信息也是加密的，啥实质性的内容也看不到。

### 3、https不会加密ip地址

例如，如下图所示，当我访问某个网站，此时通过 wireshark 抓包得到的信息，能获得仅仅是一些通信的IP地址而已。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZMXDhhGnYibuwwViaHhJ65clG1FYP713zJLxhXiaeOOOFSXQV38w13nw149QGVGaQBgDzN5vqpDDb6GLXG1nqSkeA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



### 4、https不会加密域名

HTTPS真的完全加密吗？答案是否定的。

HTTPS 在握手阶段有一个很重要的东西 —— 证书。

> 简单来说，因为现在都是虚拟主机，有nginx，所以一个ip可能对应多个域名，为了进行DNS解析，客户端必须要把要访问的域名也以明文的形式传递过去。
>
> 为什么不加密呢？因为这是在**证书**协商阶段进行的，这时候还没有进行预主秘钥的协商。

#### SNI —— 域名裸奔

当访问 HTTPS 站点时，会首先与服务器建立 SSL 连接，第一步就是请求服务器的证书。

当一个 Server IP 只对应一个域名（站点）时，很方便，任意客户端请求过来，无脑返回该域名（服务）对应的证书即可。但 IP 地址（IPv4）是有限的呀，多个域名复用同一个 IP 地址的时候怎么办？

服务器在发送证书时，不知道浏览器访问的是哪个域名，所以不能根据不同域名发送不同的证书。

因此 TLS 协议升级了，多了 SNI 这个东西，SNI 即 Server Name Indication，是为了解决一个服务器使用多个域名和证书的 SSL/TLS 扩展。

现在主流客户端都支持这个协议的。别问我怎么知道这个点的，之前工作上因为这个事情还费了老大劲儿……

它的原理是：在与服务器建立 SSL 连接之前，先发送要访问站点的域名（Hostname），这样服务器会根据这个域名返回一个合适的证书。此时还没有办法进行加解密，因此至少这个域名是裸奔的。

如下图所示，上面的截图其实是访问个人博客（www.tanglei.name）的抓包情况，客户端发送握手请求时，很自觉带上了自己的域名。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZMXDhhGnYibuwwViaHhJ65clG1FYP713zJHOsJoWJtkpJticAZmBMLxv7ml5txJgchhePibDrgm24xicicIuuwnlLMZw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

因此，即便是 HTTPS，访问的域名信息也是裸奔状态。你上班期间访问小电影网站，都留下了痕迹，若接入了公司网络，就自然而然被抓个正着。

除了域名是裸奔外，其实还有更严重的风险，那就是中间人攻击。



#### 中间人攻击

> 首先要弄明白攻击者的目的，其实就是截取客户端和服务端之间通信的具体内容，并可能实施进一步破坏，比如篡改内容，或者盗取密码等敏感信息等等。（而不是简单的尝试和服务端建立通信然后搞破坏！！这样的话他也不叫攻击者了，而是合法的。）
>
> 所以在想学习一个技术或者一种手段的时候，思路应该是：先弄清楚出现的原因（目的是什么），再去思考解决方法，然后看业界的解决方法，然后思考这样做解决了什么问题，会有什么缺陷，等等。
>
> 比如这个中间人攻击，直接看文章的话，可能以为是”攻击者目标是想要攻击服务器“（因为看到攻击嘛自然而然想得到这个常见的问题），这直接就走远了，这样越看后面的文章会越迷糊，因为”攻击者目标是截取客户端和服务端通信的内容“。所以要在看文章细节之前先问自己这几个问题，弄明白了之后再去看解决办法啥的，继续看文章，这样带着问题阅读，才是高效的！！不然看完文章，嗯他说的很有道理，但是其实连【问题的定义】都没搞清楚，印象自然是不深的。



> Q1: 对于中间人攻击，我有个疑问，既然目的是想截取通信内容，为什么要伪造证书，让客户端识别出来中间被篡改了？直接还是用相同的证书不行吗？相当于拦截了一份相同的数据而已，这次拦截对客户端服务端的交互是透明的。
>
> A1: 因为你需要私钥啊，只有私钥才能获得预主秘钥是什么，从而解密后面的同信。没有私钥的话，你就得不到预主秘钥是啥，更别提后面的解密了。而你如果想纯透明传输，那么你拦截了这份数据，但是没法解密，因为私钥只在服务端那里存着！！所以纯透明传输是达不到攻击者的目的。



前面也提到 HTTPS 中的关键其实在于这个证书。从名字可以看出来，中间人攻击就是在客户端、服务器之间多了个『中介』，『中介』在客户端、服务器双方中伪装对方，如下图所示，这个『MitmProxy』充当了中间人，互相欺骗：

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZMXDhhGnYibuwwViaHhJ65clG1FYP713zJNUTnL3xyMicfxGQMj85LxzeOicS5uAI8iavDm8n3qm4QM78P6EkOibWUGw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)中间人攻击，来源 evil0x

可以安装 MitmProxy 或者 Fiddler 之类的抓包软件尝试一把，然后开启代理。

此时用手机访问百度，得到的信息如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZMXDhhGnYibuwwViaHhJ65clG1FYP713zJsPKj82NUdPrcroyQmNLfVNgUBrY0NdZsiaQic536RUydrnUheiamPocOw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

证书信任前，提示，连接不是私密连接，其实就是浏览器识别了证书不太对劲，没有信任。而如果此时手机安装了 Fiddler 的证书，就会正常访问。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZMXDhhGnYibuwwViaHhJ65clG1FYP713zJlOcg9njZculVpYyed9BKyc5hA8hcDskSmos03xjpeXgItf6zNlwpEA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

证书信任后可正常访问

因此，当你信任证书后，在中间人面前，又是一览无余了。

而如果你用了公司电脑，估计你有相应的操作让信任证书吧，或者手机上是否有安装类似的客户端软件吧？

抓紧时间看看手机的证书安装明细（比如我手机上的）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZMXDhhGnYibuwwViaHhJ65clG1FYP713zJdpQNng2XibmIQIrxKALTKckic1PRjKeay1d9HaNq6Tn595pp5HTA21Xg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我前任公司在信息安全这块做得就非常谨慎，手机会有工作手机，未授权的任何 App 都不能安装，谁知道 App 会悄悄干些什么事情呢。（最新热点，QQ扫描浏览器历史记录，你可知道）

当然各种 App 肯定也不是吃素的，不会让『中间人攻击』这么容易就得逞的，咱们接着看。



### 5、如何防止信息安全，反爬

前面提到，要实施中间人攻击，关键在于证书是否得到信任。浏览器的行为是证书可以让用户授权是否信任，而 APP 就可以开发者自己控制。

比如我尝试通过类似的方式对某匿名社区进行抓包解密 HTTPS，但最终失败了，为什么呢？

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZMXDhhGnYibuwwViaHhJ65clG1FYP713zJhQg80srO3qOlibBI8h6EUY8xCGvKbQ4WUYaot323A7iccrVN69iavAibzg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这就要谈到『SSL Pinning』技术。

**App 可以自己检验 SSL 握手时服务端返回的证书是否合法**，“SSL pinning” 技术说的就是在 App 中只信任固定的证书或者公钥。

因为在握手阶段服务端的证书必须返回给客户端，如果客户端**在打包的时候**，就把服务端证书放到本地，在握手校验证书的环节进行比较，服务端返回的证书和本地内置的证书一模一样，才发起网络请求。否则，直接断开连接，不可用。

当然，一般情况下，用这种技术也就能防止 HTTPS 信息被解密了。

不过，也还有其他的技术能够破解这种方法，比如 Android 下的一些 Hook 技术，具体而言就是绕过本地证书强校验的逻辑。感兴趣的同学可以抱着学习目的研究一下。不过据说这种方式需要对系统进行 Root、越狱等，需要一些更高权限的设置。

因此，也告诫我们，一定不要乱安装一些软件，稍不注意可能就中招，让自己在互联网上进行裸奔。一方面个人隐私信息等泄露，另外一个方面可能一些非常重要的如账户密码等也可能被窃取。



### 6、可能的监控手段有哪些？

办公电脑当然要接入公司网络，通过上面介绍的内容，你也应该知道，你在什么时候浏览了哪些网站，公司其实都是一清二楚的。

若自己的手机如果**接入了公司网络**也是一模一样（连 Agent 软件都不需要装）。这就提醒我们，**私人上网尽量用自己的移动网络**呀。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZMXDhhGnYibuwwViaHhJ65clG1FYP713zJBr4cbh57RgpibJfeNvdvlj96Hiah4xtuJSoibWJLzYhIEtP9VTYkqN57A/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)浏览记录，来源知乎

上面提到，如一些涉及隐私的敏感信息，如一些 PC 软件、手机 App 自己内部加密传输的话，内容加密（包括但不限于 HTTPS）不被破解也问题不大。

不过，这当然依赖这些软件设计者的水平了。比如同一个匿名用户对外展示的 ID 不能相同，如果是同一个的话也恰好暴露了逻辑漏洞。

==这里是不是写的有问题，应该是匿名用户对外展示的id必须相同？==

当然，我们还是不要抱有侥幸心理，在监管的要求下，如果确实有一些违法等不恰当的言论等，始终还是有门路找到你的。



更何况，一般办公电脑都会预安装一些公司安全软件，至于这些软件究竟都干了些什么，有没有进行传说中悄悄截图什么的，这就因人（公司）而异了。（不讨论类似行为是否涉及到侵犯了员工隐私等问题）

![图片](https://mmbiz.qpic.cn/mmbiz_png/ZMXDhhGnYibuwwViaHhJ65clG1FYP713zJJHYys5VYLtW5yt1M9L1WSb82XcbGm4nbohFVxiasUspicGORMo5RWyFg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)图源知乎

不过，个人认为，咱也没必要**过度担心**。一般公司也不会因为你上班偶尔摸个鱼，逛逛淘宝、看看微博来找你麻烦的。毕竟没必要这么点芝麻事情来『大动干戈』。

但最好是不是对照员工手册来看看，是否有明令禁止的行为？自己的行为是不是太过了，免得被抓住把柄，正所谓『常在河边走哪有不湿鞋』，『欲加之罪、何患无辞』。
