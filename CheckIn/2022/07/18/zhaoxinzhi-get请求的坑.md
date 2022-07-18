## 一、写在前面

关于springboot下载文件设置filename的坑

> 参考链接1：https://blog.csdn.net/laihongfeng/article/details/123116826
>
> 参考链接2：https://blog.csdn.net/happyqwz/article/details/78595999
>
> 详细解释原理：https://www.pudn.com/news/628f8346bf399b7f351e7d98.html



## 二、坑

### 1、get请求的参数问题

#### 问题：

springboot接口使用GET类型传递一个文件名，文件名比如是`22151118+zxz`，文件名作为url的一部分传递到后端。但是后端拿到的是`22151118 zxz`，这肯定是不符合预期的。原因是因为，url里的加号会被当成空格！！（本质是URLDecoder的decode方法会把加号+和%20都解码为空格）

正确做法是现将加号转义成%2B！！就跟中文被转化成Unicode编码一样。

ps：其实用post类型的请求就没有这个问题了

问题原因虽然是找到了，但是对这个问题还是想刨根问底一下，为什么GET就会导致+转成了空格，并且如果必须用get类型的话，应该怎么解决这个问题。

#### 解决办法：

在调用URLEncoder.encode对URL进行编码后(所有加号+已被编码成%2B)，再调用replaceAll(“\\+”, “%20″)，将所有加号+替换为%20。

即：

```java
urlStr = URLEncoder.encode(filename, "UTF-8").replaceAll("\\+", "%20");
```

注意这种使用，对原有+是不影响的，因为原有+已经在encode部分变成%2B了。



### 2、下载文件并返回的filename问题

> 参考链接：https://icode.best/i/16728645516182

浏览器下载的文件名是根据`Content-Disposition`这个header值确定的，所以只要确保这个值是正确的就好了。（注意下需要保证二进制是符合预期的就可以，因为http传输的时候是不支持中文的，所以只需要保证二进制是正确的。）

如果这么写的话：

```java
response.setHeader("Content-Disposition", "attachment; filename=" + new String(filename.getBytes("UTF-8"),"ISO8859-1"));
```

很多情况下，我们在写程序的时候都会把代码设置为UTF-8的编码，可以在下载文件设置filename的时候却有违常理，竟然设置编码格式为ISO8859-1，代码如下（如是英文的话就不需要这样处理了）

```java
response.setHeader("Content-disposition", "attachment; filename=" + new String("中文文件名".getBytes("utf-8"), "ISO8859-1")); 
```

 提取出来最核心的一点，filename=new String("中文文件名".getBytes("utf-8"), "ISO8859-1");

### 3、为什么呢

先说为什么使用 ISO8859-1 编码，这个主要是由于http协议，http的header头要求其内容必须为iso8859-1编码，所以我们最终要把其编码为 ISO8859-1 编码的字符串；

但是前面为什么不直接使用 "中文文件名".getBytes("ISO8859-1"); 这样的代码呢？

因为ISO8859-1编码的编码表中，根本就没有包含汉字字符，当然也就无法通过"中文文件名".getBytes("ISO8859-1");来得到正确的“中文文件名”在ISO8859-1中的编码值了，所以再通过new String()来还原就无从谈起了。 所以先通过 "中文文件名".getBytes("utf-8") 获取其 byte[] 字节，让其按照字节来编码，即在使用 new String("中文文件名".getBytes("utf-8"), "ISO8859-1") 将其重新组成一个字符串，传送给浏览器。然后浏览器解析的时候，再使用`new String(s_iso88591.getBytes("ISO8859-1"),"UTF-8")`来得到正确的中文汉字。

> 详细链接见：https://www.pudn.com/news/628f8346bf399b7f351e7d98.html

### 3、总结

1、最好还是直接传英文做文件名吧。

2、尽量少用get类型请求。

### 4、其他常用的百分比编码

％5B和％5D分别是 [ 和 ] 

