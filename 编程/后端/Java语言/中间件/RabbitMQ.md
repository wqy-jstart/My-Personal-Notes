# RabbitMQ消息中间件

## 一、概念：

***MQ*（Message Queue）消息队列**，是基础数据结构中“先进先出”的一种数据结构。一般用来解决**<u>流量削峰</u>**，**<u>应用解耦</u>**，**<u>异步处理</u>**等问题，实现高性能，高可用，可伸缩和最终一致性架构。

### 1.流量消峰：

**问题**：

为了防止双十一因单位时间下单量过大导致的限制用户下单行为，会影响用户的满意度。

**使用MQ消息队列解决该问题**：

使用消息队列将订单做缓冲，取消该限制将订单分时间段进行处理。

### 2.应用解耦：

**问题**：

电商应用有订单系统、库存系统、支付系统，用户下单后，如果耦合性的调用三个系统时，任何一个系统出故障都会导致下单异常。

**使用MQ消息队列解决该问题**：

当物流系统出问题，会将要处理的内存数据先缓存到消息队列，不影响支付，问题解决后再进行物流系统的处理，用户感受不到故障，提升用户体验！

![image-20230204151554372](images/image-20230204151554372.png)

### 3.异步处理：

**问题**：

很多服务之间的调用是异步的，如A调用B，B花很长时间去执行，A不知B何时执行完，通常：A一段时间内调B去查询，或者提供一个`CallBack API`，当B执行完调用`CallBack API`通知A。

**远程调用问题**：

Dubbo调用普遍存在于我们的微服务项目中，这些Dubbo调用全部是同步的操作

这里的"同步"指:消费者A调用生产者B之后,A的线程会进入阻塞状态,等待生产者B运行结束返回之后,A才能运行之后的代码

![image-20220519152055211](images/image-20220519152055211.png)

Dubbo消费者发送调用后进入阻塞状态,这个状态表示该线程仍占用内存资源,但是什么动作都不做

如果生产者运行耗时较久,消费者就一直等待,如果消费者利用这个时间,那么可以处理更多请求,业务整体效率会提升

实际情况下,Dubbo有些必要的返回值必须等待,但是不必要等待的服务返回值,我们可以不等待去做别的事情

**使用MQ消息队列解决该问题**：
当A调用B服务后，只需监听B处理完成的消息，当B完成后会发送消息给MQ，并转告A服务，这样就无需循环调B查询API，也不用提供`CallBack API`,A服务也能及时得到异步处理的结果！

## 二、MQ分类：

### 1.ActiveMQ

**优点**：**单机吞吐量万级**，**时效性ms级**，**可用性高**，基于主从架构实现高可用性，消息可靠性较低的概率丢失数据

**缺点**：官方对ActiveMQ 5.X**维护越来越少**，高吞吐量较少使用

视频: http://www.gulixueyuan.com/course/322

### 2.Kafka

**大数据的杀手锏**，**百万级TPS的吞吐量**，在数据采集、传输、存储的过程中发挥着举足轻重的作用。目前已经被 LinkedIn，Uber, Twitter, Netflix 等大公司所采纳。

**优点**：性能卓越，**单机写入TPS约在百万条/秒**，**吞吐量高**，Kafka是**分布式**的，一个数据多个版本，少数宕机不会丢失数据，不会导致不可用,消费者采用 Pull 方式获取消息, 消息有序, 通过控制能够保证所有消息被消费且仅被消费一次;有优秀的第三方Kafka Web 管理界面 Kafka-Manager；在日志领域比较成熟，被多家公司和多个开源项目使用；功能支持：功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及**日志采集**被大规模使用

**缺点**：Kafka单机超过64个队列/分区，**Load会发生明显的飙高现象**，队列越多，Load越高，消息响应时间越长，使用轮询方式，实时性取决于轮询间隔时间，**消息失败不支持重试**，支持消息顺序但是一台代理宕机就会产生消息乱序，**社区更新较慢**。

### 3.RocketMQ

出自**阿里巴巴的开源产品**，用**Java语言实现**，在设计时参考了Kafka，在此之上有些改进，被阿里广泛用于订单、交易、流计算、消息推送、日志流处理。

**优点**：**单机吞吐十万级**，**可用性非常高**，**分布式架构**，**消息可以做到0丢失**，**功能完善**，**扩展性好**，**支持10亿级别的消息堆积**，不会因此下降性能，源码可阅读。

**缺点**：**支持的客户端语言不多**，目前是java以及C++，其中C++不成熟；社区活跃度一般,没有在 MQ核心中去实现 JMS 等接口,有些系统要迁移需要修改大量代码。

### 4.RabbitMQ

2007 年发布，是一个在 AMQP(高级消息队列协议)基础上完成的，可复用的企业消息系统，是**当前最主流的消息中间件之一**。

**优点**：由于**Erlang语言的高并发特性**，性能较好；**吞吐量万级**，MQ**功能比较完备**,健壮、稳定、易用、跨平台、**支持多种语言** 如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持 AJAX 文档齐全；开源提供的管理界面非常棒，用起来很好用,**社区活跃度高**；更新频率相当高。

**缺点**：商业版需要**收费**，学习**成本较高**。

## 三、RabbitMQ核心：

### 1.生产者`Producer`

产生数据发送消息的程序

### 2.交换机`Exchange`

交换机是RabbitMQ非常重要的一个部件，一方面它接收来自生产者的消息，另一方面将消息推送到队列。

**交换机需要决定如何处理接收的消息，推送到特定队列还是多个队列**。

### 3.队列`Queue`

队列是RabbitMQ内部使用的一种数据结构，尽管消息流经RabbitMQ和应用，但消息始终是储存在队列中的。队列受主机的内存和磁盘约束，本质是一个大的消息缓冲区。许多生产者可以将消息发送到一个队列，许多消费者可以尝试向一个队列取数据。

### 4.消费者`Consumer`

消费与接收具有相似意义。消费者大多是等待接收消息的程序。

生产者、消费者、消息中间件很多时候并不是在同一机器上，同一个APP既可以是生产者`Producer`也可以是消费者`Consumer`

![image-20230204155027853](images/image-20230204155027853.png)

## 四、RabbitMQ工作原理：

![image-20230204155105632](images/image-20230204155105632.png)

**描述**：

1. 首先与RabbitMQ服务建立连接
2. 生产者利用连接`connection`创建一个信道`Channel`，通过信道发送消息到交换机`Exchange`
3. 交换机接收生产者发送的消息，并将消息发送到队列`Queue`
4. 消费者`Consumer`通过连接创建信道`Channel`，通过该信道进行接收队列中的信息。

## 五、安装：

### Windows环境：

#### 1.下载软件

RabbitMQ是Erlang语言开发的,所以要先安装Erlang语言的运行环境

下载Erlang的官方路径

https://erlang.org/download/otp_versions_tree.html

![image-20220520103318825](images/image-20220520103318825.png)

安装的话就是双击

安装过程中都可以使用默认设置,需要注意的是

**不要安装在中文路径和有空格的路径下!!!**

下载RabbitMQ的官方网址

https://www.rabbitmq.com/install-windows.html

![image-20220520104034920](images/image-20220520104034920.png)

安装也是双击即可

**不要安装在中文路径和有空格的路径下!!!**

#### 2.配置Erlang的环境变量

要想运行RabbitMQ必须保证系统有Erlang的环境变量

配置Erlang环境变量

把安装Erlang的bin目录配置在环境变量Path的属性中

![image-20220907151502819](images/image-20220907151502819.png)

