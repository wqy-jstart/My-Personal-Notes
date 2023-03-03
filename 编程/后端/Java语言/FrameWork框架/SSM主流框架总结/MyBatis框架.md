# MyBatis框架

![image-20221026105111962](images/image-20221026105111962.png)

## <u>简介</u>

### 什么是 MyBatis？

- ##### MyBatis 是一款优秀的持久层框架

- ##### 它支持自定义 SQL、存储过程以及高级映射。

- ##### MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。

- ##### MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

文档地址:https://mybatis.net.cn/getting-started.html

## <u>核心价值</u>

**简化了关系型数据库编程**

## <u>突出功能</u>

### XML 映射器:

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

### 动态 SQL:

**动态 SQL 是 MyBatis 的强大特性之一**。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。

使用动态 SQL 并非一件易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语言，MyBatis 显著地提升了这一特性的易用性。

如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

- if
- choose (when, otherwise)
- trim (where, set)
- foreach

### 日志:

Mybatis 通过使用内置的日志工厂提供日志功能。内置日志工厂将会把日志工作委托给下面的实现之一：

- SLF4J
- Apache Commons Logging
- Log4j 2
- Log4j
- JDK logging

在添加了`Lombok`依赖项后，可以在类上添加`@Slf4j`注解，则Lombok框架会在编译期生成名为`log`的变量，可调用此变量的方法来输出日志。

日志是有显示级别的，根据日志内容的重要程度，从不重要到重要，依次为：

- ##### `trace`：跟踪信息，可能包含不一定关注，但是包含了程序执行流程的信息
- ##### `debug`：调试信息，可能包含一些敏感内容，比如关键数据的值
- ##### `info`：一般信息
- ##### `warn`：警告信息
- ##### `error`：错误信息

使用Slf4j时，可以使用`log`变量调用以上5个级别对应的方法，来输出不同级别的日志！

------

## <u>基础依赖项</u>

1. **mybatis-spring**：整合Spring与Mybatis，使得编程更加简单

   ```xml
   <dependency>
       <groupId>org.mybatis.spring.boot</groupId>
       <artifactId>mybatis-spring-boot-starter</artifactId>
       <version>2.1.4</version>
   </dependency>
   ```

2. **spring-jdbc**：整合Spring JDBC（依赖被mybatis-spring整合）

3. **mysql-connector-java**：Mysql和MariaDB的依赖项

   ```xml
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <scope>runtime</scope>
   </dependency>
   ```

4. **commons-dbcp2**：数据库连接池的依赖项，是经典的commons-dbcp的升级款，可替换Hikari、Druid等其他数据库连接池

