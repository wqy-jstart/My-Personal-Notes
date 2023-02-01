# Spring Cloud

> #### 官网地址:https://spring.io/projects/spring-cloud

## 依赖:

```xml
<!-- springCloud -->
<properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>2020.0.3</spring-cloud.version>
</properties>
<dependencies>
    <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>${spring-cloud.version}</version>
    <type>pom</type>
    <scope>import</scope>
    </dependency>
</dependencies>
```

## 什么是SpringCloud

SpringCloud是由Spring提供的一套能够快速搭建微服务架构程序的**框架集**

框架集表示SpringCloud不是一个框架,而是很多框架的统称

SpringCloud就是为了搭建微服务架构项目出现的

有人将SpringCloud称之为"Spring全家桶",广义上指代Spring的所有产品

## Spring Cloud的内容

**内容的提供者**

* Spring自己提供的开发出来的框架或软件
* Netflix(奈非):早期的很长一段时间,提供了大量的微服务解决方案
* alibaba(阿里巴巴):新版本的SpringCloudAlibaba正在迅速占领市场(推荐使用)

课程中使用全套的阿里巴巴组件

**功能上分类**

* 微服务的注册中心
* 微服务间的调用
* 微服务的分布式事务
* 微服务的限流
* 微服务的网关
* ....

------

#### SpringCloud是目前国内使用最广泛的微服务框架

SpringCloud集成了各种微服务功能组件,并基于SpringBoot实现了这些组件的自动装配,从而提供了良好的开箱即用的体验:

![image-20221124140034979](images/image-20221124140034979.png)

### 跨服务的远程调用控制器

先在启动类中创建`RestTemplate`并添加`@Bean`注解

```java
/**
 * 创建RestTemplate并注入Spring容器
 */
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

在Service层使用`@Autowired`进行装配,调用`getForObject()`方法,传入url和返回值类型即可

```java
@Autowired
private RestTemplate restTemplate;

public Order queryOrderById(Long orderId) {
    // 1.查询订单
    Order order = orderMapper.findById(orderId);
    // 2.利用RestTemplate发起http请求，查询用户
    // 2.1.url路径      ↓使用微服务的name做端口↓
    String url = "http://userservice/user/" + order.getUserId();
    // 2.2.发送http请求，实现远程调用
    User user = restTemplate.getForObject(url, User.class);
    // 3.封装user到Order
    order.setUser(user);
    // 4.返回
    return order;
}
```

**远程调用控制器示例**：

```java
// 装配RestTemplate调用远程方法
@Autowired
private RestTemplate restTemplate;

@PostMapping("/delete")
@ApiOperation("删除购物车中的商品")
@ApiImplicitParams({
        @ApiImplicitParam(value = "用户Id",name="userId",example = "UU100"),
        @ApiImplicitParam(value = "商品编号",name="commodityCode",example = "PC100")
})
public JsonResult deleteUserCart(String userId,String commodityCode){
    cartService.deleteUserCart(userId,commodityCode);
    // RestTemplate调用减少库存数的方法
    // 我们设计删除购物车之后减少库存,要调用RestTemplate首先确定url
    String url="http://localhost:20003/base/stock/reduce/count?" +
            "commodityCode={1}&reduceCount={2}";
    // 发起调用
    // getForObject方法参数和返回值的解释
    // 参数有3个部分
    // 1.第一个参数:请求的url,就是指定要访问的路径
    // 2.第二个参数:是返回值类型的反射,根据要求编写在参数位置即可
    // 3.从第三个参数开始,往后的每一个参数都是在给url中{x}的占位符赋值
    //    赋值规则:第三个参数赋值给{1}.第四个参数赋值给{2} ...以此类推
    JsonResult jsonResult = restTemplate
            .getForObject(url, JsonResult.class, commodityCode, 5);
    System.out.println(jsonResult);
    return JsonResult.ok("删除购物车完成!");
}
```

最后通过浏览器搜索即可获得远程微服务的信息:

![image-20221125091624384](images/image-20221125091624384.png)
