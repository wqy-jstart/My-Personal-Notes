# 第二阶段Web&集合

## 集合

### 1.HashMap的底层原理

- HashMap底层是一个散列桶数组，每个散列桶是一个单向链表结构
- 散列桶容量从16开始，每次扩容2倍
- 当出现散列冲突时，散列桶中元素形成链表结构。当散列表容量大于64，桶中元素超过8个时，会转换为红黑树，提升查询性能。
- 查找的时候，通过散列算法，计算到散列桶的位置，直接定位
- 散列表加载因子是75%，最多75%充满率，出现散列冲突机会少，严重散列冲突还有红黑树优化，在散列桶中查询性能很快。

### 2.Java8中的HashMap与Java7的区别？

1. Java7采用：散列桶数组+链表实现，Java7以前，Java很少管理超过2GB内存，散列数组很少会达到数组上限，利用适当数组扩容来减少散列冲突出现的链表长度问题，可以解决链表查询性能问题
2. Java8采用：散列数组+链表/红黑树实现，Java8是大数据时代，不能光通过扩容解决散列冲突问题，Java8的优化引入红黑树，长度超过8节点转换红黑树

### 3.HashMap的遍历方式

- iterator遍历

  - entrySet

    ```java
    Iterator<Map.Entry<String, String>> iterator = hashMap.entrySet().iterator();
    while (iterator.hasNext()) {
        Map.Entry<String, String> entry = iterator.next();
        String key = entry.getKey();
        String value = entry.getValue();
        System.out.println(key + ":" + value);
    }
    ```

  - keySet

    ```java
    Iterator<String> iterator1 = hashMap.keySet().iterator();
    while (iterator1.hasNext()) {
        String key = iterator1.next();
        String value = hashMap.get(key);
        System.out.println(key + ":" + value);
    }
    ```

- for遍历

  - entrySet

    ```java
    for (Map.Entry<String, String> entry : hashMap.entrySet()) {
        String key = entry.getKey();
        String value = entry.getValue();
        System.out.println(key + ":" + value);
    }
    ```

  - ketSet

    ```java
    for (String key : hashMap.keySet()) {
        String value = hashMap.get(key);
        System.out.println(key + ":" + value);
    }
    ```

- Lambda+foreach遍历

  ```java
  hashMap.forEach((key,value)->{
      System.out.println(key + ":" + value);
  })
  ```

- Streams API

  - 单线程

    ```java
    hashMap.entrySet().stream().forEach((entry) -> {
        String key = entry.getKey();
        String value = entry.getValue();
        System.out.println(key + ":" + value);
    });
    ```

  - 多线程

    ```java
    hashMap.entrySet().stream().parallel().forEach((entry) -> {
        String key = entry.getKey();
        String value = entry.getValue();
        System.out.println(key + ":" + value);
    });
    ```

### 4.HashMap和HashTable的区别

- 线程是否安全：HashMap不安全、HashTable安全
- 效率：HashMap>HashTable
- 对Null的支持：HashMap可以存Null值Null键，HashTable不能存Null值Null键，会报空指针

### 5.Map、List、Set的区别

1. List线性表，有序可重复的集合，按照位置访问元素
2. Set不允许元素重复，可以存储一个Null
3. Map按照K、V存储，元素可以重复，Key不能重复，按照Key查Value性能好

### 6.HashMap、LinkedHashMap、TreeMap区别

1. HashMap是散列表，查询效率高
2. LinkedHashMap继承HashMap，对Entry集合添加了一个双向链表
3. TreeMap内部红黑树，按照Key排序

### 7.Hash碰撞？

散列计算的时候，当两个不同的Key.hashCode(),计算出相同的散列值的现象，成为散列冲突（哈希冲突）

HashMap中解决办法是利用链表结构存储，超过8个元素转换为红黑树，或者选择扩容也可以减少Hash冲突

## JUC

**J**ava **U**til **C**oncurrent ：Java线程工具包

### 1.线程池有几种？

Java通过Executors(JUC)提供四种线程池，分别为：

1. newCachedThreadPool 创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活 回收空闲线程，若无可回收，则新建线程。
2. newFixedThreadPool创建一个定长线程池，可以控制最大并发数，提供队列等待（JavaSE使用了）
3. newScheduledThreadPool创建定长线程池，支持定时及周期性任务执行
4. newSingleThreadPool创建单线程化线程池，用唯一工作线程处理任务，保证任务顺序(FIFO、LIFO、优先级)执行

> **FIFO**：First in, First out--先进先出。
>
> **LIFO**：Last in First out--后进先出。

### 2.线程池参数？

- corePoolSize核心线程大小
- maximumPoolSize线程池最大数量
- keepAliveTime空闲线程存活时间
- unit空间线程存活时间单位
- workQueue工作队列
- threadFactory线程工厂
- handler拒绝策略

## NetWork

### 1.HTTP和HTTPS的区别

HTTP协议：

**H**yper **T**ext **T**ransfer **P**rotocol超文本传输协议，从Web服务器传输HTML到本地浏览器的传送协议。

HTTPS协议：

基于HTTP通道的安全协议：**HTTP+SSL（会话层）=HTTPS**

https协议需要申请证书，可能会花费一定费用

**安全性**：

1. http协议将数据以明文进行传输，不安全
2. https协议使用SSL/TLS加密传输，安全

**默认端口**：

1. http协议的默认端口是：80
2. https协议的默认端口是：443

### 2.TCP和UDP的区别

1. TCP基于连接，UDP基于无连接
2. TCP要求系统资源较多，UDP较少
3. TCP可靠，UDP不可靠（会丢包）
4. TCP可保证顺序，UDP不保证顺序

**常见网络通讯协议**：

TCP/IP是一个协议组，可分为四个层次：

1. 网络接口层
2. 网络层：IP(网络互连协议)、ICMP(控制报文协议)、ARP(地址解析协议)、RARP(反向ARP)、BOOTP(引导协议)
3. 传输层：TCP、UDP
4. 应用层：FTP、HTTP、TELNET、DNS等

### 3.HTTP请求结构

一个HTTP请求报文由四个部分构成：请求行、请求头、空行、请求数据

请求头（消息头）：

1. Accept：浏览器可接受的MIME类型
2. Accept-CharSet：浏览器可接受的字符集
3. Accept-Encoding：浏览器可接受的编码
4. Content-Length：表示请求消息的长度
5. Host：请求主机名
6. Cookies：存储在客户端的数据，通常比较重要，例如：账密、Token
7. Auth：网络信息认证

### 4.Get和Post

**Get请求**：

1. 回退无害
2. 将用户信息以地址栏传输
3. 会被浏览器主动缓存
4. 只能进行url编码
5. 长度限制
6. 可以被收藏和分享
7. 请求速度快

**Post请求**：

1. 将参数在body中传送
2. 不被浏览器主动缓存，除非手动设置
3. 没有长度限制
4. 支持多编码
5. 请求速度慢
6. 不能收藏和分享