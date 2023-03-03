# Validation框架

## →该框架用于"检查客户端请求参数的基本格式"

## 1.添加依赖

- #### 在`pom.xml`中添加`spring-boot-starter-validation`依赖项：

```xml
<!-- Spring Boot Validation的依赖项，用于检查请求参数的基本格式 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

## 2. 检查封装的请求参数

- ##### 在控制器中，对于封装类型的请求参数，应该先在请求参数之前添加`@Valid`或`@Validated`注解，表示将需要对此请求参数的格式进行检查，例如：

```java
@ApiOperation("添加相册")
    @ApiOperationSupport(order = 100)
    @PostMapping("/add-new")
    //                     检查传递的参数
    //                       ↓↓↓↓↓↓
    public JsonResult addNew(@Valid AlbumAddNewDTO albumAddNewDTO) {
        log.debug("开始处理【添加相册】的请求，参数：{}", albumAddNewDTO);
        albumService.addNew(albumAddNewDTO);
        log.debug("添加数据成功!");
        return JsonResult.ok();//类名打点调用静态ok()方法
    }
```

- ##### 然后，在此封装类型中，在需要检查的属性上，添加检查注解，例如可以添加`@NotNull`注解，此注解表示“不允许为`null`值”，例如：

```java
@Data
public class AlbumAddNewDTO implements Serializable {

    /**
     * 相册名称
     */
    @ApiModelProperty(value = "相册名称", required = true)
    @NotNull // 新添加的注解
    private String name;
    
	// 暂不关心其它代码   
}
```

- ##### 重启项目，如果客户端提交请求时，未提交`name`请求参数，就会响应`400`错误，并且，在服务器端的控制台会提示错误：

```
2022-11-01 11:27:45.398  WARN 15104 --- [nio-9080-exec-3] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.validation.BindException: org.springframework.validation.BeanPropertyBindingResult: 1 errors<EOL>Field error in object 'albumAddNewDTO' on field 'name': rejected value [null]; codes [NotNull.albumAddNewDTO.name,NotNull.name,NotNull.java.lang.String,NotNull]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [albumAddNewDTO.name,name]; arguments []; default message [name]]; default message [不能为null]]
```

## 3. 处理检查不通过时的异常

- ##### 由于检查未通过时会抛出`org.springframework.validation.BindException`异常,则可以在全局异常处理器中，添加对此异常的处理，以避免响应`400`错误到客户端，而是改为响应一段JSON数据：

  ```java
  @ExceptionHandler
  public JsonResult handleBindException(BindException e) {
      log.debug("开始处理BindException");
  
      StringJoiner stringJoiner = new StringJoiner("，", "请求参数格式错误，", "！！！");
      List<FieldError> fieldErrors = e.getFieldErrors();
      for (FieldError fieldError : fieldErrors) {
          String defaultMessage = fieldError.getDefaultMessage();
          stringJoiner.add(defaultMessage);
      }
  
      return JsonResult.fail(ServiceCode.ERR_BAD_REQUEST, stringJoiner.toString());
  }
  ```

- ##### 当请求参数可能出现多种错误时，也可以选择“快速失败”的机制，它会使得框架只要发现错误，就停止检查其它规则，这需要在配置类中进行配置，则在项目的根包下创建`config.ValidationConfiguration`类并配置：

  ```java
  package cn.tedu.csmall.product.config;
  
  import cn.tedu.csmall.product.controller.AlbumController;
  import lombok.extern.slf4j.Slf4j;
  import org.hibernate.validator.HibernateValidator;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  import javax.validation.Validation;
  
  /**
   * Validation配置类
   *
   * @author java@wqy
   * @version 0.0.1
   */
  @Slf4j
  @Configuration
  public class ValidationConfiguration {
  
      public ValidationConfiguration() {
          log.debug("创建配置类对象：ValidationConfiguration");
      }
  
      @Bean
      public javax.validation.Validator validator() {
          return Validation.byProvider(HibernateValidator.class)
                  .configure() // 开始配置
                  .failFast(true) // 快速失败，即检查请求参数时，一旦发现某个参数不符合规则，则视为失败，并停止检查（剩余未检查的部分将不会被检查）
                  .buildValidatorFactory()
                  .getValidator();
      }
  }
  ```

- ##### 使用这种做法，当客户端提交的请求参数格式错误时，最多只会发现1种错误，则处理异常的代码可以调整为：

  ```java
  @ExceptionHandler
  public JsonResult handleBindException(BindException e) {
      log.debug("开始处理BindException");
      String defaultMessage = e.getFieldError().getDefaultMessage();
      return JsonResult.fail(ServiceCode.ERR_BAD_REQUEST, defaultMessage);
  }
  ```

## 4. 检查未封装的请求参数

#### 当处理请求的方法的参数是未封装的（例如`Long id`等），检查时，需要：

- 在当前控制器类上添加`@Validated`注解
- 在需要检查的请求参数上添加检查注解

##### 例如：

```java
@Slf4j
@Validated // 新添加的注解
@RestController
@RequestMapping("/albums")
@Api(tags = "04. 相册管理模块")
public class AlbumController {

    // 暂不关心其它代码
  
