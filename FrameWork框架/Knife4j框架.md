# Knife4j框架

### Knife4j是一款基于Swagger 2的在线API文档框架。

#### 使用Knife4j需要：

- ##### 添加依赖，注意：本次使用的Knife4j的版本必须基于Spring Boot的版本在2.6之前（2.6及更高版本不可用）

  ```xml
  <!-- Knife4j Spring Boot：在线API -->
  <dependency>
      <groupId>com.github.xiaoymin</groupId>
      <artifactId>knife4j-spring-boot-starter</artifactId>
      <version>2.0.9</version>
  </dependency>
  ```

- ##### 需要在主配置文件（`application.properties`或`application.yml`）中添加配置：

  ```properties
  knife4j.enable=true
  ```

- ##### 需要添加配置类，配置类代码为（注意：可能需要修改包名）：

  ```java
  package cn.tedu.csmall.product.config;
  
  import com.github.xiaoymin.knife4j.spring.extension.OpenApiExtensionResolver;
  import lombok.extern.slf4j.Slf4j;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import springfox.documentation.builders.ApiInfoBuilder;
  import springfox.documentation.builders.PathSelectors;
  import springfox.documentation.builders.RequestHandlerSelectors;
  import springfox.documentation.service.ApiInfo;
  import springfox.documentation.service.Contact;
  import springfox.documentation.spi.DocumentationType;
  import springfox.documentation.spring.web.plugins.Docket;
  import springfox.documentation.swagger2.annotations.EnableSwagger2WebMvc;
  
  /**
   * Knife4j配置类
   *
   * @author java@tedu.cn
   * @version 0.0.1
   */
  @Slf4j
  @Configuration
  @EnableSwagger2WebMvc
  public class Knife4jConfiguration {
  
      /**
       * 【重要】指定Controller包路径
       */
      private String basePackage = "cn.tedu.csmall.product.controller";
      /**
       * 分组名称
       */
      private String groupName = "product";
      /**
       * 主机名
       */
      private String host = "http://java.tedu.cn";
      /**
       * 标题
       */
      private String title = "酷鲨商城在线API文档--商品管理";
      /**
       * 简介
       */
      private String description = "酷鲨商城在线API文档--商品管理";
      /**
       * 服务条款URL
       */
      private String termsOfServiceUrl = "http://www.apache.org/licenses/LICENSE-2.0";
      /**
       * 联系人
       */
      private String contactName = "Java教学研发部";
      /**
       * 联系网址
       */
      private String contactUrl = "http://java.tedu.cn";
      /**
       * 联系邮箱
       */
      private String contactEmail = "java@tedu.cn";
      /**
       * 版本号
       */
      private String version = "1.0.0";
  
      @Autowired
      private OpenApiExtensionResolver openApiExtensionResolver;
  
      public Knife4jConfiguration() {
          log.debug("创建配置类对象：Knife4jConfiguration");
      }
  
      @Bean
      public Docket docket() {
          String groupName = "1.0.0";
          Docket docket = new Docket(DocumentationType.SWAGGER_2)
                  .host(host)
                  .apiInfo(apiInfo())
                  .groupName(groupName)
                  .select()
                  .apis(RequestHandlerSelectors.basePackage(basePackage))
                  .paths(PathSelectors.any())
                  .build()
                  .extensions(openApiExtensionResolver.buildExtensions(groupName));
          return docket;
      }
  
      private ApiInfo apiInfo() {
          return new ApiInfoBuilder()
                  .title(title)
                  .description(description)
                  .termsOfServiceUrl(termsOfServiceUrl)
                  .contact(new Contact(contactName, contactUrl, contactEmail))
                  .version(version)
                  .build();
      }
  
  }
  ```

##### 完成后，重新启动项目，可以通过 http://localhost:8080/doc.html 查看在线API文档，并可使用其中的调用功能等。

------

------

# 相关注解:

### 1.@EnableSwagger2WebMvc注解

- ##### 该注解用来配置Knife4jConfiguration类中的代码

### 2.@Api注解

- 添加在控制器类上，通过此注解的`tags`属性，可以指定模块名称，并且，可以在模块名称前自行添加数字序号，以实现排序效果

- 框架会根据各控制器类上`@Api`注解的``tags`属性值进行升序排列

- ```java
  @Api(tags = "01.相册管理模块")
  ```

- ![image-20221027184731481](images/image-20221027184731481.png)

### 3.@ApiOperation注解

- ##### 添加在处理请求的方法上，通过此注解的`value`属性，可以指定业务名称

### 4.@ApiOperationSupport注解

- ##### 添加在处理请求的方法上，通过此注解的`order`属性（数值型），可以指定排序序号，框架会根据此属性值升序排列