5. **spring-test**：Spring测试的依赖项，用于及时监测环境配置与实现的数据访问功能

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-test</artifactId>
       <scope>test</scope>
   </dependency>
   ```

## <u>基本配置</u>

1.在`.properties`配置文件中配置连接数据库的参数

```properties
spring.datasource.username=root # 用户名
spring.datasource.password=root # 密码
spring.datasource.url=jdbc:mysql://localhost:3306/cs?characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai&rewriteBatchedStatements=true # url
```

2.读取连接数据库的配置，在配置类中使用@Bean方法创建DataSource对象

>在SpringBoot项目中不需要，已经被mybatis-spring-boot-starter自动处理

3.在`.properties`配置文件中配置使用的MyBatis时的XML文件位置

```properties
mybatis.mapper-locations=classpath:mappers/*xml
```

4.读取配置中的XML文件位置，在配置类中使用`@Bean`方法创建`SqlSessionFactoryBean`对象

> 在SpringBoot项目中不需要，已经被mybatis-spring-boot-starter自动处理

5.在配置类上使用`@MapperScan`指定Mapper接口所在的包

- 要保证此包下不会有其他接口，MyBatis会自动生成此包下所有接口的代理对象
- 可以不配置此注解，但需要在所有的Mapper接口上添加`@Mapper`注解（不推荐）

## <u>编码逻辑</u>

基本配置完成之后即可进行基于MyBatis框架的数据库编程！

**编程分两步**：

一、Mapper接口和抽象方法

二、XML文件的动态SQL和关系映射

### 1.声明Mapper接口与抽象方法

> 建议每张数据表都应当有对应的Mapper接口

#### 声明接口：

> 位置要在配置类MapperScan扫描的包下

```java
public interface UserMapper {

    UserVo selectByUsername(String username);

}
```

#### 关于抽象方法：

1. **返回值类型**：
   - 插入数据：int 表示受影响的行数
   - 删除数据：int 表示受影响的行数
   - 修改数据：int 表示受影响的行数
   - 查询数据：足以封装查询结果的类型对象（如上面的UserVO）
     - 统计查询：int/long 
     - 查询一条结果：自定义类型
     - 查询多条结果：使用List集合
2. **方法名称**：
   - 自定义
   - 阿里巴巴规范
     - 获取单个对象：get做前缀
     - 获取多个对象：用list做前缀
     - 获取统计值：count做前缀
     - 插入的方法：save/insert做前缀
     - 删除的方法：remove/delete做前缀
     - 修改的方法：update做前缀
3. **参数列表**：
   - 根据需要执行的SQL语句where条件来设计
   - 如果多个参数可以使用对象封装
   - 如果参数超过1个，可添加@Param配置参数名与XML文件的标签id属性对应

### 2.配置抽象方法映射的SQL

#### 注解配置：

> 篇幅较长的SQL不易与阅读、不易于实现复杂的配置、不易于与DBA合作

```java
@Insert()  // 配置插入
@Delete()  // 配置删除
@Update()  // 配置修改
@Select()  // 配置查询
```

#### XML配置：

> 首先XML文件必须有特定的文档声明（固定代码）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="对应的Mapper接口完全限定名">
</mapper>
```

1. 使用`<insert>`子节点配置插入语句，建议配置`useGeneratedKeys`和`keyProperty`获取自动编号的主键值，如果主键不是自动编号，可以不配置。

2. 使用`<delete>`子节点配置删除语句

3. 使用`<update>`子节点配置修改语句

4. 使用`<select>`子节点配置查询语句，必须配置`resultType`或`resultMap`

5. 使用`<sql>`封装SQL语句片段，使用`<include>`引入，通常封装查询字段

6. 使用`<resultMap>`配置结果集到抽象方法的返回值对象中

   - 使用`<id>`配置主键，一般结果使用`<result>`配置，搭配`column`和`property`属性

   - List类型的属性，使用`<collection>`

     - 如果List的元素是基本值，使用`<constructor>`配置

     - 如果List的元素是其他引用类型，使用id和result配置

       ```xml
       <resultMap id="deptRM" type="cn.tedu.boot10.pojo.vo.TeamVO">
           <id column="tid" property="id"></id>
           <result column="tname" property="name"></result>
           <result column="loc" property="loc"></result>
           <!--关联集合集合用collection标签-->
           <collection property="playerList" ofType="cn.tedu.boot10.pojo.entity.Player">
               <id column="pid" property="id"></id>
               <result column="pname" property="name"></result>
           </collection>
       </resultMap>
       ```

   - 对象类型的属性，使用`<association>`

     ```xml
     <resultMap id="playerRM" type="cn.tedu.boot10.pojo.vo.PlayerVO">
         <id column="pid" property="id"></id>
         <result column="pname" property="name"></result>
         <association property="team" javaType="cn.tedu.boot10.pojo.entity.Team">
             <id column="tid" property="id"></id>
             <result column="tname" property="name"></result>
             <result column="loc" property="loc"></result>
         </association>
     </resultMap>
     ```

7. 动态SQL

   - 使用`<foreach>`对参数遍历
   - 使用`<set>`结合`<if>`实现动态修改数据

#### 占位符：

1. `#{}`格式的占位符：

   <u>推荐</u>

   预编译，表示某一个值，不存在SQL注入，不需考虑数据类型（例如：字符串不需要引号括住）

2. `${}`格式的占位符：

   <u>不推荐</u>

   非预编译，可以表示SQL语句的片段，存在SQL注入，需要考虑数据类型（例如：字符串需要引号括住）

## <u>相关注解</u>

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
