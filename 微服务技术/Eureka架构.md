# Eureka架构

![image-20221124200851714](images/image-20221124200851714.png)

## 百科概念:

Eureka是[Netflix](https://baike.baidu.com/item/Netflix/662557?fromModule=lemma_inlink)开发的服务发现框架，本身是一个基于[REST](https://baike.baidu.com/item/REST/6330506?fromModule=lemma_inlink)的服务，主要用于定位运行在AWS域中的中间层服务，以达到负载均衡和中间层服务故障转移的目的。

SpringCloud将它集成在其子项目spring-cloud-netflix中，以实现SpringCloud的服务发现功能。

#### 服务调用出现的问题:

- 服务消费者该如何获取服务提供者的地址信息?
  - 服务提供者启动时向Eureka注册自己的信息
  - Eureka保存这些信息
  - 消费者根据服务名称向Eureka拉去提供者信息
- 如果有多个服务提供者,消费者该如何选择?
  - 服务消费者利用负载均衡算法,从服务列表中挑选一个
- 消费者如何得知服务提供者的健康状态?
  - 服务提供者会每隔30秒向EurekaServer发送心跳请求,报告健康状态
  - Eureka会更新记录列表信息,心跳不正常会被剔除
  - 消费者就可以拉取到最新的接口信息

#### Eureka的作用:

![image-20221124150705742](C:\Users\admin\Desktop\git本地仓库\Java_Important_Notes-WQY\微服务技术\images\image-20221124150705742.png)

> Eureka注册中心会将Service业务层服务提供的接口拿到保存并且每隔一定时间检查接口的状态,若某个接口不能使用时处于"非健康状态",会被Eureka从列表中剔除!
>
> 服务消费者需要某个接口时可以向Eureka中获取,当地址信息有多个时会通过负载均衡去选择,最后向服务提供者调用最终选择的接口!

## 总结:

#### 在Eureka架构中,微服务角色有两类:

- EurekaServer:服务端,注册中心
  - 记录服务信息
  - 心跳监控
- EurekaClient:客户端(这是一个基于java的客户端)
  - Provider:服务提供者,例如`user-service`
    - 注册自己的信息到EurekaServer
    - 每隔30秒向EurekaServer发送心跳
  - Consumer:服务消费者,例如`order-service`
    - 根据服务名称从EurekaServer拉去服务列表
    - 基于服务列表做负载均衡,选中一个微服务后发起远程调用

## Eureka服务

### 1.Eureka Service服务端,注册中心

#### 依赖:

```xml
<!--eureka服务端-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

#### 注解:

```java
package cn.itcast.eureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
// ↓↓↓↓↓↓↓↓↓↓↓↓↓↓
@EnableEurekaServer// 在main函数启动类上添加
@SpringBootApplication
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

#### 配置:

```yaml
server:
  port: 10086 # 服务端口
spring:
  application:
    name: eurekaserver # eureka的服务名称
eureka:
  client:
    service-url:  # eureka的地址信息
      defaultZone: http://127.0.0.1:10086/eureka
```

##### 最后访问:http://localhost:10086/ 即可开启Eureka服务

![image-20221124153631992](C:\Users\admin\Desktop\git本地仓库\Java_Important_Notes-WQY\微服务技术\images\image-20221124153631992.png)

> ##### 在Instances currently registered with Eureka下可以观察到注册过来的服务端接口和信息

### 2.Eureka Client客户端

#### 依赖:

```xml
<!--eureka客户端依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### 配置:

```yaml
server:
  port: 10086 # 服务端口
spring:
  application:
    name: userservice # user的服务名称
eureka:
  client:
    service-url:  # eureka的地址信息
      defaultZone: http://127.0.0.1:10086/eureka
```

#### 模拟多个启动实例:

![image-20221124154914329](C:\Users\admin\Desktop\git本地仓库\Java_Important_Notes-WQY\微服务技术\images\image-20221124154914329.png)

>在Environment下的VM options中配置:-Dserver.port=8082  更改启动实例的端口号,防止冲突

#### ★客户端缓存机制

即使所有的Eureka Server都挂掉，客户端依然可以利用缓存中的信息消费其他服务的API。

### 3.服务发现----拉取和负载均衡

服务拉取是基于服务名称获取服务列表,然后在对服务列表做负载均衡

1.修改OrderService服务接口的代码,修改访问的url路径,用服务名代替ip端口:

```java
String url = "http://userservice/user/"+order.getUserId();
```

2.在order-service项目的启动类OrderApplication中的RestTemplate添加**负载均衡**注解:

```java
@Bean
@LoadBalanced// 标记该请求会被Ribbon拦截处理进行负载均衡
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

### 4.Ribbon负载均衡

- ##### 负载均衡的原理流程

  ![image-20221124161155371](C:\Users\admin\Desktop\git本地仓库\Java_Important_Notes-WQY\微服务技术\images\image-20221124161155371.png)

  执行过程会被拦截并替换端口地址

  ![image-20221124161224941](C:\Users\admin\Desktop\git本地仓库\Java_Important_Notes-WQY\微服务技术\images\image-20221124161224941.png)

> #### 总体流程如下:

![image-20221124161714894](C:\Users\admin\Desktop\git本地仓库\Java_Important_Notes-WQY\微服务技术\images\image-20221124161714894.png)

### 5.IRule接口处理负载均衡规则

Ribbon的负载均衡规则是一个叫做IRule的接口来定义的,每一个子接口都是一种规则:

- 规则接口是IRule
- 默认实现的是ZoneAvoidanceRule,根据zone选择服务列表,然后轮询

![image-20221124163107474](C:\Users\admin\Desktop\git本地仓库\Java_Important_Notes-WQY\微服务技术\images\image-20221124163107474.png)

#### 负载均衡的策略:

![image-20221124163123646](C:\Users\admin\Desktop\git本地仓库\Java_Important_Notes-WQY\微服务技术\images\image-20221124163123646.png)

> #### 更换负载均衡的策略

通过定义IRule实现可以修改均衡规则,有两种方式:

1.代码方式:在order-service中的OrderApplication类中,定义一个新的IRule:

```java
@Bean// 配置负载均衡的规则---随机
public IRule randomRule() {
    return new RandomRule();
}
```

2.配置文件方式:在order-service的application.yml文件中,添加新的配置也可以修改规则:

```yaml
userservice:
  ribbon:
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule  # 负载均衡规则
```

#### 饥饿加载:

Ribbon默认是采用懒加载,即第一次访问时才会去创建LoadBalanceClient,请求时间会很长,而懒加载则会在项目启动时创建,降低第一次访问的耗时,通过下面配置开启饥饿加载:

```yaml
ribbon:
  eager-load:
    enabled: true # 开启饥饿加载
    clients: # List集合类型---可指定饥饿加载的微服务名称
      - userservice
```
