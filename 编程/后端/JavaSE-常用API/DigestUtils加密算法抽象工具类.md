# DigestUtils加密算法抽象工具类

用户注册时（管理员添加新的账号信息时）的密码必须经过某种算法进行编码，并将得到的结果存储到数据库，而不允许将原始密码直接存储到数据库中！

原始密码通常可以称之为密码的**原文**，或称之为**明文**密码，经过编码处理的结果可称之为**密文**。

对明文密码进行编码处理时，**不可以**使用任何**加密算法**，因为**所有加密算法都是可以逆向运算的**，即：根据算法类型、加密参数、密文，可以通过运算得到密码原文！通常，加密算法只是用于保障数据在传输过程中的安全性！

#### 常见的消息算法有：

- MD（Message Digest）系列
  - MD2(128位，不推荐) / MD4(128位，不推荐) / MD5(128位)
- SHA（Secure Hash Algorithm）家族
  - SHA-1(160位，不推荐) / SHA-256(256位) / SHA-384(384位) / SHA-512(512位)

##### 在Spring Boot项目中，已有`DigestUtils`工具类，提供了使用MD5算法进行编码的API：

- `org.springframework.util.DigestUtils;`在Spring框架的util工具类包下

- ```java
  String md5DigestAsHex(byte[] bytes){}//该方法可将原文转换为密文
  ```

#### 测试类:

```java
package cn.tedu.csmall.product;

import org.junit.jupiter.api.Test;
import org.springframework.util.DigestUtils;

public class Md5Tests {

    @Test
    public void encode() {
        String rawPassword = "12345678";
        String encodedPassword = DigestUtils
                    .md5DigestAsHex(rawPassword.getBytes());
        System.out.println("原文：" + rawPassword);
        System.out.println("密文：" + encodedPassword);
    }
}
```
