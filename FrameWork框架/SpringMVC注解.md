# Spring框架:

### 1.@ResponseBody注解

- #### 此注解的作用是,用JSON格式将一个方法的返回值加载到response的body区域，向客户端返回数据信息。

- #### 如果设置了返回值但不添加该注解,返回的数据,客户端请求不到.

- #### 也可以不加返回值

```java
@Controller
public class HelloController {
    @RequestMapping("/hello")
    @ResponseBody //此注解的作用是,可以通过返回值的方式给客户端响应数据
    public String hello(){
        return "服务器接收到了响应!";
    }
}
```

#### 注解源码如下:

```java
package org.springframework.web.bind.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ResponseBody {}
```

- #### 处理表单信息的传参方式

```java
@Controller
public class ParamController {
    @RequestMapping("/param1")
    @ResponseBody
    public String param1(HttpServletRequest request){
        //从Request对象中获取参数(该种方法只能传字符串,之后做转换)
        String info = request.getParameter("info");
        return "接收到了:"+info;
        //提交后浏览器路径信息:http://localhost:8080/param1?info=xxx
    }

    @RequestMapping("/param2")
    @ResponseBody
    //SpringMVC框架提供的方式,直接在参数列表中传入表单中的信息,★如果传入参数类型有误,报400错误
    public String param2(String name,int age){//变量名应当与表单上的name值一致
        return name+":"+age;
        //提交后浏览器路径信息:http://localhost:8080/param2?name=xxx&age=xxx
    }

    @RequestMapping("/param3")
    @ResponseBody
    public String param3(Emp emp){//将表单中的信息封装到对象中,在参数列表中传递该对象即可!
        return emp.toString();
        //提交后浏览器路径信息:http://localhost:8080/param3?name=xxx&salary=xxx&job=xxx
    }
}
```

#### 浏览器400错误:传参时不符合对应数据类型导致

```html
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Sat Oct 08 15:57:05 CST 2022
There was an unexpected error (type=Bad Request, status=400).
```

### 2.@RequestMapping注解★

- #### 在Spring MVC框架中，可以在处理请求的方法上添加`@RequestMapping`注解，以配置**请求路径**与**处理请求的方法**的映射关系。

  - ```java
    @RequestMapping("/testRequest")
    @ResponseBody
    public String testRequest(){
        retrun "success";
    }
    ```

- #### 此注解还可以添加在控制器类上，作为当前类中每个请求路径的统一前缀！

  - ```java
    @Controller
    @RequestMapping("/hello")
    public class RequestMappingController{}
    ```

- #### 在开发实践中，强烈建议在类上配置`@RequestMapping`！

  - ##### 当在类上和方法上都使用`@RequestMapping`配置了路径后，实践使用的路径应该是这2个路径值结合起来的路径值，而`@RequestMapping`在处理时，会自动处理两端必要的、多余的`/`符号。

  - #### 例如：

- - | 类上的配置值 | 方法上的配置值 |
    | ------------ | -------------- |
    | /album       | /add-new       |
    | /album       | add-new        |
    | album        | /add-new       |
    | album        | add-new        |
    | album/       | /add-new       |
    | album/       | add-new        |
    | /album/      | /add-new       |
    | /album/      | add-new        |

  - ##### 以上8种配置的组合是等效的！通常，建议在同一个项目中使用统一的风格，例如使用第1种，或使用第4种。

**注意：`@RequestMapping("/")`和`@RequestMapping("")`不是等效的！**

