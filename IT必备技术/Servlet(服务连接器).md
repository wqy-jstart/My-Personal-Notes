# 什么是Servlet?

- Servlet（Server Applet）是Java Servlet的简称，称为小服务程序或服务连接器，用Java编写的服务器端程序，具有独立于平台和协议的特性，主要功能在于交互式地浏览和生成数据，生成动态Web内容。

- 狭义的Servlet是指Java语言实现的一个接口，广义的Servlet是指任何实现了这个Servlet接口的类，一般情况下，人们将Servlet理解为后者。Servlet运行于支持Java的应用服务器中。从原理上讲，Servlet可以响应任何类型的请求，但绝大多数情况下Servlet只用来扩展基于HTTP协议的Web服务器。


- 最早支持Servlet标准的是JavaSoft的Java Web Server，此后，一些其它的基于Java的Web服务器开始支持标准的Servlet。

## 实现过程:

#####     最早支持 Servlet 技术的是 JavaSoft 的 Java Web Server。此后，一些其它的基于 Java 的 Web Server 开始支持标准的 Servlet API。Servlet 的主要功能在于交互式地浏览和修改数据，生成动态 Web 内容。这个过程为：

1. ##### 客户端发送请求至服务器端；

2. ##### 服务器将请求信息发送至 Servlet；

3. ##### Servlet 生成响应内容并将其传给[服务器](https://baike.baidu.com/item/服务器?fromModule=lemma_inlink)。响应内容动态生成，通常取决于客户端的请求；

4. ##### 服务器将响应返回给客户端。

- Servlet 看起来像是通常的 Java 程序。Servlet 导入特定的属于 Java Servlet API 的包。因为是对象[字节码](https://baike.baidu.com/item/字节码?fromModule=lemma_inlink)，可动态地从网络加载，可以说 Servlet 对 Server 就如同 Applet对 Client 一样，但是，由于 Servlet 运行于 Server 中，它们并不需要一个[图形用户界面](https://baike.baidu.com/item/图形用户界面?fromModule=lemma_inlink)。从这个角度讲，Servlet 也被称为 FacelessObject。

- 一个 Servlet 就是 Java 编程语言中的一个类，它被用来扩展[服务器](https://baike.baidu.com/item/服务器?fromModule=lemma_inlink)的性能，服务器上驻留着可以通过“请求-响应”编程模型来访问的应用程序。虽然 Servlet 可以对任何类型的请求产生响应，但通常只用来扩展 Web 服务器的应用程序。

## 生命周期:

1. 客户端请求该 Servlet；
2. 加载 Servlet 类到内存；
3. 实例化并调用init()方法初始化该 Servlet；
4. service()（根据请求方法不同调用doGet() 或者 doPost()，此外还有doHead()、doPut()、doTrace()、doDelete()、doOptions()、destroy())。
5. 加载和实例化 Servlet。这项操作一般是动态执行的。然而，Server 通常会提供一个管理的选项，用于在 Server 启动时强制装载和初始化特定的 Servlet。

Server 创建一个 Servlet的实例

第一个客户端的请求到达 Server

Server 调用 Servlet 的 init() 方法（可配置为 Server 创建 Servlet 实例时调用，在 web.xml 中 标签下配置 标签，配置的值为整型，值越小 Servlet 的启动优先级越高）

一个客户端的请求到达 Server

Server 创建一个请求对象，处理客户端请求

Server 创建一个响应对象，响应客户端请求

Server 激活 Servlet 的 service() 方法，传递请求和响应对象作为参数

service() 方法获得关于请求对象的信息，处理请求，访问其他资源，获得需要的信息

service() 方法使用响应对象的方法，将响应传回Server，最终到达客户端。service()方法可能激活其它方法以处理请求，如 doGet() 或 doPost() 或程序员自己开发的新的方法。

对于更多的客户端请求，Server 创建新的请求和响应对象，仍然激活此 Servlet 的 service() 方法，将这两个对象作为[参数传递](https://baike.baidu.com/item/参数传递?fromModule=lemma_inlink)给它。如此重复以上的循环，但无需再次调用 init() 方法。一般 Servlet 只初始化一次(**只有一个对象**)，当 Server 不再需要 Servlet 时（一般当 Server 关闭时），Server 调用 Servlet 的 destroy() 方法。

![image-20221020101105082](images/image-20221020101105082.png)

1.第一个到达服务器的 HTTP 请求被委派到 Servlet 容器。

2.Servlet 容器在调用 service() 方法之前加载 Servlet。

3.然后 Servlet 容器处理由多个线程产生的多个请求，每个线程执行一个单一的 Servlet 实例的 service() 方法。

## 工作模式:

- ##### 客户端发送请求至[服务器](https://baike.baidu.com/item/服务器?fromModule=lemma_inlink)；

- ##### 服务器启动并调用 Servlet，Servlet 根据客户端请求生成响应内容并将其传给服务器；

- ##### 服务器将响应返回客户端。

## 优点:

- #### 方便

##### Servlet 提供了大量的实用工具例程，例如自动地解析和解码 HTML 表单数据、读取和设置 [HTTP](https://baike.baidu.com/item/HTTP?fromModule=lemma_inlink)头、处理[Cookie](https://baike.baidu.com/item/Cookie?fromModule=lemma_inlink)、跟踪会话状态等。

- #### 功能强大

##### 在Servlet中，许多使用传统 CGI 程序很难完成的任务都可以轻松地完成。例如，Servlet 能够直接和 Web[服务器](https://baike.baidu.com/item/服务器?fromModule=lemma_inlink)交互，而普通的 CGI 程序不能。Servlet 还能够在各个程序之间共享数据，使得数据库[连接池](https://baike.baidu.com/item/连接池?fromModule=lemma_inlink)之类的功能很容易实现。