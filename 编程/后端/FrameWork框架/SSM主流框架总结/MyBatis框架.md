# MyBatis框架:

![image-20221026105111962](images/image-20221026105111962.png)

## 简介:

### 什么是 MyBatis？

- ##### MyBatis 是一款优秀的持久层框架

- ##### 它支持自定义 SQL、存储过程以及高级映射。

- ##### MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。

- ##### MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

## 文档地址:

- #### https://mybatis.net.cn/getting-started.html

## XML 映射器:

**MyBatis 的真正强大在于它的语句映射，这是它的魔力所在**。由于它的异常强大，映射器的 XML 文件就显得相对简单。如果拿它跟具有相同功能的 JDBC 代码进行对比，你会立即发现省掉了将近 95% 的代码。MyBatis 致力于减少使用成本，让用户能更专注于 SQL 代码。

SQL 映射文件只有很少的几个顶级元素（按照应被定义的顺序列出）：

- `cache` – 该命名空间的缓存配置。
- `cache-ref` – 引用其它命名空间的缓存配置。
- `resultMap` – 描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
- `parameterMap` – 老式风格的参数映射。此元素已被废弃，并可能在将来被移除！请使用行内参数映射。文档中不会介绍此元素。
- `sql` – 可被其它语句引用的可重用语句块。
- `@insert` – 映射插入语句。
- `@update` – 映射更新语句。
- `@delete` – 映射删除语句。
- `@select` – 映射查询语句。

## 动态 SQL:

**动态 SQL 是 MyBatis 的强大特性之一**。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。

使用动态 SQL 并非一件易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语言，MyBatis 显著地提升了这一特性的易用性。

如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

- if
- choose (when, otherwise)
- trim (where, set)
- foreach

## 日志:

Mybatis 通过使用内置的日志工厂提供日志功能。内置日志工厂将会把日志工作委托给下面的实现之一：

- SLF4J
- Apache Commons Logging
- Log4j 2
- Log4j
- JDK logging

##### 在添加了`Lombok`依赖项后，可以在类上添加`@Slf4j`注解，则Lombok框架会在编译期生成名为`log`的变量，可调用此变量的方法来输出日志。

日志是有显示级别的，根据日志内容的重要程度，从不重要到重要，依次为：

- ##### `trace`：跟踪信息，可能包含不一定关注，但是包含了程序执行流程的信息
- ##### `debug`：调试信息，可能包含一些敏感内容，比如关键数据的值
- ##### `info`：一般信息
- ##### `warn`：警告信息
- ##### `error`：错误信息

使用Slf4j时，可以使用`log`变量调用以上5个级别对应的方法，来输出不同级别的日志！

------

# 相关注解:

### 1.@Param注解

- ##### 当Mapper接口中的抽象方法有多个参数时,使用该注解来配置参数名称,与Xml文件中的SQL语句进行对应

  - ##### Mapper:

  ```java
  int updatePasswordById(@Param("i") Long id,@Param("pass") String password);
  ```

  - ##### Xml:

  ```xml
  <!--int updatePasswordById(@Param("i") Long id,@Param("pass") String 	password);-->
  <update id="updatePasswordById">
      UPDATE ams_admin SET password=#{pass} WHERE id = #{i}
  </update>
  ```

  - ##### MapperTests:

  ```java
  @Test
  void updatePasswordById(){
      Long i = 21L;
      String pass = "1234";
      adminMapper.updatePasswordById(i,pass);
  }
  ```

#### 该注解源码如下:

```java
package org.apache.ibatis.annotations;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.PARAMETER})
public @interface Param {
    String value();
}
```

------

------

| 注解          | 所属框架 | 作用                                                         |
| ------------- | -------- | ------------------------------------------------------------ |
| `@MapperScan` | Mybatis  | 添加在配置类上，用于指定Mapper接口的根包，<br />Mybatis将根据此根包执行扫描，以找到各Mapper接口 |
| `@Mapper`     | Mybatis  | 添加在Mapper接口上，用于标记此接口是Mybatis的Mapper接口，如果已经通过`@MapperScan`配置能够找到此接口，则不需要使用此注解 |
| `@Param`      | Mybatis  | 添加在Mapper接口中的抽象方法的参数上，用于指定参数名称，当使用此注解指定参数名称后，SQL中的`#{}` / `${}`占位符中的名称必须是此注解指定的名称，通常，当抽象方法的参数超过1个时，强烈建议在每个参数上使用此注解配置名称 |
| `@Select`     | Mybatis  | 添加在Mapper接口的抽象方法上，可以通过此注解直接配置此抽象方法对应的SQL语句（不必将SQL语句配置在XML文件中），用于配置`SELECT`类的SQL语句，但是，非常不推荐这种做法 |
| `@Insert`     | Mybatis  | 同上，用于配置`INSERT`类的SQL语句                            |
| `@Update`     | Mybatis  | 同上，用于配置`UPDATE`类的SQL语句                            |
| `@Delete`     | Mybatis  | 同上，用于配置`DELETE`类的SQL语句                            |
