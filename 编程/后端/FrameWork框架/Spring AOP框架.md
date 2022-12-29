# Spring AOP框架

#### AOP:  Aspect Oriented Programming 面向切面编程

注意：AOP并不是Spring框架特有的技术，只是Spring框架很好的支持了AOP。

AOP主要用于：**日志**、**安全**、**事务管理**、**自定义的业务规则**

#### 优点:

1. 基于AOP思想的一个框架,它的使用可以有效的减少项目中的重复代码,降低耦合度
2. 将业务逻辑的各个部分进行了隔离,提高开发人员的开发效率

在项目中，数据的处理流程大致是：

```
添加品牌：请求 ------> Controller ------> Service ------> Mapper ------> DB

添加类别：请求 ------> Controller ------> Service ------> Mapper ------> DB

删除品牌：请求 ------> Controller ------> Service ------> Mapper ------> DB
```

其实，各请求提交到服务器端后，数据的处理流程是相对固定的！

假设存在某个需求，无论是添加品牌、添加类别、删除品牌，或其它请求的处理，都需要在Service组件中执行相同的任务，应该如何处理？

例如，假设需要实现“统计所有Service中的业务方法的执行耗时”，首先，需要添加依赖项：

```xml
<!-- Spring Boot AOP的依赖项 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

然后，在根包下创建`aop.TimerAspect`类，作为切面类，必须添加`@Aspect`注解，由于是通过Spring来实现AOP，所以，此类还应该交由Spring管理，它应该是个组件类，则再补充添加`@Component`注解，并在类中自定义方法，且通过注解来配置方法何时执行：

```java
package cn.tedu.csmall.product.aop;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Slf4j
@Aspect
@Component
public class TimerAspect {

    public TimerAspect() {
        log.debug("创建切面对象：TimerAspect");
    }

    // @Around注解表示“包裹”，通常也称之为“环绕”
    // @Around注解中的execution内部配置表达式，以匹配上需要哪里执行切面代码
    // 表达式中，星号（*）是通配符，可匹配1次任意内容
    // 表达式中，2个连接的小数点（..）也是通配符，可匹配0~n次，只能用于包名和参数列表
    @Around("execution(* cn.tedu.csmall.product.service.*.*(..))")
    //                 ↑ 此星号表示需要匹配的方法的返回值类型
    //                   ↑ ---------- 根包 ----------- ↑
    //                                                  ↑ 类名
    //                                                    ↑ 方法名
    //                                                      ↑↑ 参数列表
    public Object a(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();

        // 注意：必须获取调用此方法的返回值，作为当前切面方法的返回值
        Object result = pjp.proceed(); // 相当于执行了匹配的方法，即业务方法

        long end = System.currentTimeMillis();
        log.debug("执行耗时：{}毫秒", end - start);

        return result;
    }
}
```

## 【AOP的核心概念】

连接点（JoinPoint）：数据处理过程中的某个时间节点，可能是调用了方法，或抛出了异常

切入点（PointCut）：选择一个或多个连接点的表达式

通知（Advice）：在选择到的连接点执行的代码

切面（Aspect）：是包含了切入点和通知的模块

####  【通知注解】

@Before注解：表示“在……之前”，且方法应该是无参数的

@After注解：表示“在……之后”，无论是否抛出异常，或是否返回结果，都会执行，且方法应该是无参数的

@AfterReturning注解：表示“在返回结果之后”，且方法的参数是JoinPoint和返回值

@AfterThrowing注解：表示“在抛出异常之后”，且方法的参数是JoinPoint和异常对象

@Around注解：表示“包裹”，通常也称之为“环绕”，且方法的参数是ProceedingJoinPoint

####  【大致执行流程】

```java
@Around开始
try {
    @Before
    表达式匹配的方法
    @AfterReturning
} catch (Throwable e) {
    @AfterThrowing
} finally {
    @After
}
@Around结束
```