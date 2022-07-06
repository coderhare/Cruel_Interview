## 一、写在前面

关于@ConditionalOnProperty注解

> 参考链接：https://blog.csdn.net/sqlgao22/article/details/96476754
>
> https://blog.51cto.com/u_2870645/3055689

## 二、Springboot注解

### 1、关于使用

在spring boot中有时候需要控制配置类是否生效,可以使用@ConditionalOnProperty注解来控制@[Configuration](https://so.csdn.net/so/search?q=Configuration&spm=1001.2101.3001.7020)是否生效。

1、name或value是必填项
2、matchIfMissing：当未找到对应配置是否匹配(默认不匹配)
3、常用组合：(prefix)+name+havingValue 判断是否包含某属性且属性值与havingValue一致
(prefis)+value 判断是否包含所有vvalue

### 2、一些例子

> 参考链接: https://blog.51cto.com/u_2870645/3055689

> https://blog.csdn.net/skh2015java/article/details/120354752
