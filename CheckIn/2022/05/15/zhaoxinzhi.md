## 一、写在前面

SpringMVC拦截器-HandlerInterceptorAdapter

> 参考链接：https://blog.csdn.net/Suubyy/article/details/83340962

## 二、Spring

### 1、拦截器的作用

HandlerInterceptorAdapter是SpringMVC中的拦截器，它是用于拦截URL请求的，主要是为了请求的预处理和后续处理。（应该是进Controller之前的？）

> 关于拦截器，过滤器，处理器的区别：
>
> 参考链接：https://blog.csdn.net/shisannnn/article/details/119990205

### 2、使用

我们只需要自定义一个“拦截器”去继承HandlerInterceptorAdapter这个抽象类就可以了，这个类提供了三个方法，我们只需要根据自己的业务需求来覆写这个三个方法就可以了

```java
@Component("CustomerInterceptor ")
public class CustomerInterceptor extends HandlerInterceptorAdapter{

	//这方法是在请求未到达Conterller之前进行预处理，
	//如果返回false,就不会继续调用后续的程序，我们不需要利用response来产生响应
	//如果返回true则继续执行后续程序，例如下一个拦截器
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		//处理你自己的业务逻辑
		return true;
	}
	
	//这个方法的执行顺序是请求对应Controller执行完毕之后，但是页面为渲染之前
	//我们可以利用modelAndView来对页面进行额外的处理
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
		//处理你自己的业务逻辑
	}
	
	//这个方法的执行顺序是Controller执行完毕并且页面渲染之后的回调方法
	//类似于try catch finally 里的finally代码块
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
		//处理你自己的业务逻辑
	}
	
	public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		//处理你自己的业务逻辑
	}
}

```



```xml
<!--配置拦截器, 多个拦截器,顺序执行 -->  
<mvc:interceptors>    
    <mvc:interceptor>    
        <!-- 匹配的是url路径， 如果不配置或/**,将拦截所有的Controller -->  
        <mvc:mapping path="/" />  
        <mvc:mapping path="/**" />  
        <!--这个标签标识不会拦截 /login/*的所有请求-->
        <mvn:excloud-mapping="/login/*">
        <bean ref="CustomerInterceptor "/>    
    </mvc:interceptor>  
    <!-- 当设置多个拦截器时，先按顺序调用preHandle方法，然后逆序调用每个拦截器的postHandle和afterCompletion方法 -->  
</mvc:interceptors> 

```

