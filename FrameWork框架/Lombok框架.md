# Lombok框架

##### Lombok框架的主要作用是通过注解可以**在编译期生成**某些代码，例如Setters & Getters、`hashCode()`与`equals()`、`toString()`方法等，可以简化开发。

#### Lombok的pom依赖:

```xml
<!-- Lombok的依赖项，主要用于简化POJO类的编写 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.20</version>
            <scope>provided</scope>
        </dependency>
```

##### 由于源代码（`.java`）文件中并没有相关代码，所以，默认情况下，开发工具无法智能提示，直接写出相关代码也会提示错误，则需要在开发工具中安装Lombok插件，安装方式：打开IntelliJ IDEA的设置，在**Plugins**一栏，在`Marketplace`中找到`Lombok`并安装即可。

#### Lombok的常用注解有：

- **`@Slf4j`**：添加在类上,便于输出日志,例如:log.debug()
- **`@Data`**：添加在类上，可在编译期生成全部属性对应的Setters & Getters、`hashCode()`与`equals()`、`toString()`和**无参构造**,使用此注解时，**若继承了父类时必须保证当前类的父类存在无参数构造方法**
  - 可在Target中的.class文件中查看

- `@Setter`：可以添加在属性上，将仅作用于当前属性，也可以添加在类上，将作用于类中所有属性，用于生成对应的Setter方法
- `@Getter`：同上，用于生成对应的Getter方法
- `@EqualsAndHashCode`：添加在类上，用于生成规范的`equals()`和`hashCode()`，关于`equals()`方法，如果2个对象的所有属性的值完全相同，则返回`true`，否则返回`false`，关于`hashCode()`也是如此，如果2个对象的所有属性的值完全相同，则生成的HashCode值相同，否则，不应该相同
- `@ToString`：添加在类上，用于生成全属性对应的`toString()`方法
- `@AllArgsConstructor`:全参构造方法
- `@NoArgsConstructor`:无参构造方法

## 关于Slf4j日志

在Spring Boot项目中，基础依赖项（`spring-boot-starter`）中已经包含了日志的相关依赖项。

在添加了`Lombok`依赖项后，可以在类上添加`@Slf4j`注解，则Lombok框架会在编译期生成名为`log`的变量，可调用此变量的方法来输出日志。

日志是有显示级别的，根据日志内容的重要程度，从不重要到重要，依次为：

- `trace`：跟踪信息，可能包含不一定关注，但是包含了程序执行流程的信息
- `debug`：调试信息，可能包含一些敏感内容，比如关键数据的值
- `info`：一般信息
- `warn`：警告信息
- `error`：错误信息

使用Slf4j时，可以使用`log`变量调用以上5个级别对应的方法，来输出不同级别的日志！

在Spring Boot项目中，默认的日志显示级别为【info】，将只会显示此级别及更重要级别的日志！可以在`application.properties`中配置`logging.level.根包=日志显示级别`来设置当前显示级别，例如：

```properties
# 日志的显示级别
logging.level.cn.tedu.csmall=error
```

在开发实践中，应该使用`trace`或`debug`级别的日志来输出与流程相关的、涉及敏感数据的日志，使用`info`输出一般的、被显示在控制台也无所谓的信息，使用`warn`和`error`输出更加重要的、需要关注的日志。

输出日志时，通常建议使用`void trace(String message, Object... args)`方法（也有其它级别日志的同样参数列表的方法），在第1个参数`String message`中，可以使用`{}`作为占位符，表示某变量的值，然后，通过第2个参数`Object... args`来表示各占位符对应的值，例如：

```java
int x = 1;
int y = 2;
log.info("{}+{}={}", x, y, x + y);//写法前后对应
```

使用以上做法，可以避免字符串的拼接，提高了代码的可阅读性，也提高了程序的执行效率！并且，在Slf4j日志中，以上第1个参数由于是字符串常量，将被缓存，如果反复执行此日志输出，执行效率也会更高！

另外，SLF4j是一个日志标准，并不是具体的实现，通常，是通过`logback`或`log4j`等日志框架实现的，当前主流的Spring Boot版本中，都是使用`logback`来实现的。

## 关于@Slf4j日志的Profile配置问题

- ##### 如果application.properties与被激活的Profile配置中存在同名的属性(trace,debug,info,warn,error),配置值却不相同,在执行时,将以Profile配置为准!

- ##### 不同环境下使用不同的日志级别

```yaml
# 激活Profile配置(不同环境下使用不同的日志级别,当前为dev开发环境)
spring:
  profiles:
    active: dev

# Mybatis相关配置
mybatis:
  mapper-locations: classpath:mapper/*.xml

#logging.level.cn.tedu.csmall=info
```

#### 1.开发环境:

```yaml
# ######################### #
# 此配置文件是【开发环境】的配置 #
#  此配置文件需要被激活才会生效 #
# ######################## #

# 连接数据库的配置参数
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mall_pms?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: root
    password: root

# 日志的显示级别默认info(根包尽量这样写,若简写会扩大不必要的范围,多写比较麻烦)
logging:
  level:
    cn.tedu.csmall: trace
```

#### 2.生产环境:

```yaml
# ######################### #
# 此配置文件是【生产环境】的配置 #
#  此配置文件需要被激活才会生效 #
# ######################## #

# 连接数据库的配置参数
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mall_pms?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: root
    password: root

# 日志的显示级别默认info(根包尽量这样写,若简写会扩大不必要的范围,多写比较麻烦)
logging:
  level:
    cn.tedu.csmall: info
```

#### 3.测试环境:

```yaml
# ######################### #
# 此配置文件是【测试环境】的配置 #
#  此配置文件需要被激活才会生效 #
# ######################## #

# 连接数据库的配置参数
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mall_pms?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: root
    password: root

# 日志的显示级别默认info(根包尽量这样写,若简写会扩大不必要的范围,多写比较麻烦)
logging:
  level:
    cn.tedu.csmall: debug
```



------

------

| 注解                  | 所属框架 | 作用                                                         |
| --------------------- | -------- | ------------------------------------------------------------ |
| `@Data`               | Lombok   | 添加在类上，将在编译期生成此类中所有属性的Setter、Getter方法，及`hashCode()`、`equals()`、`toString()`方法 |
| `@Setter`             | Lombok   | 添加在类上，将在编译期生成此类中所有属性的Setter方法，<br/>也可以添加在类的属性上，将在编译期生成此属性的Setter方法 |
| `@Getter`             | Lombok   | 添加在类上，将在编译期生成此类中所有属性的Getter方法，<br/>也可以添加在类的属性上，将在编译期生成此属性的Getter方法 |
| `@EqualsAndHashcode`  | Lombok   | 添加在类上，将在编译期生成基于此类中所有属性的`hashCode()`、`equals()`方法 |
| `@ToString`           | Lombok   | 添加在类上，将在编译期生成基于此类中所有属性的`toString()`方法 |
| `@NoArgConstructor`   | Lombok   | 添加在类上，将在编译期生成此类的无参数构造方法               |
| `@AllArgsConstructor` | Lombok   | 添加在类上，将在编译期生成基于此类中所有属性的全参构造方法   |
