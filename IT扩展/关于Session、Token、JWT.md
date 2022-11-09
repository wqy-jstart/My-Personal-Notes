# 关于Session、Token、JWT

## 1.关于Session

HTTP协议本身是**无状态**协议，无法保存用户信息，即：某客户端第1次访问了服务器端，可能产生了某些数据，此客户端再次访问服务器端时，服务器端无法识别出这个客户端是此前曾经来访的客户端。

为了能够识别客户端的身份，当某客户端第1次向服务器端发起请求时，服务器端将向客户端响应一个JSESSIONID数据，其本质是一个UUID数据，在客户端后续的访问中，客户端会自动携带此JSESSIONID，以至于服务器端能够识别此客户端的身份。同时，在服务器端，还是一个`Map`结构的数据，此数据是使用JSESSIONID作为Key的，所以，每个客户端在服务器端都有一个与之对应在的在此`Map`中的`Value`，也就是Session数据！

> 提示：UUID是全球唯一的，从设计上，它能够保证在同一时空中的唯一性。

由于Session的运作机制，决定了它必然存在缺点：

- **默认不适用于集群或分布式系统**，因为Session是内存中的数据，所以，默认情况下，Session只存在于与客户端交互的那台服务器上，如果使用了集群，客户端每次请求的服务器都不是同一台服务器，则无法有效的识别客户端的身份
  - 可以通过共享Session等机制解决
- **不适合长时间保存数据**，因为Session是内存中的数据，并且，所有来访的客户端在服务器端都有对应的Session数据，就必须存在Session清除机制，如果长期不清除，随着来访的客户端越来越多，将占用越来越多的内存，服务器将无法存储这大量的数据，通常，会将Session设置为15分钟或最多30分钟清除

## 2.关于Token

Token：票据、令牌

由于客户端种类越来越多，目前，主流的识别用户身份的做法都是使用Token机制，Token可以理解为“票据”，例如现实生活中的“火车票”，某客户端第1次请求服务器，或执行登录请求，则可视为“购买火车票”的行为，当客户端成功登录，相当于成功购买了火车票，客户端的后续访问应该携带Token，相当于乘坐火车需要携带购票凭证，则服务器端可以识别客户端的身份，相当于火车站及工作人员可以识别携带了购买凭证的乘车人。

与Session最大的区别在于：Token是包含可识别的有效信息的！对于需要获取信息的一方而言，只需要具备读取Token信息的能力即可。

> Session机制中客户端需要携带的JSESSIONID本身上是UUID，此数据只具有唯一性，并不是有意义的数据，真正有意义的数据是服务器端内存中的Session数据。

所以，Token并不需要占用较多的内存空间，是可以长时间，甚至非常长时间保存用户信息的！

## 3.关于JWT

- **JWT**：**J**SON **W**eb **T**oken
  - JWT是一种使用JSON格式来组织数据的Token。

- #### JWT官网:https://jwt.io/

  JWT分为头部,载荷,签名三个部分

  ![image-20221108104611125](images/image-20221108104611125.png)

- ### 生成与解析JWT:

  - 需要添加依赖项:


```xml
<!-- JJWT（Java JWT） -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

关于**生成JWT**与**解析JWT**的示例代码：

```java
/**
 * 测试JWT的生成和解析
 *
 * @Author java.@Wqy
 * @Version 0.0.1
 */
@SpringBootTest
public class JwtTests {

    // 设置签名(一个签名对应一个JWT)
    String secretKey = "a9F8ujGDhjgFvfEd3SA90ukDS";

    // 1.生成JWT
    @Test
    public void generate() {
        // 该值应当是保密值,不能外泄
        Date date = new Date(System.currentTimeMillis() + 1000 * 60 * 60);// 设置有效时间,直接new表示当前时间

        Map<String, Object> claims = new HashMap<>();
        claims.put("id", 9527);
        claims.put("username", "fanchuanqi");

        String jwt = Jwts.builder() // 构建者模式(通常用于设置多种参数)
                // 头部Header--固定值
                .setHeaderParam("alg", "HS256")
                .setHeaderParam("typ", "JWT")
                // 载荷PayLoad
                .setClaims(claims) // 传入要放的数据
                // 签名Signature
                .setExpiration(date) // 有效时间
                .signWith(SignatureAlgorithm.HS256, secretKey)
                .compact();
        System.out.println(jwt);// 生成Token----相当于有了火车票
        // eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6OTUyNywiZXhwIjoxNjY3ODc5MDU5LCJ1c2VybmFtZSI6ImZhbmNodWFucWkifQ.ZcMcg9XtUF0Q1-gOOjP3mAr4hMh2fFYE8r6cB20t50Q
    }

    // 2.解析JWT
    @Test
    public void parse() {
        // 解析前需准备之前生成的JWT,和签名
        String jwt = "eyJhbGciOiJIUzI1NiIsInRycCI6IkpXVCJ9.eyJleHAiOjE2NjgzMjE1NjIsInVzZXJuYW1lIjoicm9vdCJ9._bSGd2e7fZ8XtWYzVJrxqSJ0dHaNqF4J0tPmSs6EXZk";
        Claims claims = Jwts.parser()
                .setSigningKey(secretKey) // 设置需要解析的JWT签名
                .parseClaimsJws(jwt) // 拿到需要解析的JWT代码
                .getBody();

        // <T> T get(String var1, Class<T> var2)该重载可返回自定义的数据类型
        Long id = claims.get("id",Long.class);
        String username = claims.get("username",String.class);
        System.out.println("id=" + id + ";用户名=" + username);
    }
}
```

#### 当尝试解析JWT时，可能会出现以下错误：

- 如果JWT已过期，会抛出`ExpiredJwtException`，例如：

```
io.jsonwebtoken.ExpiredJwtException: JWT expired at 2022-11-08T11:24:49Z. Current time: 2022-11-08T11:38:01Z, a difference of 792152 milliseconds.  Allowed clock skew: 0 milliseconds.
```

- 如果JWT数据有误，会抛出`MalformedJwtException`，例如：

```
io.jsonwebtoken.MalformedJwtException: Unable to read JSON value: {"alg":"HS7#�$�uB'
```

- 如果JWT签名不匹配，会抛出`SignatureException`，例如：

```
io.jsonwebtoken.SignatureException: JWT signature does not match locally computed signature. JWT validity cannot be asserted and should not be trusted.
```
