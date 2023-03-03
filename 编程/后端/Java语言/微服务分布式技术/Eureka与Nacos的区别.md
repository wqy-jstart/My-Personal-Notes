# Eureka与Nacos的区别

### 1.CAP理论

**C一致性,A高可用,P分区容错性**

1. eureka只支持AP

2. nacos支持CP和AP两种

nacos是根据配置识别CP或AP模式,如果注册Nacos的client节点注册时是ephemeral=true即为临时节点,那么Naocs集群对这个client节点效果就是AP,反之则是CP,即不是临时节点

```properties
#false为永久实例，true表示临时实例开启，注册为临时实例
spring.cloud.nacos.discovery.ephemeral=true
```

### 2.连接方式

1. nacs使用的是netty和服务直接进行连接,属于长连接
2. eureka是使用定时发送和服务进行联系,属于短连接

### 3.服务异常剔除

#### 1.eureka:

Eureka client在默认情况每隔30s想Eureka Server发送一次心跳,当Eureka Server在默认连续90s秒的情况下没有收到心跳, 会把Eureka client 从注册表中剔除,在由Eureka-Server 60秒的清除间隔,把Eureka client 给下线

```java
// EurekaInstanceConfigBean类下
private int leaseRenewalIntervalInSeconds = 30;  //心跳间隔30s
private int leaseExpirationDurationInSeconds = 90;  //默认90s没有收到心跳从注册表中剔除
EurekaServerConfigBean  类下
private long evictionIntervalTimerInMs = 60000L; //异常服务剔除下线时间间隔
```
也就是在极端情况下Eureka 服务 从异常到剔除在到完全不接受请求可能需要 30s+90s+60s=3分钟左右(还是未考虑ribbon缓存情况下)

#### 2.nacos:

nacos client 通过心跳上报方式告诉 nacos注册中心健康状态,默认心跳间隔5秒，nacos会在超过15秒未收到心跳后将实例设置为不健康状态，可以正常接收到请求超过30秒nacos将实例删除,不会再接收请求

### 4.操作实例方式

**nacos**:提供了nacos console可视化控制话界面,可以对实例列表进行监听,对实例进行上下线,权重的配置,并且config server提供了对服务实例提供配置中心,且可以对配置进行CRUD,版本管理

**eureka**:仅提供了实例列表,实例的状态,错误信息,相比于nacos过于简单

### 5.自我保护机制

#### 相同点：

保护阈值都是个比例，0-1 范围，表示健康的 instance 占全部instance 的比例。

#### 不同点：

##### 1）保护方式不同

Eureka保护方式：当在短时间内，统计续约失败的比例，如果达到一定阈值，则会触发自我保护的机制，在该机制下，Eureka Server不会剔除任何的微服务，等到正常后，再退出自我保护机制。自我保护开关(eureka.server.enable-self-preservation: false)

Nacos保护方式：当域名健康实例 (Instance) 占总服务实例(Instance) 的比例小于阈值时，无论实例 (Instance) 是否健康，都会将这个实例 (Instance) 返回给客户端。这样做虽然损失了一部分流量，但是保证了集群的剩余健康实例 (Instance) 能正常工作。

##### 2）范围不同

Nacos 的阈值是针对某个具体 Service 的，而不是针对所有服务的。但 Eureka的自我保护阈值是针对所有服务的。