    // http://localhost:8080/albums/9527/delete
    @ApiOperation("根据id删除相册")
    @ApiOperationSupport(order = 200)
    @ApiImplicitParam(name = "id", value = "相册id", required = true, dataType = "long")
    @PostMapping("/{id:[0-9]+}/delete")
    //        新添加的注解 ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
    public String delete(@Range(min = 1, message = "删除相册失败，尝试删除的相册的ID无效！")
                             @PathVariable Long id) {
        String message = "尝试删除id值为【" + id + "】的相册";
        log.debug(message);
        return message;
    }

}
```

##### 当请求参数不符合以上`@Range(min =1)`的规则时（例如请求参数值为`0`或负数），默认情况下会出现`500`错误，在服务器端控制台可以看到以下异常信息：

```
javax.validation.ConstraintViolationException: delete.id: 删除相册失败，尝试删除的相册的ID无效！
```

##### 则需要在全局异常处理器中，添加新的处理异常的方法，用于处理以上异常：

```java
@ExceptionHandler
public JsonResult handleConstraintViolationException(ConstraintViolationException e) {
    log.debug("开始处理ConstraintViolationException");
    StringJoiner stringJoiner = new StringJoiner("，");
    Set<ConstraintViolation<?>> constraintViolations = e.getConstraintViolations();
    for (ConstraintViolation<?> constraintViolation : constraintViolations) {
        stringJoiner.add(constraintViolation.getMessage());
    }
    return JsonResult.fail(ServiceCode.ERR_BAD_REQUEST, stringJoiner.toString());
}
```

------

# 相关注解:

> #### 在`javax.validation.constraints`和`org.hibernate.validator.constraints`都有大量的检查注解，常用的检查注解有：

### 1.@Validated注解

- 添加在方法的参数上时,标记此参数需要经过Validation框架的检查

#### 该注解源码如下:

```java
package org.springframework.validation.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Validated {
    Class<?>[] value() default {};
}
```

### 2.@Valid注解

#### 该注解源码如下:

```java
package javax.validation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.METHOD, ElementType.FIELD, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Valid {
}
```

### 3.@NotNull注解

- ##### 该注解通常标注在封装DTO对象的属性上

- ##### 表示客户端传递的信息不允许为Null

  ```java
      @NotNull(message = "必须提交相册名称")
      private String name;
  ```

### 4.@Range注解

- ##### 此注解有`min`和`max`属性，分别通过`@Min`和`@Max`实现，且`min`的默认值为`0`，`max`的默认值为`long`类型的最大值，此注解只能添加在整型的数值类型上，用于设置取值区间

  ```java
      @ApiOperation("根据id删除相册")
      @ApiOperationSupport(order = 200)
      @ApiImplicitParam(name = "id", value = "相册id", required = true, dataType = "long")
      @GetMapping("/{id:[0-9]+}/delete")//在请求路径中先用占位符进行占位,使用正则来限制输入的内容
      public String deleteAlbum2(@Range(min = 1,message = "删除相册失败,尝试删除的相册的ID无效!")
                                     @PathVariable Long id) {//接收路径中通过占位符传入的信息(类型要匹配否则报400)
          String message = "尝试删除id为[" + id + "]的相册";
          log.debug(message);//输出日志
          return message;//向客户端返回结果
      }
  ```

### 5.@NotEmpty注解

- ##### 该注解不允许为空字符串，即长度为0的字符串

- ##### 此注解只能添加在字符串类型的参数上

  ```java
      @NotEmpty//该属性参数不能为空
      private String description;
  ```

### 6.@NotBlank注解

- ##### 不允许为空白，即不允许是仅由空格、TAB制表位、换行符等空白字符组成的

- ##### 此注解只能添加在字符串类型的参数上

  ```java
      @NotBlank//该属性参数不允许是仅由空格、TAB制表位、换行符等空白字符组成
      private String description;
  ```

### 7.@Pattern注解

- ##### 此注解有`regexp`属性，可通过此属性配置正则表达式，在检查时将根据正则表达式所配置的规则进行检查

- ##### 此注解只能添加在字符串类型的参数上

  ```java
  	@Pattern(regexp = "[0-9]+")
      private Integer sort;
  ```

> **注意：除了`@NotNull`注解以外，其它注解均不检查请求参数为`null`的情况，例如在某个请求参数上配置了`@NotEmpty`，当提交的请求参数为`null`时将通过检查（视为正确），所以，当某个请求参数需要配置为不允许为`null`时，必须使用`@NotNull`，且以上不冲突的多个注解可以同时添加在同一个请求参数上！**

------

------

| 注解         | 所属框架          | 作用                                                         |
| ------------ | ----------------- | ------------------------------------------------------------ |
| `@Valid`     | Spring Validation | 添加在方法的参数上，标记此参数需要经过Validation框架的检查   |
| `@Validated` | Spring Validation | 添加在方法的参数上，标记此参数需要经过Validation框架的检查；<br />添加在类上，并结合方法上的检查注解（例如`@NotNull`等）实现对未封装的参数的检查 |
| `@NotNull`   | Spring Validation | 添加在需要被检查的参数上，或添加在需要被检查的封装类型的属性上，用于配置“不允许为`null`”的检查规则 |
| `@NotEmpty`  | Spring Validation | 使用位置同`@NotNull`，用于配置“不允许为空字符串”的检查规则   |
| `@NotBlank`  | Spring Validation | 使用位置同`@NotNull`，用于配置“不允许为空白”的检查规则       |
| `@Pattern`   | Spring Validation | 使用位置同`@NotNull`，用于配置正则表达式的检查规则           |
| `@Range`     | Spring Validation | 使用位置同`@NotNull`，用于配置“数值必须在某个取值区间”的检查规则 |
