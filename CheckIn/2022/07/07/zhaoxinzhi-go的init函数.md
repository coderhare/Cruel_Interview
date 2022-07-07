## 一、写在前面

关于go的init函数

> 参考链接；https://blog.csdn.net/dqz_nihao/article/details/122888050
>
> https://blog.51cto.com/ghostwritten/5345384

## 二、go

### 1、init函数介绍

一句话理解：类似python的包里面的`__init__.py`文件，或者说类似一个py文件中的global语句（print啥的也算）

注意是`init()`函数而不是`Init()`函数！！

注意`init()`函数没有参数，也没有返回值！！

注意`init()`函数在程序运行时，自动自动被调用执行，不能在代码中主动调用它。

> 主动调用会报错：undefined: init

在Go语言程序执行时导入包语句会自动触发包内部init()函数的调用。

每个包可以有多个init函数，包的每个源文件也可以有多个init函数。

同一个包的init执行顺序，golang没有明确定义，编程时要注意程序不要依赖这个执行顺序。

不同包的init函数按照包导入的依赖关系决定执行顺序。

### 2、init函数的主要用途

#### 1、初始化不能使用初始化表达式初始化的变量

比如：

```go
import _ "net/http/pprof"
```

golang对没有使用的导入包会编译报错，但是有时我们只想调用该包的init函数，不使用包导出的变量或者方法，这时就采用上面的导入方案。（==此时pprof中的全局声明会生效吗？生效指被执行==）

执行上述导入后，init函数会启动一个异步协程采集该进程实例的资源占用情况，并以http服务接口方式提供给用户查询。

#### 2、程序运行前的注册。

#### 3、实现sync.Once功能。

##### 

### 3、init函数执行时机

先执行该包下所有`.go`文件的`init()`函数，在执行`main()`函数

![在这里插入图片描述](https://img-blog.csdnimg.cn/71a8591f93824655b2c76092d65c8c7e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZHF6X25paGFv,size_20,color_FFFFFF,t_70,g_se,x_16)

输出结果

```
10
Hello 沙河
```

我理解，对于一个pkg，先执行所有全局语句，然后执行`main`函数所在文件的`init()`，然后按照字典序执行其他文件的`init()`，然后执行`main()`

> 对于这一点，在https://blog.51cto.com/ghostwritten/5345384文章中的示例3也有说明

### 4、多个包，init函数执行顺序

`Go`语言包会从`main`包检查其导入的所有包 ，每个包又可能导入了其他的包。`Go`编译器由此构建出一个树状的包引用关系，再根据引用顺序决定编译顺序，依次编译这些包的代码。

在运行时，最后导入的包会最先初始化并调用其`init()`函数，如下图示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/e41f6ce5a4224f5baef76e5e36a6f27e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZHF6X25paGFv,size_20,color_FFFFFF,t_70,g_se,x_16)

init函数比较特殊，可以在包里被多次定义。

同一个包不同源文件的init函数执行顺序，golang spec没做说明，以上述程序输出来看，执行顺序是源文件名称的字典序。



> 更详细的链接：（其实可以参考python的模块管理进行理解）
>
> https://blog.51cto.com/ghostwritten/5345384