##### 关于`@RequestMapping`注解的源代码，声明部分为：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
    // 暂不关心内部代码
}
```

以上源代码中，`@Target({ElementType.TYPE, ElementType.METHOD})`表示当前注解可以添加在哪些位置，`ElementType.TYPE`表示可以添加在“类型”上，`ElementType.METHOD`表示可以添加在“方法”上。

在`@RequestMapping`注解源代码的内部，还有：

```java
@AliasFor("path")
String[] value() default {};
```

以上代码中，`value()`表示此注解可以配置名为`value`的属性，`String[]`表示此`value`属性的值是`String[]`类型的，`default {}`表示此属性的默认值是`{}`，即空数组，则可以配置为：

```java
@RequestMapping(value = {"a", "b", "c"})
```

而`value`的意义需要通过学习来了解。

另外，`@AliasFor("path")`表示当前`value`属性**等效于**`path`属性。

在Java语言中，如果某个注解属性的值是某种数组类型，但是，需要配置的值只有1个（数组中只有1个元素），可以不必使用一对大括号将其框住！

例如，以下2种配置是完全等效的：

```java
@RequestMapping(value = {"a"})
@RequestMapping(value = "a")
```

在Java语言中，`value`是各注解默认的属性名，如果注解只需要配置这1个属性，可以不必显式指定属性名！

例如，以下2种配置是完全等效的：

```java
@RequestMapping(value = "a")
@RequestMapping("a")
```

另外，在`@RequestMapping`的源代码中，还包含：

```java
RequestMethod[] method() default {};
```

此属性的作用的是限制请求方式，在默认情况下，客户端提交请求时，所有请求方式都是允许的！如果配置此属性，则只有与配置值匹配的请求方式才是允许的，如果客户端提交请求的方式有误，则会导致`405`错误！

> 提示：目前，提交POST请求的方式有：使用HTML的`<form>`标签的`method="post"`、使用`axios`的`post()`函数，或其它通过JavaScript程序发出POST请求，除此以外，都是GET请求，例如：直接在浏览器的地址栏中输入URL并访问、点击页面上的超链接、打开浏览器收藏夹中的收藏地址等。

通常，建议：**以获取数据为主要目的的请求，应该限制为GET方式，除此以外，都应该限制为POST方式。**

Spring MVC框架还定义了已经限制了请求方式的、与`@RequestMapping`类似的注解，包括：

- ##### `@GetMapping`

- ##### `@PostMapping`

- ##### `@PutMapping`

- ##### `@DeleteMapping`

- ##### `@PatchMapping`

> 通过地址栏访问的都是get请求,为便于开发人员测试,可使用Knife4j文档框架进行测试

以`@GetMapping`为例，其源代码片段：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RequestMapping(method = RequestMethod.GET)
public @interface GetMapping {

	/**
	 * Alias for {@link RequestMapping#name}.
	 */
	@AliasFor(annotation = RequestMapping.class)
	String name() default "";

	/**
	 * Alias for {@link RequestMapping#value}.
	 */
	@AliasFor(annotation = RequestMapping.class)
	String[] value() default {};

	/**
	 * Alias for {@link RequestMapping#path}.
	 */
	@AliasFor(annotation = RequestMapping.class)
	String[] path() default {};

	/**
	 * Alias for {@link RequestMapping#params}.
	 */
	@AliasFor(annotation = RequestMapping.class)
	String[] params() default {};

	/**
	 * Alias for {@link RequestMapping#headers}.
	 */
	@AliasFor(annotation = RequestMapping.class)
	String[] headers() default {};

	/**
	 * Alias for {@link RequestMapping#consumes}.
	 * @since 4.3.5
	 */
	@AliasFor(annotation = RequestMapping.class)
	String[] consumes() default {};

	/**
	 * Alias for {@link RequestMapping#produces}.
	 */
	@AliasFor(annotation = RequestMapping.class)
	String[] produces() default {};

}
```

### 3.封装对象时要注意的细节

- #### getter,setter方法,重写toString,有参和全参构造器(行业标准)

```java
//无参构造器默认存在,但是当有了有参构造器,无参构造器就不存在了,由于SpringMVC内部会用到无参构造器,所以需再加入无参构造器
    public Product(){}

    public Product(Integer id, String title, Double price, Integer num) {
        this.id = id;
        this.title = title;
        this.price = price;
        this.num = num;
    }
```

### 4.@RestController注解

- #### 与@Controller注解相似,不同的是使用@RestController相当于在每一个方法上都添加了@ResponseBody注解(该注解通过返回值方式向浏览器响应数据)

#### 该注解源码如下:

```java
package org.springframework.web.bind.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.stereotype.Controller;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {}
```

### 5.@Mapper注解

- #### 该注解是一个接口类型,在接口中书写实体类和数据库中表之间的对应关系,Mybatis框架会自动通过此关系生成JDBC代码

- #### @Insert/Select/Delete/Update()该注解方法用来写SQL语句,配合抽象方法传入的参数完成与数据库的相关操作(增删改查...)

#### 该注解源码如下:

```java
package org.apache.ibatis.annotations;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Documented
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER})
public @interface Mapper {}
```

### 6.@Autowired注解

- ##### 自动装配 此框架添加之后,Mybatis和Spring框架会生成ProductMapper的实现类,并且实例化该实现类(实现类里面会实现ProductMapper中的抽象方法,实现的方法里面写的就是JDBC代码),并且把实例化好的对象赋值给了mapper变量

- ##### Spring会自动帮你把bean里面引用的对象的setter/getter方法省略,它会自动帮你set/get

#### 该注解源码如下:

