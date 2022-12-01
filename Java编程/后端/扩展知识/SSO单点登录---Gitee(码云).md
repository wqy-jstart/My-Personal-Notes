# SSO单点登录---Gitee(码云)

#### 该项目参考Gitee官方的OAuth文档来实现授权过程

![image-20221201184225945](images/image-20221201184225945.png)

### 授权流程:

该项目使用了授权码的模式来完成!

1.应用通过 浏览器 或 Webview 将用户引导到码云三方认证页面上（ **GET请求** ）

```sh
https://gitee.com/oauth/authorize?client_id={client_id}&redirect_uri={redirect_uri}&response_type=code
```

2.用户对应用进行授权

如果之前已经授权过的需要跳过授权页面，需要在上面第一步的 URL 加上 scope 参数，且 scope 的值需要和用户上次授权的勾选的一致。如用户在上次授权了user_info、projects以及pull_requests。则步骤A 中 GET 请求应为：

```sh
https://gitee.com/oauth/authorize?client_id={client_id}&redirect_uri={redirect_uri}&response_type=code&scope=user_info%20projects%20pull_requests
```

3.码云认证服务器通过回调地址{redirect_uri}将 用户授权码 传递给 应用服务器 或者直接在 Webview 中跳转到携带 用户授权码的回调地址上，Webview 直接获取code即可（{redirect_uri}?code=abc&state=xyz)

4.应用服务器 或 Webview 使用 access_token API 向 码云认证服务器发送post请求传入 用户授权码 以及 回调地址（ **POST请求** ）**注：请求过程建议将 client_secret 放在 Body 中传值，以保证数据安全。**

```sh
https://gitee.com/oauth/token?grant_type=authorization_code&code={code}&client_id={client_id}&redirect_uri={redirect_uri}&client_secret={client_secret}
```

5.码云认证服务器返回 access_token,应用通过 access_token 访问 Open API 使用用户数据。

6.当 access_token 过期后（有效期为一天），你可以通过以下 refresh_token 方式重新获取 access_token（ **POST请求** ）

```sh
https://gitee.com/oauth/token?grant_type=refresh_token&refresh_token={refresh_token}
```

- 注意：如果获取 access_token 返回 403，可能是没有设置User-Agent的原因。
  详见：[获取Token时服务端响应状态403是什么情况](https://gitee.com/oschina/git-osc/issues/IDBSA)

> ##### 后端项目地址:https://gitee.com/jstart/login-sso-system
>
> ##### 前端项目地址:https://gitee.com/jstart/login-sso-client

### 网络传输OkHttp:

文档地址:https://square.github.io/okhttp/

提供:

1.Post请求传入url获取响应信息

2.Get请求传入url获取信息

**依赖:**

```xml
<!--http客户端-->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.6</version>
</dependency>

<!--Apache的FastJson-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.51</version>
</dependency>

<!--okhttp-->
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>3.14.1</version>
</dependency>
```