#### 3.启动RabbitMQ

找到RabbitMQ的安装目录

例如：

```shell
D:\tools\rabbit\rabbitmq_server-3.10.1\sbin
```

具体路径根据自己的情况寻找

地址栏运行cmd

输入启动指令如下

```sh
D:\tools\rabbit\rabbitmq_server-3.10.1\sbin>rabbitmq-plugins enable rabbitmq_management
```

结果如下：

![image-20220810153001965](images/image-20220810153001965.png)

运行完成后,验证启动状态

#### 4.测试访问

RabbitMQ自带一个管理的界面,所以我们可以访问这个界面来验证它的运行状态

http://localhost:15672

![image-20220810153420567](images/image-20220810153420567.png)

登录界面用户名密码

```sh
账号：guest
密码：guest
```

登录成功后看到RabbitMQ运行的状态

如果启动失败,可以手动启动RabbitMQ

参考路径如下

https://baijiahao.baidu.com/s?id=1720472084636520996&wfr=spider&for=pc

### Linux环境：

官网：https://rabbitmq.com/download.html

安装RabbitMQ的前提需要安装[Erlang](https://baike.baidu.com/item/Erlang/1152752?fr=aladdin)语言的环境（网盘中有）

通常企业使用Linux系统运行RabbitMQ

#### 1.安装Erlang语言环境：

```sh
[root@localhost dev]# ls
erlang-21.3-1.el7.x86_64.rpm  rabbitmq-server-3.8.8-1.el7.noarch.rpm
[root@localhost dev]# rpm -ivh erlang-21.3-1.el7.x86_64.rpm
```

#### 2.安装RabbitMQ

先安装一个依赖包：

```sh
yum install socat -y
```

安装RabbitMQ：

```sh
[root@localhost dev]# ls
erlang-21.3-1.el7.x86_64.rpm  rabbitmq-server-3.8.8-1.el7.noarch.rpm
[root@localhost dev]# rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm
```

#### 3.启动RabbitMQ

```sh
chkconfig rabbitmq-server on # 开机自启动
/sbin/service rabbitmq-server start # 启动
/sbin/service rabbitmq-server status # 查看状态
/sbin/service rabbitmq-server stop # 停止
```

> active说明是活着的，running说明已经启动！

#### 4.开启Web管理插件

先停止服务

```sh
/sbin/service rabbitmq-server stop # 停止rabbitmq服务
```

安装插件：

```sh
rabbitmq-plugins enable rabbitmq_management
```

#### 5.测试访问

保证RabbitMQ服务启动的情况下打开Google浏览器。

地址栏输入：<u>连接的系统主机IP:端口号15672</u>

第一次访问可能无果，原因是因为Linux服务器的防火墙未关闭，需关闭：

```sh
systemctl stop firewalld
```

![image-20230203204637992](images/image-20230203204637992.png)

> 出现该结果说明插件测试成功！

##### 初始化账户密码guest：

初次登录会出错：`User can only log in via localhost`

#### 6.添加新用户

##### 创建账号和密码：

```sh
[root@localhost dev]# rabbitmqctl add_user admin 123456
Adding user "admin" ...
```

##### 设置用户角色：

```sh
[root@localhost dev]# rabbitmqctl set_user_tags admin administrator
Setting tags for user "admin" to [administrator] ...
```

##### 设置用户权限：

```sh
# 规范 set_permissions[-p <vhostpath>] <user> <conf> <write> <read>
[root@localhost dev]# rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
Setting permissions for user "admin" in vhost "/" ...
```

用户user_admin具有/vhost1这个virtual host中所有资源的配置写、读权限

##### 查看当前用户和角色：

```sh
rabbitmqctl list_users
```

##### 登录：

![image-20230203205941772](images/image-20230203205941772.png)

进入服务内部！

## 六、使用RabbitMQ：

### <u>1.Hello World</u>

使用Java编写两个程序。发送单个消息的生产者和接受消息并打印出来的消费者。介绍Java API中的一些细节。

在下图中，“ P”是我们的生产者，“ C”是我们的消费者。中间的框是一个队列-RabbitMQ 代表使用者保留的消息缓冲区

![image-20230203210648018](images/image-20230203210648018.png)

#### 1.依赖：

```xml
<!--指定 jdk 编译版本-->
<build>
   <plugins>
      <plugin>
         <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
              <configuration>
                <source>8</source>
                <target>8</target>
              </configuration>
       </plugin>
   </plugins>
</build>
<dependencies>
     <!--rabbitmq 依赖客户端-->
     <dependency>
         <groupId>com.rabbitmq</groupId>
         <artifactId>amqp-client</artifactId>
         <version>5.8.0</version>
     </dependency>
     <!--操作文件流的一个依赖-->
     <dependency>
         <groupId>commons-io</groupId>
         <artifactId>commons-io</artifactId>
         <version>2.6</version>
     </dependency>
</dependencies>
```

#### 2.代码逻辑：

![image-20230203212135867](images/image-20230203212135867.png)

#### 3.消息生产者：

// 生产消息的Java代码

```java
package com.jstart.rabbitmq.one;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * 生产者：发消息
 *
 * @author java@Wqy
 * @version 0.0.1
 * @since 2023.2.3
 */
public class Producer {
    // 队列名称
    public static final String QUEUE_NAME = "hello";

    // 发消息
    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 工厂IP连接RabbitMQ的队列
        factory.setHost("192.168.159.128");
        // 用户名
        factory.setUsername("admin");
        // 密码
        factory.setPassword("123456");

        // 创建连接connection
        Connection connection = factory.newConnection();
        // 获取信道channel
        Channel channel = connection.createChannel();
        /*
         * 生成一个队列
         * 1.队列名称
         * 2.队列里面的消息是否持久化（存储在磁盘上）默认情况消息保存在内存中
         * 3.该队列是否只供一个消费者进行消费 是否进行消息的共享，允许可以多个消费者
         * 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true自动删除
         */
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        // 发消息
        String message = "Hello World";// 初次使用

        /*
         * 发送一个消费
         * 1.发送到哪个交换机
         * 2.路由的Key值是哪一个 本次是队列的名称
         * 3.表示其他参数信息
         * 4.发送消息的消息体
         */
        channel.basicPublish("", QUEUE_NAME, null,message.getBytes());
        System.out.println("消息发送完毕！");
    }
}
```

查看服务页面：

![image-20230203214538028](images/image-20230203214538028.png)

#### 4.消息消费者：

// 消费生产者生产的消息

```java
package com.jstart.rabbitmq.one;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * 消费者：消费消息
 *
 * @author java@Wqy
 * @version 0.0.1
 * @since 2023.2.3
 */
public class Consumer {
    // 队列名称
    public static final String QUEUE_NAME = "hello";
    // 接收消息
    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 工厂IP连接RabbitMQ的队列
        factory.setHost("192.168.159.128");
        // 用户名
        factory.setUsername("admin");
        // 密码
        factory.setPassword("123456");
        // 建立连接connection
        Connection connection = factory.newConnection();
        // 创建信道channel
        Channel channel = connection.createChannel();

        // Lambda表达式()->{};
        // 声明接收消息
        DeliverCallback deliverCallback = (consumerTag,message)
            // 获取消息体(如果不用消息体会输出被处理的消息对象例如:com.rabbitmq.client.Delivery@4039c79e)
                -> System.out.println(new String(message.getBody()));
        // 取消消息时的回调
        CancelCallback cancelCallback = consumerTag -> System.out.println("消息消费被中断");

        /*
         * 消费者消费消息
         * 1.消费哪个队列
         * 2.消费成功之后是否自动应答 true/false
         * 3.消费者未成功消费的回调
         * 4.消费者取消消费的回调
         */
        channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
    }
}
```

### <u>2.Work Queues</u>

工作队列(又称任务队列)的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。

相反我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行作业。当有多个工作线程时，这些工作线程将一起处理这些任务。

![image-20230203221240487](images/image-20230203221240487.png)

#### 1.轮询分发消息：

关于对RabbitMQ的连接代码会被重复使用，为防止冗余，将其抽取到工具类中。

```java
public class RabbitMqUtils {

    // 获取信道channel的静态方法
    public static Channel getChannel() throws Exception {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 工厂IP连接RabbitMQ的队列
        factory.setHost("192.168.159.128");
        // 用户名
        factory.setUsername("admin");
        // 密码
        factory.setPassword("123456");
        // 建立连接connection
        Connection connection = factory.newConnection();
        // 创建信道channel
        return connection.createChannel();
    }
}
```

**多线程消费者**：

```java
public class WorkerTh1 {
    // 队列名称
    public static final String QUEUE_NAME = "hello";

    // 接收消息
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();

        // 声明接收消息
        DeliverCallback deliverCallback = (consumerTag, message)
                // 获取消息体(如果不用消息体会输出被处理的消息对象例如：com.rabbitmq.client.Delivery@4039c79e)
                -> System.out.println("接收到的消息：" + new String(message.getBody()));
        // 取消消息时的回调
        CancelCallback cancelCallback = consumerTag -> System.out.println(consumerTag + "消息消费被消费者取消消费接口回调逻辑");

        // 消息的接收
        System.out.println("C2等待接收消息......");
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}
```

**生产者发送大量消息**：

```java
public class Task01 {

    // 队列名称
    public static final String QUEUE_NAME = "hello";

    // 发送大量消息
    public static void main(String[] args) throws Exception {
        try (Channel channel = RabbitMqUtils.getChannel()) {
            channel.queueDeclare(QUEUE_NAME, true, false, false, null);
            // 从控制台当中接收信息
            Scanner scanner = new Scanner(System.in);// 扫描控制台输入的内容
            while (scanner.hasNext()) {
                String message = scanner.next();
                channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
                System.out.println("发送消息完成：" + message);
            }
        }
    }
}
```

**测试**：

开启多个消费者线程，开启一个生产者利用控制台发送大量消息，消费者进行轮询接收消息。

![image-20230204113306113](images/image-20230204113306113.png)

### 3.消息应答

#### 1.概念

消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然它挂掉了，会发生什么情况。RabbitMQ 一旦向消费者传递了一条消息，便立即将该消息标记为删除。在这种情况下，突然有个消费者挂掉了，我们将丢失正在处理的消息。以及后续发送给该消费这的消息，因为它无法接收到。

为了保证消息在发送过程中不丢失，rabbitmq 引入消息应答机制，消息应答就是: **消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了。**

#### 2.自动应答

消息发送后立即被认为已经传送成功，这种模式需要在**高吞吐量和数据传输安全性方面做权衡**,因为这种模式如果消息在接收到之前，消费者那边出现连接或者 channel 关闭，那么消息就丢失了,当然另一方面这种模式消费者那边可以传递过载的消息，**没有对传递的消息数量进行限制**，当然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使得内存耗尽，最终这些消费者线程被操作系统杀死，**所以这种模式仅适用在消费者可以高效并以某种速率能够处理这些消息的情况下使用**。

#### 3.消息应答的方法

**A.Channel.basicAck(用于肯定确认)**

​    RabbitMQ已知道该消息并且成功的处理消息，可以将其丢弃了

**B.Channel.basicNack(用于否定确认)**

**C.Channel.basicReject(用于否定确认)**

​    与Channel.basicNack相比少一个参数

​    不处理该消息了直接拒绝，可以将其丢弃了。

#### 4.Multiple的解释

**<u>手动应答的好处是可以批量应答并且减少网络拥堵</u>**

multiple 的 true 和 false 代表不同意思：

- true 代表批量应答 channel 上未应答的消息

比如说 channel 上有传送 tag 的消息 5,6,7,8 当前 tag 是 8 那么此时5-8 的这些还未应答的消息都会被确认收到消息应答

![image-20230204115449575](images/image-20230204115449575.png)

- false 同上面相比只会应答 tag=8 的消息 5,6,7 这三个消息依然不会被确认收到消息应答

![image-20230204115516036](images/image-20230204115516036.png)

#### 5.消息的自动重新入队

如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息未发送 ACK 确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队。**如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者**。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息。

#### 6.手动应答

**消息手动应答时，如果消费方宕机可保证消息不丢失，重新放回到队列中以供消费！**

消息生产者：

```java
public class ProducerToRes {

    public static final String TASK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();

        // 声明队列
        channel.queueDeclare(TASK_QUEUE_NAME,false,false,false,null);
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String message = scanner.nextLine();
            channel.basicPublish("", TASK_QUEUE_NAME,null,message.getBytes("UTF-8"));
            System.out.println("生产者发出消息："+message);
        }
    }
}
```

消息消费者：

```java
public class Worker01 {

    // 队列名称
    public static final String TASK_QUEUE_NAME = "ack_queue";

    // 接收消息
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("C1等待接收消息处理时间较短...");

        // Lambda表达式()->{};
        // 声明接收消息
        DeliverCallback deliverCallback = (consumerTag, message) -> {
            // 沉睡1s
            SleepUtils.sleep(1); // 另一个消费者睡眠30秒
            System.out.println("接收的消息：" + new String(message.getBody(),"UTF-8"));
            // 手动应答
            /*
             1.消息的标记 tag
             2.是否批量应答 false：不批量应答信道中的消息
             */
            channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
        };
        // 取消消息时的回调
        CancelCallback cancelCallback = consumerTag -> System.out.println("消费者取消消费接口回调逻辑");

        // 采用手动应答 ←←←
        boolean autoAck = false;
        channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, cancelCallback);
    }
}
```

**总结**：

消息手动应答模式下，消息生产者发送消息后，两个消费者处理消息的时间不同，但在不宕机的情况下，最终都会把消息处理掉，即使宕机，消息也不会宕机，而是重新放回到队列，供其他消费者消费！

![image-20230207170408524](images/image-20230207170408524.png)

### 4.RabbitMQ的持久化

#### 1.概念

刚刚我们已经看到了如何处理任务不丢失的情况，但是**如何保障当 RabbitMQ 服务停掉以后消息生产者发送过来的消息不丢失**。默认情况下 RabbitMQ 退出或由于某种原因崩溃时，它忽视队列和消息，除非告知它不要这样做。确保消息不会丢失需要做两件事：**我们需要将队列和消息都标记为持久化**。

#### 2.队列实现持久化

在手动应答时我们创建的生产者是非持久化的，

```java
// 第二个参数durable是false是为非持久化！
channel.queueDeclare(TASK_QUEUE_NAME,false,false,false,null);
```

如果在当前生产者代码将其改为持久化，启动会报错！

```sh
channel error; protocol method: #method<channel.close>(reply-code=406, reply-text=PRECONDITION_FAILED - inequivalent arg 'durable' for queue 'ack_queue' in vhost '/': received 'true' but current is 'false', class-id=50, method-id=10)
```

原因是该生产者的信道已经创建了，不能直接对其修改，需要删除或重新创建，将其定义为持久化才行！---- **持久化：Durable**

![image-20230207171531695](images/image-20230207171531695.png)

#### 3.消息实现持久化

修改生产者代码，发送时第三个参数为：`MessageProperties.PERSISTENT_TEXT_PLAIN`

```java
/*
 * 发送一个消费
 * 1.发送到哪个交换机
 * 2.路由的Key值是哪一个 本次是队列的名称
 * 3.表示其他参数信息
 * 4.发送消息的消息体
 */
channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
 ↓↓                           ↓↓                        ↓↓
channel.basicPublish("", QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
```

将消息标记为持久化并不能完全保证不会丢失消息。尽管它告诉 RabbitMQ 将消息保存到磁盘，但是这里依然存在当消息刚准备存储在磁盘的时候 但是还没有存储完，消息还在缓存的一个间隔点。此时并没有真正写入磁盘。持久性保证并不强，但是对于我们的简单任务队列而言，这已经绰绰有余了。

> 绝对持久化需要发布确认实现！

**队列+消息持久化代码实现**：

```java
// 声明队列
boolean durable = true; // 需要让Queue进行持久化  ←←←
channel.queueDeclare(TASK_QUEUE_NAME, durable, false, false, null);
Scanner scanner = new Scanner(System.in);
while (scanner.hasNext()) {
    String message = scanner.nextLine();
    // 设置生产者发送的消息为持久化消息（要求保存到磁盘上）保存在内存中  ←←←
    channel.basicPublish("", TASK_QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes("UTF-8"));
    System.out.println("生产者发出消息：" + message);
}
```

#### 4.不公平分发

在最开始的时候我们学习到 RabbitMQ 分发消息采用的**轮训分发**，但是在某种场景下这种策略并不是很好。

比方说有两个消费者在处理任务，**其中有个消费者 1 处理任务的速度非常快，而另外一个消费者 2处理速度却很慢**，这个时候我们还是采用轮训分发的化就会到这处理速度快的这个消费者很大一部分时间处于空闲状态，而处理慢的那个消费者一直在干活，这种分配方式在这种情况下其实就不太好，但是RabbitMQ 并不知道这种情况它依然很公平的进行分发。

**为了避免这种情况，我们可以设置参数信道`channel.basicQos(1)`为不公平信道（默认为0：公平轮询信道）**

两个消费者修改：

```java
// 设置不公平分发
channel.basicQos(1);

// 采用手动应答
boolean autoAck = false;
channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, cancelCallback);
```

测试情况：

```sh
# 生产者
AAA
生产者发出消息：AAA
BBB
生产者发出消息：BBB
CCC
生产者发出消息：CCC
# 消费者1
C1等待接收消息处理时间较短...
接收的消息：AAA
接收的消息：CCC
# 消费者2
C2等待接收消息处理时间较长...
接收的消息：BBB
```

#### 5.预取值

前面的消费者消费消息分为：轮询和不公平分发两种情况

这样不同消费者消费消息的权重会不一样，但是没有具体消费消息的具体量值，这里我们用到**预取值**`prefetch`来代表消费的消息量。

**提前在信道`channel`上设置预取值`prefetch`**:

![image-20230207175320614](images/image-20230207175320614.png)

代码实现：

```java
// 设置预取值为2
int prefetch = 2; // 设置预取值的数量
channel.basicQos(prefetch);

// 采用手动应答
boolean autoAck = false;
channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, cancelCallback);
```

![image-20230207181102970](images/image-20230207181102970.png)

#### 6.发布确认

前面我们设置了信道持久化和消息持久化，其实**真正的持久化是将数据保存到磁盘上**。

<u>如果在保存磁盘的过程中服务宕机，那么即使设置了信道消息持久化，也会丢失！</u>

所以必须要求做第三条：**发布确认**

> 发布确认：从生产者生产消息开始到队列中保存到磁盘后MQ向生产者发送确认消息的行为！

##### 首先开启发布确认

```java
Channel channel = RabbitMqUtils.getChannel();
channel.confirmSelect(); // 开启信道发布确认
```

##### 1.单个发布确认

这是一种简单的确认方式，它是一种**同步确认发布**的方式，也就是发布一个消息之后只有它被确认发布，后续的消息才能继续发布,waitForConfirmsOrDie(long)这个方法只有在消息被确认的时候才返回，如果在指定时间范围内这个消息没有被确认那么它将抛出异常。

这种确认方式有一个最大的缺点就是:**发布速度特别的慢，**因为如果没有确认发布的消息就会阻塞所有后续消息的发布，这种方式最多提供每秒不超过数百条发布消息的吞吐量。当然对于某些应用程序来说这可能已经足够了。

**代码**：

```java
// 单个确认 ---- 5551ms
public static void publishMessageIndividually() throws Exception {
    Channel channel = RabbitMqUtils.getChannel();
    // 声明队列
    String queueName = UUID.randomUUID().toString();
    // 配置信道，设置持久化 ←
    channel.queueDeclare(queueName, true, false, false, null);
    // 开启发布确认
    channel.confirmSelect();
    // 开始时间
    long start = System.currentTimeMillis();
    // 批量发消息
    for (int i = 0; i < MESSAGE_COUNT; i++) {
        String message = i + "";
        // 消息持久化 ←
        channel.basicPublish("", queueName, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes(StandardCharsets.UTF_8));
        // 单个消息就马上进行发布确认 ←←←←←←←←←
        boolean flag = channel.waitForConfirms();
        if (flag)
            System.out.println("消息发布成功！");
    }
    // 结束时间
    long end = System.currentTimeMillis();
    System.out.println("发布" + MESSAGE_COUNT + "条单独确认消息，耗时" + (end - start) + "ms");
}
```

##### 2.批量发布确认

单个发布确认方式非常慢，与单个等待确认消息相比，**先发布一批消息然后一起确认可以极大地提高吞吐量**。

当然这种方式的缺点就是:当发生故障导致发布出现问题时，**无法确认是哪个消息出现问题**，我们必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。当然这种方案仍然是同步的，也一样阻塞消息的发布。

**代码**：

```java
// 批量发布确认 ---- 412ms
public static void publishMessageBatch() throws Exception {
    Channel channel = RabbitMqUtils.getChannel();
    // 声明队列
    String queueName = UUID.randomUUID().toString();
    // 配置信道，设置持久化 ←
    channel.queueDeclare(queueName, true, false, false, null);
    // 开启发布确认
    channel.confirmSelect();
    // 开始时间
    long start = System.currentTimeMillis();
    // 批量发布消息的大小
    int batchSize = 100;
    // 批量发消息
    for (int i = 0; i < MESSAGE_COUNT; i++) {
        String message = i + "";
        // 消息持久化 ←
        channel.basicPublish("", queueName, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes(StandardCharsets.UTF_8));
        // 判断达到100条的时候，批量确认一次 ←←←←←←←←←
        if (i % batchSize == 0) {
            if (channel.waitForConfirms()) {
                System.out.println("批量确认成功！");
            }
        }
    }
    // 结束时间
    long end = System.currentTimeMillis();
    System.out.println("发布" + MESSAGE_COUNT + "条批量确认消息，耗时" + (end - start) + "ms");
}
```

##### 3.异步发布确认

异步确认虽然编程逻辑比上两个要**复杂**，但是**性价比最高**，无论是可靠性还是效率都很高。

他是利用**回调函数**来达到消息可靠性传递的，这个中间件也是通过函数回调来保证是否投递成功

![image-20230207185721790](images/image-20230207185721790.png)

使用监听器：

![image-20230207190526364](images/image-20230207190526364.png)

**代码**：

```java
// 异步发布确认 ---- 62ms
public static void publishMessageAsync() throws Exception {
    Channel channel = RabbitMqUtils.getChannel();
    // 声明队列
    String queueName = UUID.randomUUID().toString();
    // 配置信道，设置持久化 ←
    channel.queueDeclare(queueName, true, false, false, null);
    // 开启发布确认
    channel.confirmSelect();
    // 开始时间
    long start = System.currentTimeMillis();

    /*
      消息的监听器，监听哪些消息成功和失败(定义两个函数式接口)
      1.deliveryTag 消息的标记
      2.multiple 是否为批量确认
     */
    // 监听成功
    ConfirmCallback ackCallback = (deliveryTag, multiple) -> {
        System.out.println("确认的消息：" + deliveryTag);
    };
    // 监听失败
    ConfirmCallback nackCallback = (deliveryTag, multiple) -> {
        System.out.println("未确认的消息：" + deliveryTag);
    };
    /* ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓
      1.监听哪些消息成功了：ackCallback
      2.监听哪些消息失败了：nackCallback
     */
    channel.addConfirmListener(ackCallback, nackCallback); 
    // 批量发消息
    for (int i = 0; i < MESSAGE_COUNT; i++) {
        String message = i + "";
        // 消息持久化 ←
        channel.basicPublish("", queueName, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes(StandardCharsets.UTF_8));
    }
    // 结束时间
    long end = System.currentTimeMillis();
    System.out.println("发布" + MESSAGE_COUNT + "条异步确认消息，耗时" + (end - start) + "ms");
}
```

##### 如何处理异步未确认的消息？

1.需要准备在高并发下能记录当前发送的消息序号和内容：

```java
/*
   准备线程安全有序的哈希表，适用于高并发的情况
   1.轻松的将序号与消息进行关联
   2.轻松的批量删除条目，只要给到序号
   3.支持高并发（多线程）
 */
ConcurrentSkipListMap<Long, Object> outstandingConfirms =
        new ConcurrentSkipListMap<>();
```

2.在发送消息的for循环里将消息序号和内容put到Map中：

```java
for (int i = 0; i < MESSAGE_COUNT; i++) {
    String message = i + "";
    // 消息持久化
    channel.basicPublish("", queueName, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes(StandardCharsets.UTF_8));
    // 此处记录所有要发送的消息，消息的总和 ←←←←←←
    outstandingConfirms.put(channel.getNextPublishSeqNo(), message);
}
```

3.在监听成功（已确认）的函数式接口中调用Map删除成功的消息：

```java
// 分为单个和批量两种情况
// 监听成功
ConfirmCallback ackCallback = (deliveryTag, multiple) -> {
    // 删除已经确认的消息，剩下未确认的消息
    if (multiple) {
        ConcurrentNavigableMap<Long, Object> confirmed =
                outstandingConfirms.headMap(deliveryTag);
        confirmed.clear(); // 批量删除 ←←←←←←
    } else {
        outstandingConfirms.remove(deliveryTag); // 删除当前的 ←←←←←←
    }
    System.out.println("确认的消息：" + deliveryTag);
};
```

4.在监听失败（未确认）的函数式接口中调用Map打印失败的消息：

```java
// 监听失败
ConfirmCallback nackCallback = (deliveryTag, multiple) -> {
    // 打印未确认的消息
    String message = (String) outstandingConfirms.get(deliveryTag);
    System.out.println("未确认的消息是：" + message + "，未确认的消息tag：" + deliveryTag);
};
```

##### 三种发布确认方式效率对比

1. 单独发布确认消息

   同步等待确认，简单，但吞吐量非常有限，耗时

2. 批量发布确认消息

   批量同步等待确认，合理吞吐量，但是出了问题不知是哪条消息

3. 异步发布确认消息

   最佳性能的资源使用，出错可以控制，但难度较高

```java
public static void main(String[] args) throws Exception {
    // 1.单个确认
    publishMessageIndividually(); // 发布1000条单独确认消息，耗时5551ms
    // 2.批量确认
    publishMessageBatch(); // 发布1000条单独确认消息，耗时412ms
    // 3.异步确认
    publishMessageAsync(); // 发布1000条异步确认消息，耗时62ms
}
```

### <u>5.交换机(发布/订阅)</u>

之前我们利用工作队列将生产者生产的每条消息交付给一个消费者，现在我们将消息传达给多个消费者，此做法称为"发布/订阅"。

![image-20230208115930778](images/image-20230208115930778.png)

#### 1.Exchanges概念

RabbitMQ消息传递模型的核心思想是：**生产者生产的消息从不会直接发送到队列**，通常生产者甚至都不知道这些消息传递到了哪个队列中。

相反，**生产者只能将消息发送到交换机(exchange)**，交换机工作的内容非常简单，一方面它接收来自生产者的消息，另一方面将它们推入队列。交换机必须确切知道如何处理收到的消息。是应该把这些消息放到特定队列还是说把他们到许多队列中还是说应该丢弃它们。这就的由交换机的类型来决定。

#### 2.Exchange的类型

直接（direct）、主题（topic）、标题（headers）、扇出（fanout）

![image-20230208114656928](images/image-20230208114656928.png)

#### 3.默认Exchange

前面部分我们对 exchange 一无所知，但仍然能够将消息发送到队列。之前能实现的原因是因为我们使用的是默认交换，我们**通过空字符串("")进行标识**。

```java
/*
 * 发送一个消费
 * 1.发送到哪个交换机
 * 2.路由的Key值是哪一个 本次是队列的名称
 * 3.表示其他参数信息
 * 4.发送消息的消息体
 */
channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
```

> 空字符串表示默认或无名交换机，消息能路由发送到队列中其实是由 routingKey(bindingkey)绑定 key 指定的

#### 4.临时队列

队列的名称决定我们的消费者取消费哪个队列的消息。

每当我们连接到 Rabbit 时，我们都需要一个全新的空队列，为此我们可以创建一个具有**随机名称的队列**，或者能让服务器为我们选择一个随机队列名称那就更好了。其次**一旦我们断开了消费者的连接，队列将被自动删除。**

**创建临时队列**：

```java
String queueName = channel.queueDeclare().getQueue();
//名称：amq.gen-Gs4JTNmcpGg2qUariZynuQ
```

#### 5.绑定(Bindings)

**binding 其实是 exchange 和 queue 之间的桥梁**，它告诉我们 exchange 和那个队列进行了绑定关系。比如说下面这张图告诉我们的就是 X 与 Q1 和 Q2 进行了绑定:

![image-20230208121126573](images/image-20230208121126573.png)

**操作**：

![image-20230208121515466](images/image-20230208121515466.png)

> 绑定操作可以控制消息发给任意队列

#### 6.Fanout广播类型

正如从名称中猜到的那样，它是将接收到的所有消息**广播**到它知道的所有队列中。系统中默认有些交换机`amq.fanout`类型就是Fanout。

Logs 和临时队列的逻辑如下图:

![image-20230208122306390](images/image-20230208122306390.png)

Logs 和临时队列的绑定关系如下图:

![image-20230208122444462](images/image-20230208122444462.png)

> Routing Key并非为空，而是空字符串！

**实验代码**：

生产者将消息发送给指定的交换机logs

```java
public class EmitLog {

    // 交换机的名称
    public static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.nextLine();
            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
            System.out.println("生产者发出消息" + message);
        }
    }
}
```

消费者声明消费的交换机`logs`并指定类型`Fanout`进行等待消费

```java
public class ReceiveLogs1 {
    // 交换机的名称
    public static final String EXCHANGE_NAME = "logs";
    
    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMqUtils.getChannel();
        /*
          声明一个交换机
          1.交换机名称
          2.交换机类型
         */
        channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
        /*
          声明一个临时队列,名称随机
          当消费者断开与队列连接的时候，自动删除
         */
        String queueName = channel.queueDeclare().getQueue();
        /*
          绑定交换机与队列
          1.队列名
          2.交换机
          3.Routing Key（决定路由哪个队列）
         */
        channel.queueBind(queueName,EXCHANGE_NAME,"");
        System.out.println("等待接收消息,把接收到的消息打印在屏幕.....");

        // 声明接收消息
        DeliverCallback deliverCallback = (consumerTag, message)
                -> System.out.println("01在控制台打印接收到的消息："+new String(message.getBody()));
        // 取消消息时的回调
        CancelCallback cancelCallback = consumerTag -> System.out.println("消息消费被中断");

        /*
         * 消费者消费消息
         * 1.消费哪个队列
         * 2.消费成功之后是否自动应答 true/false
         * 3.消费者未成功消费的回调
         * 4.消费者取消消费的回调
         */
        channel.basicConsume(queueName,true,deliverCallback,cancelCallback);
    }
}
```

**实验结果**：生产者者发送的每一个消息通过`fanout`类型的交换机`logs`转发给队列（随机的，作用后自动删除），每个消费者都能消费到相同的消息，类似于广播效果！

```sh
# 生产者：
11
生产者发出消息11
22
生产者发出消息22
33
# 消费者01：
等待接收消息,把接收到的消息打印在屏幕.....
01在控制台打印接收到的消息：11
01在控制台打印接收到的消息：22
01在控制台打印接收到的消息：33
# 消费者02：
等待接收消息,把接收到的消息打印在屏幕.....
02在控制台打印接收到的消息：11
02在控制台打印接收到的消息：22
02在控制台打印接收到的消息：33
```

#### 7.Direct直接类型

队列只对它绑定的交换机的消息感兴趣，绑定时的参数`routingKey`也可以叫`bindingKey`，绑定之后的队列消息由交换机决定。

例如我们希望将日志消息写入磁盘的程序仅接收严重错误(errros)，而不存储那些警告(warning)或信息(info)日志消息避免浪费磁盘空间。Fanout 这种交换类型并不能给我们带来很大的灵活性-它只能进行无意识的广播，在这里我们将使用 direct 这种类型来进行替换，这种类型的工作方式是，**消息只去到它绑定的routingKey 队列中去**。

![image-20230209185822605](images/image-20230209185822605.png)

**实现多重绑定**：

如果 exchange 的绑定类型是 direct，**但是它绑定的多个队列的** **key** **如果都相同**，在这种情况下虽然绑定类型是 direct **但是它表现的就和** **fanout** **有点类似了**，就跟广播差不多。

**实验代码**：

生产者：

```java
public class EmitLogDirect {

    public static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        // 创建一个HashMap，封装routingKey和输出信息
        Map<String, String> bindingKeyMap = new HashMap<>();
        bindingKeyMap.put("info", "普通的info信息");
        bindingKeyMap.put("warning", "警告的warning信息");
        bindingKeyMap.put("error", "错误的error信息");
        bindingKeyMap.put("debug", "错误的debug信息");
        for (Map.Entry<String, String> bindingKeyEntry : bindingKeyMap.entrySet()) {// 遍历HashMap
            String bingingKey = bindingKeyEntry.getKey();
            String message = bindingKeyEntry.getValue();
            channel.basicPublish(EXCHANGE_NAME, bingingKey, null, message.getBytes("UTF-8"));
            System.out.println("生产者发出的消息：" + message);
        }
    }
}
```

消费者：

```java
public class ReceiveToDirectLogs1 {

    public static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        /*
          声明一个交换机
          1.交换机名称
          2.交换机类型
         */
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

        // 声明接收消息
        DeliverCallback deliverCallback = (consumerTag, message)
                -> System.out.println("在控制台打印接收到的消息："+new String(message.getBody()));// 获取消息体(如果不用消息体会输出被处理的消息对象例如：com.rabbitmq.client.Delivery@4039c79e)
        // 取消消息时的回调
        CancelCallback cancelCallback = consumerTag -> System.out.println("消息消费被中断");

        // 创建队列并与Direct交换机进行绑定，设置routingKey
        channel.queueDeclare("console", true, false, false, null);
        channel.queueBind("console", EXCHANGE_NAME, "info");
        channel.queueBind("console", EXCHANGE_NAME, "warning");
        channel.basicConsume("console",true,deliverCallback,cancelCallback);
    }
}
```

**实现结果**：消费者绑定指定的`Direct Exchange`交换机后，可以设置根据的`routingKey`来连接队列获取指定的消息进行消费！

```sh
# 生产者
生产者发出的消息：错误的debug信息
生产者发出的消息：警告的warning信息
生产者发出的消息：错误的error信息
生产者发出的消息：普通的info信息
# 消费者1
在控制台打印接收到的消息：警告的warning信息
在控制台打印接收到的消息：普通的info信息
# 消费者2
在控制台打印接收到的消息：错误的error信息
```

#### 8.Topic主题类型

尽管使用 direct 交换机改进了我们的系统，但是它仍然存在局限性-比方说我们想接收的日志类型有`info.base` 和 `info.advantage`，某个队列只想 `info.base` 的消息，那这个时候 direct 就办不到了。这个时候就只能使用 topic 类型。

**要求**：

发送到类型是 topic 交换机的消息的 routing_key 不能随意写，必须满足一定的要求，它**必须是一个单词列表，以点号分隔开**。这些单词可以是任意单词，比如说："stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit".这种类型的。当然这个单词列表最多不能超过 255 个字节。

在这个规则列表中，其中有两个替换符是大家需要注意的：

1. **(*)可以代替1个单词**
2. **(#)可以代替0个或多个单词**

**示例**：

1. Q1-->绑定的是：

   - 中间带 orange 带 3 个单词的字符串(*.orange.*)

2. Q2-->绑定的是：

   - 最后一个单词是 rabbit 的 3 个单词(*.*.rabbit)

   - 第一个单词是 lazy 的多个单词(lazy.#)

![image-20230209195927383](images/image-20230209195927383.png)

**特殊routingKey**：

当队列绑定关系是下列这种情况时需要引起注意:

1. 当一个队列绑定键是(#),那么这个队列将接收所有数据，就有点像 fanout 了
2. 如果队列绑定键当中没有(*#)和出现，那么该队列绑定类型就是direct了

**演示代码**：与`fanout`和`direct`类型的区别是`topic`绑定的`routingKey`匹配队列会更加的灵活可变！

生产者：

```java
public class EmitLogTopic {

    public static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        // 创建一个HashMap，封装routingKey和输出信息
        Map<String, String> bindingKeyMap = new HashMap<>();
        bindingKeyMap.put("quick.orange.rabbit","被队列 Q1Q2 接收到");
        bindingKeyMap.put("lazy.orange.elephant","被队列 Q1Q2 接收到");
        bindingKeyMap.put("quick.orange.fox","被队列 Q1 接收到");
        bindingKeyMap.put("lazy.brown.fox","被队列 Q2 接收到");
        bindingKeyMap.put("lazy.pink.rabbit","虽然满足两个绑定但只被队列 Q2 接收一次");
        bindingKeyMap.put("quick.brown.fox","不匹配任何绑定不会被任何队列接收到会被丢弃");
        bindingKeyMap.put("quick.orange.male.rabbit","是四个单词不匹配任何绑定会被丢弃");
        bindingKeyMap.put("lazy.orange.male.rabbit","是四个单词但匹配 Q2");
        for (Map.Entry<String, String> bindingKeyEntry : bindingKeyMap.entrySet()) {// 遍历HashMap
            String bingingKey = bindingKeyEntry.getKey();
            String message = bindingKeyEntry.getValue();
            channel.basicPublish(EXCHANGE_NAME, bingingKey, null, message.getBytes("UTF-8"));
            System.out.println("生产者发出的消息：" + message);
        }
    }
}
```

消费者：

```java
public class ReceiveToTopicLogs1 {

    public static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        /*
          声明一个交换机
          1.交换机名称
          2.交换机类型
         */
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

        // 声明接收消息
        DeliverCallback deliverCallback = (consumerTag, message)
                -> System.out.println("在控制台打印接收到的消息："+new String(message.getBody()));// 获取消息体(如果不用消息体会输出被处理的消息对象例如：com.rabbitmq.client.Delivery@4039c79e)
        // 取消消息时的回调
        CancelCallback cancelCallback = consumerTag -> System.out.println("消息消费被中断");

        // 创建队列并与Direct交换机进行绑定，设置routingKey
        channel.queueDeclare("Q1", true, false, false, null);
        channel.queueBind("Q1", EXCHANGE_NAME, "*.orange.*");// 中间一个单词是orange的三个单词
        channel.basicConsume("Q1",true,deliverCallback,cancelCallback);
    }
}
```

**演示结果**：

```sh
# 生产者
生产者发出的消息：是四个单词不匹配任何绑定会被丢弃
生产者发出的消息：不匹配任何绑定不会被任何队列接收到会被丢弃
生产者发出的消息：被队列 Q1Q2 接收到
生产者发出的消息：被队列 Q2 接收到
生产者发出的消息：被队列 Q1Q2 接收到
生产者发出的消息：被队列 Q1 接收到
生产者发出的消息：虽然满足两个绑定但只被队列 Q2 接收一次
生产者发出的消息：是四个单词但匹配 Q2
# 消费者Q1
在控制台打印接收到的消息：被队列 Q1Q2 接收到
在控制台打印接收到的消息：被队列 Q1Q2 接收到
在控制台打印接收到的消息：被队列 Q1 接收到
# 消费者Q2
在控制台打印接收到的消息：被队列 Q1Q2 接收到
在控制台打印接收到的消息：被队列 Q2 接收到
在控制台打印接收到的消息：被队列 Q1Q2 接收到
在控制台打印接收到的消息：虽然满足两个绑定但只被队列 Q2 接收一次
在控制台打印接收到的消息：是四个单词但匹配 Q2
```

### 6.死信队列

#### 1.死信的概念

死信，顾名思义就是无法被消费的消息，字面意思可以这样理解，一般来说，producer 将消息投递到 broker 或者直接到 queue 里了，consumer 从 queue 取出消息进行消费，但某些时候由于特定的**原因导致** **queue** **中的某些消息无法被消费**，这样的消息如果没有后续的处理，就变成了死信，有死信自然就有了死信队列。

**应用场景**:

为了保证订单业务的消息数据不丢失，需要使用到 RabbitMQ 的死信队列机制，当消息消费发生异常时，将消息投入死信队列中.还有比如说: 用户在商城下单成功并点击去支付后在指定时间未支付时自动失效

#### 2.死信的来源

1. 消息TTL过期
2. 队列达到最大长度
3. 消息被拒绝(basic.reject 或 basic.nack)并且 requeue=false(不放回队列)

#### 3.死信架构图

正常消息通过`direct`交换机分配给`zhangsan`队列给C1消费，因为**消息被拒**、**消息TTL过期**、**队列达到最大长度**导致该信息成为**死信**，使得另一个`direct`交换机交给了`lisi`死信队列处理。

![image-20230210154432462](images/image-20230210154432462.png)

#### 4.死信的实战

##### 1.消息TTL过期

> 重点：普通队列-->死信交换机-->死信队列

生产者：

```java
public class Producer {

    public static final String NORMAL_EXCHANGE = "normal_exchange";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMqUtils.getChannel();
        // 死信消息，设置TTL时间
        AMQP.BasicProperties properties = new AMQP.BasicProperties()
                .builder()
                .expiration("10000").build(); // 构建一个TTL时间参数10s
        for (int i = 0; i < 11; i++) {
            String message = "info"+i;
            channel.basicPublish(NORMAL_EXCHANGE,"zhangsan",null,message.getBytes());
        }
    }
}
```

普通消费者(连接死信交换机和死信队列)：

```java
public class Consumer1 {
    // 普通交换机的名称
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    // 死信交换机
    public static final String DEAD_EXCHANGE = "dead_exchange";
    // 普通队列的名称
    public static final String NORMAL_QUEUE = "normal_queue";
    // 死信队列
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMqUtils.getChannel();
        // 声明普通和死信交换机，类型为driect
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        // 声明死信和普通队列
        Map<String, Object> arguments = new HashMap<>();
        /*
          过期时间
          正常队列设置的死信交换机是谁？
          正常队列设置的死信队列是谁？
         */
        arguments.put("x-message-ttl",100000);// 可以不指定(通常在生产者发送消息时就已经指定了过期时间--可以任意修改)
        arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);// 死信交换机
        arguments.put("x-dead-letter-routing-key","lisi");// 死信routingKey以及队列名
        channel.queueDeclare(NORMAL_QUEUE,false,false,false,arguments);// 普通队列
        channel.queueDeclare(DEAD_QUEUE,false,false,false,null);// 死信队列

        // 绑定普通交换机和队列
        channel.queueBind(NORMAL_QUEUE,NORMAL_EXCHANGE,"zhangsan");
        // 绑定死信交换机和队列
        channel.queueBind(DEAD_QUEUE,DEAD_EXCHANGE,"lisi");
        System.out.println("等待接收消息.....");

        // 声明接收消息
        DeliverCallback deliverCallback = (consumerTag, message)
                -> System.out.println("Consumer1接收到的消息："+new String(message.getBody()));
        // 取消消息时的回调
        CancelCallback cancelCallback = consumerTag -> System.out.println("消息消费被中断");

     channel.basicConsume(NORMAL_QUEUE,true,deliverCallback,cancelCallback);
    }
}
```

死信消费者（消费死信队列的消息）：

```java
public class Consumer2 {
    // 死信队列
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("等待接收消息.....");

        // 声明接收消息
        DeliverCallback deliverCallback = (consumerTag, message)
                -> System.out.println("Consumer1接收到的消息：" + new String(message.getBody()));// 获取消息体(如果不用消息体会输出被处理的消息对象例如：com.rabbitmq.client.Delivery@4039c79e)
        // 取消消息时的回调
        CancelCallback cancelCallback = consumerTag -> System.out.println("消息消费被中断");

        channel.basicConsume(DEAD_QUEUE, true, deliverCallback, cancelCallback);
    }
}
```

**演示结果**：

1. 普通队列创建后，会**绑定与普通交换机**的关系，**连接死信交换机**，绑定死信交换机与死信队列的关系

2. 生产者负责发送消息给普通队列，**设置消息的TTL**过期时间为10秒

3. 消息过期后，普通队列将消息由连接的死信交换机进而发给死信队列

   ![image-20230210164421366](images/image-20230210164421366.png)

4. 死信消费者连接死信队列，消费里面的消息

   ![image-20230210164438752](images/image-20230210164438752.png)

##### 2.超出队列最大长度

1. 删除生产者为消息设置的TTL过期时间

2. 修改消费者绑定的队列

   ```java
   arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);// 死信交换机
   arguments.put("x-dead-letter-routing-key","lisi");// 死信routingKey以及队列名
   arguments.put("x-max-length",6);// 设置普通队列最多6条消息，多余为死信 ←←←←
   channel.queueDeclare(NORMAL_QUEUE,false,false,false,arguments);// 普通队列
   ```

**演示结果**：

![image-20230210165654589](images/image-20230210165654589.png)

##### 3.拒绝消息

1. 生产者正常发送消息（无TTL）

2. 消费者不设置队列长度

3. 消费者设置手动应答，并添加接收消息的条件

   ```java
   // 声明接收消息
   DeliverCallback deliverCallback = (consumerTag, message)
           -> {
       String msg = new String(message.getBody());
       if (msg.equals("info5")) {
           System.out.println("Consumer1接收的消息是：" + msg + ";此消息被C1拒绝了！");
           channel.basicReject(message.getEnvelope().getDeliveryTag(), false);
       } else {
           System.out.println("Consumer1接收到的消息是：" + msg);
           channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
       }
   };
   CancelCallback cancelCallback = consumerTag -> System.out.println("消息消费被中断");
   
   // 开启手动应答
   channel.basicConsume(NORMAL_QUEUE, false, deliverCallback, cancelCallback);
   ```

**演示结果**：

![image-20230210170857874](images/image-20230210170857874.png)

### 7.延迟队列

#### 1.概念

延时队列,队列内部是有序的，最重要的特性就体现在它的延时属性上，延时队列中的元素是希望在指定时间到了以后或之前取出和处理，简单来说，延时队列就是用来**存放需要在指定时间被处理的元素的队列**。

#### 2.使用场景

1. 订单在十分钟之内未支付则自动取消
2. 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。
3. 用户注册成功后，如果三天内没有登陆则进行短信提醒。
4. 用户发起退款，如果三天内没有得到处理则通知相关运营人员。
5. 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议

**业务示例**：

这些场景都有一个特点，**需要在某个事件发生之后或者之前的指定时间点完成某一项任务**，如：

发生订单生成事件，在十分钟之后检查该订单支付状态，然后将未支付的订单进行关闭；看起来似乎使用定时任务，**一直轮询数据，每秒查一次**，取出需要被处理的数据，然后处理。如果数据量比较少，确实可以这样做，比如：对于“如果账单一周内未支付则进行自动结算”这样的需求，如果对于时间不是严格限制，而是宽松意义上的一周，那么每天晚上跑个定时任务检查一下所有未支付的账单，确实也是一个可行的方案。

但对于数据量比较大，并且时效性较强的场景，如：“订单十分钟内未支付则关闭“，短期内未支付的订单数据可能会有很多，活动期间甚至会达到百万甚至千万级别，**对这么庞大的数据量仍旧使用轮询的方式显然是不可取的，很可能在一秒内无法完成所有订单的检查，同时会给数据库带来很大压力，无法满足业务要求而且性能低下**。

**订单业务逻辑**：

![image-20230210174853530](images/image-20230210174853530.png)

#### 3.整合SpringBoot

##### 1.依赖

```xml
<dependencies>
    <!--RabbitMQ 依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <!--Web基础框架-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--SpringBoot测试框架-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <!--转换JSON框架-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.47</version>
    </dependency>
    <!--日志框架-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <!--swagger-->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.9.2</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.9.2</version>
    </dependency>
    <!--RabbitMQ 测试依赖-->
    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

##### 2.yml文件配置

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    # 设置虚拟host 单机模式下固定编写"/"即可
    virtual-host: /
```

##### 3.交换机\路由Key\队列的配置类

RabbitMQ要求我们在java代码级别设置交换机\路由Key\队列的关系

我们在quartz包下,创建config包

包中创建配置信息类RabbitMQConfig

```java
// 当前配置类配置RabbitMQ中 交换机\路由Key和队列的关系
// 因为它们的关系需要保存到Spring容器中才能生效,所以需要这个配置类
@Configuration
public class RabbitMQConfig {
    // 将业务中需要的所有交换机\路由Key\队列的名称都声明为常量
    public static final String STOCK_EX="stock_ex";
    public static final String STOCK_ROUT="stock_rout";
    public static final String STOCK_QUEUE="stock_queue";

    // 绑定关系中,交换机和队列是实际对象,直接实例化保存到Spring容器
    @Bean
    public DirectExchange stockDirectExchange(){
        return new DirectExchange(STOCK_EX);
    }
    @Bean
    public Queue stockQueue(){
        return new Queue(STOCK_QUEUE);
    }
    // 路由Key是关系对象,保存方式特殊
    @Bean
    public Binding stockBinding(){
        return BindingBuilder.bind(stockQueue())
                            .to(stockDirectExchange()).with(STOCK_ROUT);
    }

}
```

##### 4.RabbitMQ发送消息

我们在QuartzJob类中输出时间的代码后继续编写代码

实现RabbitMQ消息的发送

```java
@Slf4j
public class QuartzJob implements Job {

    // 装配能够向RabbitMQ发送消息的对象
    // 这个对象也是添加好依赖和配置之后,在springBoot启动时自动向容器中保存的
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    static int i=1;
    // 这个方法就是当前job要定时执行的任务代码
    @Override
    public void execute(JobExecutionContext jobExecutionContext)
            throws JobExecutionException {
        // 一个简单的任务演示,输出当前系统时间,使用sout或log皆可
        log.info("-------------------" + LocalDateTime.now() + "--------------------");
        // 实例化Stock对象用于发送
        Stock stock=new Stock();
        stock.setId(i++);
        stock.setCommodityCode("PC100");
        stock.setReduceCount(1+ RandomUtils.nextInt(20));
        // 下面开始发送消息的RabbitMQ
        rabbitTemplate.convertAndSend(
                RabbitMQConfig.STOCK_EX,
                RabbitMQConfig.STOCK_ROUT,
                stock);
        log.info("发送消息完成:{}",stock);
    }
}
```

我们可以通过修改QuartzConfig类中的Cron表达式修改调用的周期**

```java
CronScheduleBuilder cron=
        CronScheduleBuilder.cronSchedule("0/10 * * * * ?");
```

按上面的cron修改之后,会每隔10秒运行一次发送消息的操作

启动服务,观察是否每隔10秒发送一条消息

启动Nacos\RabbitMQ\Seata

启动stock-webapi

根据Cron表达式,消息会在0/10/20/30/40/50秒数时运行

##### 5.接收消息

quartz包下再创建一个新的类用于接收信息

RabbitMQConsumer代码如下

```java
// Spring连接RabbitMQ的依赖中提供的资源,需要接收消息的类保存到Spring容器中才能使用
@Component
// 和Kafka不同,RabbitMQ监听器注解要求写在类上
@RabbitListener(queues = RabbitMQConfig.STOCK_QUEUE)
@Slf4j
public class RabbitMQConsumer {

    // 类上添加了监听器注解,但是不能指定接收消息后要运行的方法
    // 使用@RabbitHandler注解标记,我们接收到消息后要运行的方法
    // 每个类只允许一个方法被这个注解标记
    // 注解下面编写方法,参数比Kafka简单
    @RabbitHandler
    public void process(Stock stock){
        // Stock就是发送到RabbitMQ的消息,直接使用即可
        log.info("接收到消息:{}",stock);
    }
}
```
