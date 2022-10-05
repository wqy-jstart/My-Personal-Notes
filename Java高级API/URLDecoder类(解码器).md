## URLDecoder类(解码器)

#### 作用:利用URLDecoder解码器将16进制按照UTF-8转换为汉字

#### 代码如下:

```java
package com.webserver.test;

import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;

/**
 * 利用URLDecoder解码器将16进制按照UTF-8转换为汉字
 */
public class URLDecoderDemo {
    public static void main(String[] args) {
        String str = "武";
        //一个中文占据三个字节
        byte[] data = str.getBytes(StandardCharsets.UTF_8);
        //1110|0110 1010|1101 1010|0110(武)------对应二进制
        // E    6 |  A    D |  A    6  (武)------对应十六进制---%E6%AD%A6
        System.out.println(Arrays.toString(data));//[-26, -83, -90]

        str = "%E6%AD%A6%E6%B8%85%E6%BA%90";//带百分号的十六进制要被解码器翻译成二进制再用UTF-8字符集转成字符串
        try {
            str = URLDecoder.decode(str,"UTF-8");//URLDecoder解码器
            System.out.println(str);//武清源
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }
}
```

