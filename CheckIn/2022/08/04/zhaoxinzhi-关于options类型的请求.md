## 一、写在前面

什么是options请求？为什么会有options请求？

> 参考链接：https://blog.csdn.net/gwdgwd123/article/details/100554117

## 二、http

### 1、什么是options？

options请求是用于请求服务器对于某些接口等资源的支持情况的，包括各种请求方法、头部的支持情况，仅作查询使用。

### 2、浏览器级行为

这个概念听着有点耳生，嗯是我自己这么说的。。。我们可以把浏览器自主发起的行为称之为“浏览器级行为”。之所以说options是一种浏览器级行为，是因为在某些情况下，普通的get或者post请求回首先自动发起一次options请求，当options请求成功返回后，真正的ajax请求才会再次发起。

再来看下这个“某些情况下”都是什么情况？

1、跨域请求，非跨域请求不会出现options请求
2、自定义请求头
3、请求头中的content-type是application/x-www-form-urlencoded，multipart/form-data，text/plain之外的格式

当满足条件12或者13的时候，简单的ajax请求就会出现options请求，有没有感觉到一点同源策略的意思，个人理解这个就是浏览器底层对于同源策略的一个具体实现。首先得到服务器端的确认，才能继续下一步的操作，这也是为什么options请求也被叫做“预检”请求的原因吧。

### 3、出现之后怎么处理？服务端怎么响应这个？

这个基本思路就是server端在接收到请求的时候，先去判断下是不是options请求，判断下来源，没问题的时候返回个200之类的成功就可以了。

header里面包含自定义字段，浏览器是会先发一次options请求，如果请求通过，则继续发送正式的post请求，而如果不通过则返回以上错误。

比如：

```java
  // TODO 支持跨域访问        
response.setHeader("Access-Control-Allow-Origin", "*");
response.setHeader("Access-Control-Allow-Credentials", "true");
response.setHeader("Access-Control-Allow-Methods", "*");
response.setHeader("Access-Control-Allow-Headers", "Content-Type,Access-Token");
response.setHeader("Access-Control-Expose-Headers", "*");        
if (request.getMethod().equals("OPTIONS")) {
  HttpUtil.setResponse(response, HttpStatus.OK.value(), null);            
  return;
}
```




上面代码需要加入允许的头部，content-type和access-token，并且判断请求的方法是options的时候，返回ok（200）给客户端，这样才能继续发正式的post请求。
