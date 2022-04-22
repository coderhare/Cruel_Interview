# TLS详解

> 主要参考 https://mp.weixin.qq.com/s/U9SRLE7jZTB6lUZ6c8gTKg 

HTTP是明文传输，截取通信内容相当容易，没有加密方式，信息容易泄露。于是发展出了HTTPS

### TLS握手

![图解](https://mmbiz.qpic.cn/mmbiz_jpg/J0g14CUwaZeMSNGtbYLcgAkWmcscQz2FkDKoDib57FOWVb8zZyXf65GEme6ibHkPTVxOmazHqicLDicX7iacFFMt22A/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

HTTPS在HTTP与TCP之间加入了TLS来解决「窃听」，「篡改」，「冒充」的风险问题

TLS协议如何解决HTTP风险：

1. **信息加密**：HTTP交互信息是被加密的，第三方无法被窃取

2. **校验机制**：校验信息传输过程中是否有被第三方篡改过，如果篡改过，就会有警告提示

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

消息里头有客户端使用的TLS版本号，支持的密码套件列表，以及生成的随机数，这个随机数会被服务端保留，他是生成对称加密密钥的材料之一。

#### TLS第二次握手

当服务端收到客户端的「Client Hello」消息后，会确认TLS版本号是否支持，和才能够密码套件列表中选择一个密码套件，以及生成随机数。

接着，返回「Server Hello」消息，消息里面有服务器确认的TLS版本号，也给出了随机数，然后从客户端的密码套件列表选择了一个合适的密码套件。![](/Users/wocaibujiaoquanmei/Library/Application%20Support/marktext/images/2022-04-20-11-44-51-image.png)



可以看到，服务端选择的密码套件是`Cipher Suite: TLS_RSA_WITH_AES_128_GCM_SHA256`。

密码套件的基本形式是：**密钥交换算法 + 签名算法 + 对称加密算法 + 摘要算法**，一般WITH单词前面有两个单词，第一个单词是约定密钥交换的算法，第二个单词是约定证书的验证算法。

比如说刚才的密码套件的意思是：

- 由于WITH单词只有一个RSA，则说明握手时密钥交换算法和签名算法都是使用RSA

- 握手后的通信使用AES对称算法，密钥长度128位，分组模式是GCM

- 摘要算法SHA256用于消息认证和产生随机数

在「**Client Hello**」与「**Server Hello**」后，客户端和服务端确认了TLS版本和使用的密码套件，并且客户端和服务端各自生成了一个随机数，并将随机数传递给了对方，这两个随机数是后续作为生成「会话密钥」的条件，所谓的会话密钥就是数据传输时，所使用的对称加密密钥

然后，服务端为了证明自己的身份，会发送「**server certificate**」给客户端，这个消息里含有数字证书

![](/Users/wocaibujiaoquanmei/Library/Application%20Support/marktext/images/2022-04-20-14-33-45-image.png)

随后，服务端发送「**server hello done**」消息，目的是告诉客户端，本次打招呼完成。





#### 客户端拿到了服务端的数字证书之后，如何校验数字证书的真实性？

##### 数字证书包含：

1. 公钥

2. 持有者信息

3. 证书认证机构（CA）的信息

4. CA对这份文件的数字签名及使用的算法

5. 证书有效期

6. 其他的额外信息





### TLS第三次握手

客户端验证完证书之后，认为可信则继续往下走。接着，客户端就会生成一个新的随机数，用服务器的RSA公钥加密该随机数，通过「**Change Ciper Key Exchange**」消息传给服务端。

![](/Users/wocaibujiaoquanmei/Library/Application%20Support/marktext/images/2022-04-20-14-56-09-image.png)

服务端收到后，用RSA私钥解密，得到客户端发来的随机数（**Pre-master**)。

至此，客户端和服务端双方都共享了三个随机数，分别是**Client Random， Server Random，pre-master**

于是双方根据已经得到的三个随机数，生成会话密钥，他是对称密钥，用于对后序的HTTP请求/响应的数据加解密。

生成完会话密钥之后，客户端发送一个「**Change Cipher Spec**」，告诉服务端开始使用加密方式发送消息。

![](/Users/wocaibujiaoquanmei/Library/Application%20Support/marktext/images/2022-04-20-15-07-10-image.png)

然后·，客户端再发送一个「**Encrypted Handshake Message（Finished）**」消息，把之前所有发送的消息做个摘要，再用会话密钥加密一下，让服务器做验证，验证加密通信是否可用以及之前握手信息是否有被中途篡改过。

![](/Users/wocaibujiaoquanmei/Library/Application%20Support/marktext/images/2022-04-20-15-10-56-image.png)

可以发现，「**Change Cipher Spec**」之前传输的TLS握手数据都是明文，之后都是对称密钥加密的密文。



#### TLS第四次握手

服务器是同样的操作，发「**Change Cipher Spec**」和「**Encrypted Handshake Message**」，如果双方都验证加密和解密没问题，那么握手正式完成。

最后，就用「会话密钥」加密解密HTTP请求和响应。





### 使用RSA算法的缺陷：

不支持前向保密。因为客户端传递随机数（用于生成对称加密密钥的条件之一）给服务端时使用的是公钥加密的，服务端收到后，会用私钥解密得到随机数。所以一旦服务端的私钥泄露了，过去被第三方截获的所有TLS通讯密文都会被破解。
