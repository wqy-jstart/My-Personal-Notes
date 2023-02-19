# 第四阶段SSM框架

## Spring框架

核心价值：解决对象的创建和管理

### 1.Spring的两大核心技术

IOC和AOP

### 2.对IOC和AOP的理解

1. IOC/DI

   IOC名为控制反转，将对象的创建控制权交给SpringIOC容器

   DI名为依赖注入，处理容器中Bean与Bean之间的关系，完善或实现了IOC

2. AOP

   AOP是面向切面编程，是一种编程思想，对面向对象的一种补充，统一管理程序的横向关注点，统一处理横向关注点，场景：安全、日志、事务等

3. AOP的实现

   动态代理

### 3.AOP中常用的注解

- @Before(execution)：在方法执行前拦截
- @AfterReturning(execution)：在方法返回结束后拦截
- @AfterThrowing(execution)：在方法抛出异常时拦截
- @After(execution)：在方法结束后（不论是否异常）拦截
- @Around(execution)：可以使ProceedingJoinPoint参数来控制流程，在方法执行前拦截，可以在切面逻辑中手动释放拦截，也可以添加逻辑代码，在方法执行之后执行
- @PointCut：声明切入点

### 4.@Autowired和@Resource注解的区别？

- @Autowired属于Spring框架，默认使用类型注入，再使用名字注入
- @Resource是JavaX的注解，Spring支持@Resource，而@Resource先按照名字注入，而后再按类型注入

### 5.事务注解以及作用

@Transactional注解

可以标注在类或方法上

标注该注解的类或方法，默认会开启事务，spring底层的数据库管理器会将提交改为手动，默认情况下，如果出现RuntimeException，会自动回滚

声明式事务是通过SpringAOP实现的，底层依赖数据库事务

### 6.Spring的常用注解

#### 1.声明bean的注解：

- @Component：声明组件，项目启动加载
- @Service：业务层使用
- @Repository：数据访问层
- @Controller：控制器层

#### 2.注入bean的注解：

- @Autowired：Spring提供
- @Resource：JavaX提供

#### 3.配置类注解：

- @Configuration：声明当前类为配置类
- @Bean：注解方法，返回一个bean
- @ComponentScan：扫描Component扫描
- @EnableAspectJAutoProxy：开启Spring对AspectJ代理的支持
- @Scope：设置Bean的作用域（默认单例）
- @PostConstruct：由JSR-250提供，在构造函数执行完之后执行
- @PreDestory：由JSR-250提供，在Bean销毁之前执行
- @Value：为属性注入值
- @PropertySource：加载配置文件
- @EnableScheduling：在配置类上使用，开启计划任务的支持

#### 4.切面AOP注解：

- @Before(execution)：在方法执行前拦截
- @AfterReturning(execution)：在方法返回结束后拦截
- @AfterThrowing(execution)：在方法抛出异常时拦截
- @After(execution)：在方法结束后（不论是否异常）拦截
- @Around(execution)：可以使ProceedingJoinPoint参数来控制流程，在方法执行前拦截，可以在切面逻辑中手动释放拦截，也可以添加逻辑代码，在方法执行之后执行
- @PointCut：声明切入点

#### 5.测试注解：

- @SpringBootTest

### 7.依赖注入的三种方式？

1. 属性注入
2. 构造器注入
3. setter方法注入

### 8.Spring中Bean的作用域？

Spring框架支持一下五个作用域：

1. singleton（默认）
2. prototype（非单例）
3. request
4. session
5. global session

常用：singleton、prototype

request、session、global session在Web环境有效！

### 9.SpringMVC、SpringCloud、SpringBoot区别

- Spring是一个轻量级IOC和AOP的容器框架，是Spring全家桶的基石。
- SpringMVC是一款基于MVC架构的Web框架，低耦合。
- Spring配置复杂，利用SpringBoot，约定大于配置，简化了配置流程做到开箱即用。
- SpringCloud是一个框架集，用于搭建微服务应用。

