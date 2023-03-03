## HttpURLConnection类:

```java
java.lang.Object
  java.net.URLConnection
      java.net.HttpURLConnection
```

URLConnection是个抽象类，它有两个直接子类分别是HttpURLConnection和JarURLConnection。

另外一个重要的类是URL，通常URL可以通过传给构造器一个String类型的参数来生成一个指向特定地址的URL实例。
每个 HttpURLConnection 实例都可用于生成单个请求，但是其他实例可以透明地共享连接到 HTTP 服务器的基础网络。

请求后在 HttpURLConnection 的 InputStream 或 OutputStream 上调用 close() 方法可以释放与此实例关联的网络资源，但对共享的持久连接没有任何影响。如果在调用 disconnect() 时持久连接空闲，则可能关闭基础套接字。