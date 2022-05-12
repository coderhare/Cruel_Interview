## 一、写在前面

关于Java中string和byte如何相互转化。

可以用于base64编码的加解密，TCP通信等。

> 参考链接：https://blog.csdn.net/cyan20115/article/details/106554182
>
> 关于base64编码：https://blog.csdn.net/qq_16268979/article/details/118252846

### 二、Java

 ```java
package com.mkyong.io;
 
import java.nio.charset.StandardCharsets;
 
public class JavaSample {
 
    public static void main(String[] args) {
 
        String example = "This is raw text!";
        byte[] bytes = example.getBytes(); //String -> byte
 
        System.out.println("Text : " + example);
        System.out.println("Text [Byte Format] : " + bytes);
        // no, don't do this, it returns the address of the object in memory
        System.out.println("Text [Byte Format] : " + bytes.toString());
 
        // convert bytes[] to string
        String s = new String(bytes, StandardCharsets.UTF_8);
        System.out.println("Output : " + s);
 
        // UnsupportedEncodingException
        //String s = new String(bytes, "UTF_8");
 
    }
}
输出：
Text : This is raw text!
Text [Byte Format] : [B@5ae9a829
Text [Byte Format] : [B@5ae9a829
Output : This is raw text!
 ```