### 10.SpringBoot的作用

1. 以Spring框架为基础，快速搭建Spring应用程序，开箱即用
2. 提供统一的依赖管理，使用很少的代码整合各种框架
3. 内嵌servlet服务器，提供安全、健康检查等功能
4. 提供打包工具，便于部署或使用shell进行启动

## SpringMVC框架

核心价值：处理用户的请求和响应

1. 配置请求路径
2. 限制请求方式
3. 接收请求参数
4. 处理响应
5. 全局异常处理

### 1.MVC的设计理念

1. MVC是一种使用MVC(Model、View、Controller)设计的Web应用模式
2. Model表示应用核心，处理数据逻辑部分
3. View视图：显示数据
4. Controller：控制器处理请求与交互流程

### 2.SpringMVC的处理流程

1. 用户发送请求到达前端控制器DispatcherServlet
2. 调用处理器映射器HandlerMapping请求根据url映射对应处理器
3. 根据处理器获取处理器适配器HandlerAdapter进行参数封装、格式转换等操作
4. 执行处理器Controller
5. 执行完封装ModelAndView对象并返回
6. HandlerAdpater将处理返回的ModelAndView返回到前端控制器
7. 前端控制器将ModelAndView传给ViewReslover视图解析器
8. 解析后返回具体View
9. DispatcherServlet对View进行渲染（将Model模型数据填充到View中）
10. 响应给前端页面

### 3.常用注解

- @Controller：用于创建控制器对象，配合@RequestMapping接收前端的请求，配合@ResponseBody提供响应
- @RestController：标注类，不需每个方法都添加@ResponseBody
- @ResponseBody：将控制器方法的返回值绑定到HTTP的响应体中
- @RequestMapping：映射请求到控制器方法上
- @PostMapping、@GetMapping、@PutMapping、DeleteMapping
- @RequestBody：标注方法参数，接收JSON格式的对象
- @RequestParam：将请求参数映射到控制器的方法参数上
- @PathVariable：接收请求路径上的参数
- @ResponseStatus：设定HTTP响应状态码
- @ControllerAdvice：标注类，声明一个处理异常的类，配合@HandlerException和@ResponseBody返回异常结果
- @HandlerException：标注方法，声明处理异常的方法

## MyBatis框架

一款优秀的持久层框架，省去了95%的数据库编程底层的JDBC代码，提供强大的XML映射器、动态SQL、日志、缓存机制。

### 1.MyBatis中#{}和${}的区别

- #{}是预编译处理，${}是字符串替换，占位符
- MyBatis在处理#{}时，会将sql中的#{}替换为？，调用PreparedStatement方法来进行赋值
- MyBatis在处理${}时，进行替换变量的值，字符串拼接，容易造成SQL注入

### 2.MyBatis如何防止SQL注入？

1. #{}预编译
2. 将接收的单引号''替换成双引号

### 3.MyBatis的缓存机制

MyBatis有两层缓存，一级默认开启

一级缓存：`SqlSession`级别

要求：同一个SqlSession、同一个Mapper、相同SQL查询、查询参数相同

当出现写操作或者调用`clearCache()`方法时一级缓存会失效

二级缓存：`NameSpace`级别

要求：同一个Mapper、相同SQL查询、查询参数相同

默认不开启，需要在XML中添加`<cache>`标签开启，或者`<select>`标签配置`useCache=true`,需要保证查询结果对象实现了序列化接口

当出现写操作，缓存失效

### 4.单个参数或多个参数的声明

1. 单个参数直接定义
2. 多个参数使用@Param注解

### 5.常用注解：

- @Mapper：声明持久层
- @MapperScan：指定持久层所在的包
- @Insert：编写插入语句
- @Delete：编写删除语句
- @Update：编写更新语句
- @Select：编写查询语句
- @Param：映射查询参数