```java
package org.springframework.beans.factory.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {}
```

### 7.@RequestBody(请求体)注解

- ##### 在Spring MVC项目中（包括添加了`spring-boot-starter-web`依赖项的Spring Boot项目），可以在处理请求的方法的参数列表中，在某参数上添加`@RequestBody`注解。

  当请求参数添加了`@RequestBody`注解时，则客户端提供的请求参数必须是**JSON对象格式**的，例如：

  ```json
  {
      'name': '测试相册名称',
      'description': '测试相册简介',
      'sort': 188
  }
  ```

  如果客户端提交的请求参数不是对象格式时，将出现以下错误：

  ```
  Resolved [org.springframework.web.HttpMediaTypeNotSupportedException: Content type 'application/x-www-form-urlencoded;charset=UTF-8' not supported]
  ```

  ##### 当请求参数没有添加`@RequestBody`注解时，则客户端提供的请求参数必须是**FormData格式**的，例如：

  ```
  name=测试相册名称&description=测试相册简介&sort=188
  ```

  如果请求不是FormData格式的，则服务器端将接收不到请求参数，各参数值将为`null`。

#### 该注解源码如下:

```java
package org.springframework.web.bind.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestBody {}
```

### 8.@JsonFormat注解

- ##### 该注解处理事件属性的呈现格式和时区

  ```java
  private Integer id;
      private String content;
      private String url; //微博图片路径
      //通过JsonFormat设置显示的时间格式
      // 2022年10月12号 15时23分22秒   2022-10-12 15:23:22
      // yyyy年MM月dd号 HH时mm分ss秒   yyyy-MM-dd HH:mm:ss (h为1-12时,H为1-24时)
      @JsonFormat(pattern = "yyyy年MM月dd号 HH时mm分ss秒",timezone = "GMT+8")//指定时间呈现格式和时区GMT+8东八区
      private Date created; //发布微博时间
  ```

### 9.@Configuration注解

- ##### @Configuration注解可告诉编译器该工程所有Mapper接口都在这个包路径path里面.

- ##### 该注解用于config配置类,可在项目启动时对该类进行加载

#### 该注解源码如下:

```java
package org.springframework.context.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {}
```

### 10.@Value注解

- ##### 该注解会将XML配置文件中的变量赋给注解下的变量

- ##### 例如配置文件中:

  - ```properties
    spring.web.resources.static-locations=file:${dirPath},classpath:static
    #设置上传文件大小  默认1MB
    spring.servlet.multipart.max-file-size=10MB
    #配置Mybatis书写SQL语句的xml文件的位置
    mybatis.mapper-locations=classpath:mappers/*xml
    #配置静态资源文件夹路径,利用${}调用
    dirPath=G:/files
    ```

- ##### 在java类中使用时:

  - ```java
    @RestController
    public class UploadController {
        @Value("${dirPath}")//该注解会将配置文件中的变量传递到当下的全局变量中
        private String dirPath;
        //准备保存图片的文件夹
        File dirFile = new File(dirPath);
        if (!dirFile.exists()){
            dirFile.mkdirs();
        }
    ```

#### 该注解源码如下:

```java
package org.springframework.beans.factory.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Value {
    String value();
}
```

### 11.@ServletComponentScan注解

- ##### 该注解作用在类上,通常在启动类中

- ##### 作用是扫描当前包以及子包中的过滤器

  ![image-20221020102649124](images/image-20221020102649124.png)

