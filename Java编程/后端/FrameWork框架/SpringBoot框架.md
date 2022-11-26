# SpringBoot框架

## 1. Spring Boot框架的作用

Spring Boot框架主要解决了统一管理依赖项与简化配置相关的问题。

**统一管理依赖项**：在开发实践中，项目中需要使用到的依赖项可能较多，在没有Spring Boot时，可能需要自行添加若干个依赖项，并且，需要自行保证这些依赖项的版本不会发生冲突，或保证不存在不兼容的问题，Spring Boot提供了`starter`依赖项（依赖项的名称中有`starter`字样），这种依赖项包含了使用某个框架时可能涉及的一系列依赖项，并处理好了各依赖项的版本相关问题，保证各依赖项的版本兼容、不冲突！例如`spring-boot-starter-web`就包含了Spring MVC的依赖项（`spring-webmvc`）、`jackson-databind`、`tomcat`等。

**简化配置**：Spring Boot默认完成了各项目中最可预测的配置，它是一种**“约定大于配置”**的思想，当然，这些配置也都是可以修改的，例如通过在`application.properties`中添加指定属性名的配置。

## 2. Spring Boot框架的依赖项

当项目中需要使用Spring Boot框架时，需要添加的基础依赖项是：`spring-boot-starter`。

此基础依赖项被其它各带有`starter`字样的依赖项所包含，所以，通常不必显式的添加此基础依赖项。

## 3. 典型的应用技巧

**关于日志**

在`spring-boot-starter`中已经包含`spring-boot-starter-logging`，所以，在添加任何`starter`依赖后，在项目中均可使用日志。

**关于配置文件**

Spring Boot项目默认在`src/main/resources`下已经准备好了配置文件，且Spring Boot默认识别的配置文件的文件名是`application`，其扩展名可以是`.properties`或`.yml`。

此配置文件会被Spring Boot自动读取，框架指定属性名称的各配置值会被自动应用，自定义的配置需要通过`Environment`对象或通过`@Value`注解自行读取并应用，所有不是指定属性名称的配置都是自定义配置。

其实，读取`.properties`配置文件是Spring框架做到的功能，而`.yml`配置是Spring框架并不支持的，但是，在Spring Boot中，添加了解析`.yml`文件的依赖项，所以，在Spring Boot中既可以使用`.properties`也可以使用`.yml`。

另外，Spring Boot还很好的支持了Profile配置（也是Spring框架做到的功能）。

------

------

## 注解:

| 注解                       | 所属框架    | 作用                                                         |
| -------------------------- | ----------- | ------------------------------------------------------------ |
| `@SpringBootApplication`   | Spring Boot | 添加在类上，用于标记此类是Spring Boot的启动类，每个Spring Boot项目应该只有1个类添加了此注解 |
| `@SpringBootConfiguration` | Spring Boot | 通常不需要显式的使用，它是`@SpringBootApplication`的元注解之一 |
| `@SpringBootTest`          | Spring Boot | 添加在类上，用于标记此类是加载Spring环境的测试类             |
