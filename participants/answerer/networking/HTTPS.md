# TLS详解

> 主要参考 https://mp.weixin.qq.com/s/U9SRLE7jZTB6lUZ6c8gTKg 

HTTP是明文传输，截取通信内容相当容易，没有加密方式，信息容易泄露。于是发展出了HTTPS

### TLS握手

![图解](https://mmbiz.qpic.cn/mmbiz_jpg/J0g14CUwaZeMSNGtbYLcgAkWmcscQz2FkDKoDib57FOWVb8zZyXf65GEme6ibHkPTVxOmazHqicLDicX7iacFFMt22A/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

HTTPS在HTTP与TCP之间加入了TLS来解决「窃听」，「篡改」，「冒充」的风险问题

TLS协议如何解决HTTP风险：

1. **信息加密**：HTTP交互信息是被加密的，第三方无法被窃取

2. **校验机制**：校验信息传输过程中是否有被第三方篡改过，如果百日篡改过，就会有警告提示

3. **身份证书**（CA证书）

可见，有了 TLS 协议，能保证 HTTP 通信是安全的了，那么在进行 HTTP 通信前，需要先进行 TLS 握手。TLS 的握手过程，如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeMSNGtbYLcgAkWmcscQz2FpC63OTRvfL58f1ia8BMeWkV4JiaWP3H7icHXRNzRicOcAgksdHEKhfghDA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

上图简要概述来 TLS 的握手过程，其中每一个「框」都是一个记录（*record*），记录是 TLS 收发数据的基本单位，类似于 TCP 里的 segment。多个记录可以组合成一个 TCP 包发送，所以**通常经过「四个消息」就可以完成 TLS 握手，也就是需要 2个 RTT 的时延**，然后就可以在安全的通信环境里发送 HTTP 报文，实现 HTTPS 协议。

所以可以发现，HTTPS 是应用层协议，需要先完成 TCP 连接建立，然后走 TLS 握手过程后，才能建立通信安全的连接。

事实上，不同的密钥交换算法，TLS 的握手过程可能会有一些区别。

### RSA握手过程

传统的TLS握手基本都是使用RSA算法来实现密钥交换的，在将TLS证书部署于服务端时，证书文件内包含一堆公私钥，其中公钥会在TLS握手阶段传递给客户端，私钥则一直留在服务端，一定要确保私钥不能被窃取。

在RSA密钥协商算法中，客户端会生成随机密钥，并使用服务端的公钥加密后再传给服务端。根据非对称加密算法，公钥加密的消息技能通过私钥解密，这样服务员解密后，双方就得到了相同的密钥，再用它加密应用消息。

> 在现实应用场景下，由于每次发送信息都采用非对称加密方式，会导致效率受到极大影响，因此，HTTPS采用混合加密的手段，利用非对称加密来确保密钥是安全的，然后就利用密钥进行对称加密。

#### TLS第一次握手

客户端首先会发送「Cient Hello」消息，与服务器打招呼

![](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeMSNGtbYLcgAkWmcscQz2FCOMAhxqC7oqlIvuPv4Ey5RTZJ4EZBEib53BDrqG9ibZGMz47eiaIc1lAg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

消息里头有客户端使用的TLS版本号，支持的密码套件列表，以及生成的随机数，这个随机数会被服务端保留，他是生成堆成加密密钥的材料之一。

#### TLS第二次握手

当服务端收到客户端的「Client Hello」消息后，会确认TLS版本号是否支持，和才能够密码套件列表中选择一个密码套件，以及生成随机数。
