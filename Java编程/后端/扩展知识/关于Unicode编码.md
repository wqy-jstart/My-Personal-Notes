# 关于Unicode编码

#### 1.Unicode创立之初,规定了编码范围:0~1114111个编码,111万+个码点。

#### 2.Unicode 1.0 收集字符，对应码点

#### 3.Unicode 15.0收集了字符11万多个。

### Unicode只是规定了一个字符一个编码！

Unicode网络传输的时候，使用UTF-8等转换编码进行序列化！

### UTF-8是Unicode的序列化转换编码！！！

```java
//      字符转换为字节                                  字节转换为字符
Unicode字符 --UTF-8编码--> N个字节-->网络-->N个字节 -->UTF-8解码--> Unicode字符
```

##### Unicode 如何转换为UTF-8?

按照Unicode编码数值分段转换:

- 0~127 (2^7-1)范围:Unicode转换为1个字节的UTF-8,  格式:0XXXXXXX    "X中存储编码数据"
- 128~2047 范围Unicode转换为2个字节UTF-8,格式:110XXXXX 10XXXXXX
- 2048~65535 范围Unicode转换为3个字节UTF-8,格式:1110XXXX 10XXXXX 10XXXXX
  - 中文编码在2048~65535范围内,正好时3字节编码!
- 65535~1114111 范围Unicode转换为4个字节UTF-8,格式:11110XXX 10XXXXXX 10XXXXXX 10XXXXXX

UTF-8为了能够标识字节数据的顺序,设计格式位!

> #### Unicode: Universal Character Set ---万国码、通用字符集
>
> #### UTF: Unicode Transformation Format ---Unicode转换格式

### 官网：https://unicode-table.com/