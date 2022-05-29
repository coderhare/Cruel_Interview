## 一、写在前面

关于utf8和utf8mb4。

> https://blog.csdn.net/grl18840839630/article/details/105597074/

## 二、mysql字符集

> 一句话：utf8mb4相比utf8多了emoji的支持。排序规则选用 utf8mb4_general_ci
> 即utf8mb4组合为：utf8mb4	|	 utf8mb4_general_ci 

1、导读
我们新建mysql数据库的时候，需要指定数据库的字符集，一般我们都是选择utf8这个字符集，但是还会又一个utf8mb4这个字符集，好像和utf8有联系，今天就来解析一下这两者的区别。

2、起源
MySQL在5.5.3之后增加了这个utf8mb4的编码，mb4就是most bytes 4的意思，专门用来兼容四字节的unicode。好在utf8mb4是utf8的超集，除了将编码改为utf8mb4外不需要做其他转换。当然，为了节省空间，一般情况下使用utf8也就够了。
可以简单的理解 utf8mb4 是目前最大的一个字符编码,支持任意文字。

3、为什么mysql有utf8和utf8mb4两种几乎差不多的字符集
utf8 是 Mysql 中的一种字符集，只支持最长三个字节的 UTF-8字符，也就是 Unicode 中的基本多文本平面。
Mysql 中的 utf8 为什么只支持持最长三个字节的 UTF-8字符呢？我想了一下，可能是因为 Mysql 刚开始开发那会，Unicode 还没有辅助平面这一说呢。那时候，Unicode 委员会还做着 “65535 个字符足够全世界用了”的美梦。Mysql 中的字符串长度算的是字符数而非字节数，对于 CHAR 数据类型来说，需要为字符串保留足够的长。当使用 utf8 字符集时，需要保留的长度就是 utf8 最长字符长度乘以字符串长度，所以这里理所当然的限制了 utf8 最大长度为 3，比如 CHAR(100) Mysql 会保留 300字节长度。至于后续的版本为什么不对 4 字节长度的 UTF-8 字符提供支持，我想一个是为了向后兼容性的考虑，还有就是基本多文种平面之外的字符确实很少用到。

要在 Mysql 中保存 4 字节长度的 UTF-8 字符，需要使用 utf8mb4 字符集，但只有 5.5.3 版本以后的才支持。我觉得，为了获取更好的兼容性，应该总是使用 utf8mb4 而非 utf8. 对于 CHAR 类型数据，utf8mb4 会多消耗一些空间，根据 Mysql 官方建议，使用 VARCHAR 替代 CHAR。

4、为什么要使用utf8mb4字符集
既然utf8应付日常使用完全没有问题，那为什么还要使用utf8mb4呢? 低版本的MySQL支持的utf8编码，最大字符长度为 3 字节，如果遇到 4 字节的字符就会出现错误了。三个字节的 UTF-8 最大能编码的 Unicode 字符是 0xFFFF，也就是 Unicode 中的基本多文平面（BMP）。也就是说，任何不在基本多文平面的 Unicode字符，都无法使用MySQL原有的 utf8 字符集存储。这些不在BMP中的字符包括哪些呢？最常见的就是Emoji 表情（Emoji 是一种特殊的 Unicode 编码，常见于 ios 和 android 手机上），和一些不常用的汉字，以及任何新增的 Unicode 字符等等。
那么utf8mb4比utf8多了什么的呢?
多了emoji编码支持.
如果实际用途上来看,可以给要用到emoji的库或者说表,设置utf8mb4.
比如评论要支持emoji可以用到。

5、新建mysql库的排序规则
utf8_unicode_ci比较准确，utf8_general_ci速度比较快。通常情况下 utf8_general_ci的准确性就够我们用的了，在我看过很多程序源码后，发现它们大多数也用的是utf8_general_ci，所以新建数据 库时一般选用utf8_general_ci就可以了
如果是utf8mb4那么对应的就是 utf8mb4_general_ci utf8mb4_unicode_ci