#### 该注解源码如下:

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({ServletComponentScanRegistrar.class})
public @interface ServletComponentScan {}
```

### 12.@SpringBootTest注解

- ##### 该注解用于测试类上

### 13.@Test注解

- ##### 该注解用于测试类中的方法上

### 14.@Data注解

- #### 该注解是Lombok的依赖项产生的注解(Lombok框架),可自动提供getter和setter方法,重写toString()和hashCode()方法

- ```xml
   <!-- Lombok的依赖项，主要用于简化POJO类的编写 -->
          <dependency>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
              <version>1.18.20</version>
              <scope>provided</scope>
          </dependency>
  ```

### 15.@Repository注解

- #### 当自动装配Mapper接口的对象时，IntelliJ IDEA可能会报错，提示无法装配此对象，但是，并不影响运行!

- #### 添加该注解是为了引导IntelliJ IDEA作出正确的判断

- ```java
  @Repository//该注解用于引导IDEA作出正确的判断
  public interface CategoryMapper {}
  ```

#### 该注解源码如下:

```java
package org.springframework.stereotype;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {}
```

### 16.@Configuration和@MapperScan()注解的结合使用

- ##### @Configuration注解可告诉编译器该工程所有Mapper接口都在这个包路径path里面.

- ##### @MapperScan()注解用于传入要告知服务器mapper接口的包路径.

- ##### 这两种注解Config配置完成后,在接口中就无序添加@Mapper注解和增删改查的@Insert()/@Delete()/@Update()/@Select()注解方法了

  ```java
  @Configuration
  @MapperScan("cn.tedu.boot08.mapper")
  public class MyBatisConfig {
  }
  ```

#### 注解源码如下:

```java
package org.mybatis.spring.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.context.annotation.Import;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({MapperScannerRegistrar.class})
@Repeatable(MapperScans.class)
public @interface MapperScan {}
```

### 17.@Resource注解

- #### 自动装配/注入,与@Autowired注解类似

### 18.@Service注解

- ##### `@Service`注解用于类上，标记当前类是一个service类，位于Service层

- ##### 加上该注解会将当前类自动注入到spring容器中，不需要再在applicationContext.xml文件定义bean了。

#### 该注解源码如下:

```java
package org.springframework.stereotype;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {}
```

### 19.@ExceptionHandler注解

- ##### 首先处理请求方法抛出的异常由SpringMVC框架进行处理

- ##### 该注解会让SpringMVC自行捕获当前Controller类中处理请求的所有方法抛出的异常

- ##### 在该注解下可捕获可能产生的不同种类的异常

  - **全局异常处理**(只要出现异常,执行这个处理)
  - **特定异常处理**(针对特定异常处理)
  - **自定义异常处理**(自己编写异常类,手动抛出异常)
    - 自定义类编写完成并添加注解后,它不会自动抛出,因此需要在对应的异常代码中使用try-catch来将该自定义异常进行抛出

- ##### 防止频繁的进行try-catch

#### 该注解源码如下:

```java
package org.springframework.web.bind.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ExceptionHandler {
    Class<? extends Throwable>[] value() default {};
}
```

### 20.ControllerAdvice注解

- ##### 该注解用来标注类,用于处理当前项目的全局异常

- ##### 该类应当定义处理各种`@RequestMapping`标注的处理业务方法可能抛出的所有异常,达到全局的效果

- ##### 每种经`@RequestMapping`标注的处理业务的方法都还需增加`@ResponseBody`注解,用于将返回值以JSON的格式加载到response的Body区域,给用户返回

#### 该注解源码如下:

```java
package org.springframework.web.bind.annotation;

import java.lang.annotation.Annotation;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.stereotype.Component;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface ControllerAdvice {}
```

### 21.@RestControllerAdvice注解

- ##### 该注解用于处理全局异常(对于整个项目而言),作用在类上

- ##### 该类应当定义处理各种`@RequestMapping`标注的处理业务方法可能抛出的所有异常,达到全局的效果

- ##### 使得任何标注`@RequestMapping`处理请求的方法对于XXXException都应该是抛出的且各`Controller`控制器类中都不必关心如何处理XXXException

- ##### 添加该注解后就不用在每一个处理异常的方法上添加`@ResponseBody`注解来以特定的JOSN格式返回数据

#### 该注解源码如下:

```java
package org.springframework.web.bind.annotation;

import java.lang.annotation.Annotation;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ControllerAdvice
@ResponseBody
public @interface RestControllerAdvice {}
```

### 22.@PathVariable注解

- ##### 该注解作用在参数列表中,作用是获取`@RequestMapping`的请求路径中用`{}`占位符标注的部分

- ##### 每添加一个占位符(添加一个参数),参数列表中的参数声明前都要添加一次该注解

- #### 例:

  ```java
      @PostMapping("/{name:[a-z]+}/{sort:[a-z]+}/delete")//在请求路径中先用占位符进行占位
      //接收路径中通过占位符传入的信息(类型要匹配否则报400)
      public String deleteAlbum1(@PathVariable String name,@PathVariable Integer sort{
          String message = "尝试删除名称为[" + name + "],排序为[" + sort + "]的相册";
          log.debug(message);//输出日志
          return message;//向客户端返回结果
      }
  ```

#### 该注解源码如下:

> @AliasFor该注解有相较于的意思,表示这里的"value"相当于"name"

```java
package org.springframework.web.bind.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface PathVariable {
    //@AliasFor该注解有相较于的意思,这里的"value"相当于"name"
    @AliasFor("name")
    String value() default "";

    @AliasFor("value")
    String name() default "";

    boolean required() default true;
}
```

