# Redis

Redis是一款基于内存的、使用K-V结构来实现读写数据的NoSQL非关系型数据库

- 基于内存的：Redis访问的数据都在内存中
  - Redis的读写效率非常高
  - 其实Redis也会自动的处理持久化，但是正常读写都是在内存中执行的

- NoSQL：不涉及SQL语句，`No`可理解为日常英语中的`no`（没有），也可理解为`No Operation`（不操作）
- ##### 非关系型数据库：不关心数据库中存储的是什么数据，几乎没有数据种类的概念，更不存在数据与数据之间的关联

通常，在项目中，Redis用于实现缓存！

![image-20221111143809078](images/image-20221111143809078.png)

当使用Redis后：

- 【优点】读取数据的效率会高很多
- 【优点】能够一定程度上保障缓解数据库的查询压力，提高数据库的安全性
- 【缺点】需要关注数据一致性问题，即Redis中的数据与MySQL中的数据是否一致，如果不一致，是否需要处理

# 1. Redis的基本操作

在Windows操作系统中，通过`.msi`安装包来安装的Redis，会自动注册Redis服务，开机会自动启动Redis，所以，Redis处于随时可用的状态。

可以在命令提示符窗口或终端窗口通过`redis-cli`命令，登录Redis控制台：

```
D:\IdeaProjects\jsd2207-csmall-product-teacher>redis-cli
127.0.0.1:6379>
```

当操作提示符变成 `127.0.0.1:6379>` 后，表示已经登录到Redis客户端的控制台。

在Redis客户端控制台中，可以通过`ping`命令实时检测Redis是否仍处理可用状态，如果Redis服务正常可用，将反馈`PONG`：

```
127.0.0.1:6379> ping
PONG
```

在Redis客户端控制台中，可以通过`exit`命令退出，以回到操作系统的终端：

```
127.0.0.1:6379> exit

D:\IdeaProjects\jsd2207-csmall-product-teacher>
```

在Redis客户端控制台中，可以通过`set`命令向Redis中存入值数据，例如：

```
127.0.0.1:6379> set name wangkejing
OK
```

在Redis客户端控制台中，可以通过`get`命令向Redis中存入值数据，例如：

```
127.0.0.1:6379> get name
"wangkejing"
```

```
127.0.0.1:6379> get email
(nil)
```

提示：以上使用的`set`命令，既是新增数据的命令（当Key尚且不存在时），也是修改数据的命令（当Key已经存在时），例如：

```
127.0.0.1:6379> set name fanchuanqi
OK
127.0.0.1:6379> get name
"fanchuanqi"
```

在Redis客户端控制台中，可以通过`keys`命令向Redis中存入值数据，此命令必须有参数，在参数中可以使用通配符，例如：

```
127.0.0.1:6379> keys username1
1) "username1"

127.0.0.1:6379> keys username0
(empty list or set)

127.0.0.1:6379> keys user*
1) "username1"
2) "username3"
3) "username2"
4) "username4"

127.0.0.1:6379> keys *
1) "username1"
2) "email1"
3) "email2"
4) "username3"
5) "name"
6) "username2"
7) "username4"
```

**注意：`keys`命令会根据模式查找当前Redis中的Key，可能耗时较长，会导致“阻塞”，所以，在生产环境中，一般不允许使用此命令！**

在Redis客户端控制台中，可以通过`dbsize`命令查看Redis中的数据的数量，例如：

```
127.0.0.1:6379> dbsize
(integer) 7
```

在Redis客户端控制台中，可以通过`del`命令删除Redis中指定Key的数据，例如：

```
127.0.0.1:6379> del username1
(integer) 1
```

在Redis客户端控制台中，可以通过`flushall`命令删除Redis中的数据，例如：

```
127.0.0.1:6379> flushall
OK
127.0.0.1:6379> keys *
(empty list or set)
```

更多命令，可查阅资料，例如：https://blog.csdn.net/weixin_46742102/article/details/109483603

Redis中的传统数据类型有：string、list、hash、set、zset。

# 2. Redis编程

需要添加依赖项：

```xml
<!-- Spring Boot Data Redis的依赖项，用于实现Redis编程 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

在Spring系列框架中，Redis编程需要使用`RedisTemplate`工具类，此工具类应该事先创建、配置，并保存到Spring容器中，当需要使用时，自动装配此工具类的对象。

在根包下创建`RedisConfiguration`配置类，在此类中使用`@Bean`方法来配置`RedisTemplate`：

```java
package cn.tedu.csmall.product.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;

import java.io.Serializable;

/**
 * Redis配置类
 *
 * @author java@tedu.cn
 * @version 0.0.1
 */
@Slf4j
@Configuration
public class RedisConfiguration {

    public RedisConfiguration() {
        log.debug("创建配置类对象：RedisConfiguration");
    }

    // ★添加@Bean注解,向Spring容器中创建该对象,交由Spring管理,使用时可以进行自动装配
    @Bean
    public RedisTemplate<String, Serializable> redisTemplate(
            RedisConnectionFactory redisConnectionFactory) {// 配置Redis连接工厂
        RedisTemplate<String, Serializable> redisTemplate = new RedisTemplate<>();// 1.创建一个Redis模板
        redisTemplate.setConnectionFactory(redisConnectionFactory);// 2.设置Redis连接工厂
        
        redisTemplate.setKeySerializer(RedisSerializer.string());// 3.创建String序列化器,序列化key
        redisTemplate.setValueSerializer(RedisSerializer.json());// 4.创建JSON序列化器,序列化value值部分
        return redisTemplate;// 5.返回配置后的Redis模板
    }
}
```

> #### Redis中的Value比较特殊,可以是数据,可以是对象,可以是集合,不同状态不同处理方式

# 3.Redis编程测试

前提,先在`RedisConfiguration`组件配置类中进行配置Redis相关的设置,并添加`@Bean`注解,配置完成后,使用前,需对配置的Redis方法对象进行装配,即可使用!!!

```java
@Autowired
RedisTemplate<String , Serializable> redisTemplate;
```

> ##### ValueOperations opsForValue() >>> 获取ValueOperations对象，操作Redis中的string类型时需要此对象
>
> ##### ListOperations opsForList() >>> 获取ListOperations对象，操作Redis中的list类型时需要此对象

- #### 1.存入数据

  - ##### 利用装配进来的方法`redisTemplate()`来调用`opsForValue()`返回`ValueOperations<K, V>`接口

  - ```java
    void set(String key, Serializable value) >>> 向Redis中写入数据
    ```

```java
// 向Redis中存入数据
@Test
void valueSet(){
    String key = "username";
    String value = "fanchaunqi";

    ValueOperations<String ,Serializable> ops = redisTemplate.opsForValue();
    ops.set(key,value);
}
```

- #### 2.取出数据

  - 利用返回的`ValueOperations<K, V>`接口类,调用该接口类中的方法

  - ```java
    Serializable get(String key) >>> 读取Redis中的数据
    ```

```java
// 向Redis中取出数据
@Test
void valueGet(){
    String key = "username";

    ValueOperations<String ,Serializable> ops = redisTemplate.opsForValue();
    Serializable value = ops.get(key);
    log.debug("从Redis中取出Key值为[{}]的数据,结果:{}",key,value);// wqy
}
```

- #### 3.存入对象

  - 和存入数据相同,但`set`方法中的`value`应放入准备的对象

```java
// 向Redis中存入对象
@Test
void valueSetObject(){
    String key = "album2022";
    AlbumStandardVO album = new AlbumStandardVO();
    album.setId(2022L);
    album.setName("测试相册00001");
    album.setDescription("测试简介00001");
    album.setSort(998);

    ValueOperations<String,Serializable> ops = redisTemplate.opsForValue();
    ops.set(key,album);
}
```

- #### 4.取出对象

  - 和取出数据相同,传入对象的key返回一个Serializable可序列化接口,最后直接输出

```java
// 向Redis中取出对象
@Test
void valueGetObject(){
    String key = "album2022";

    ValueOperations<String,Serializable> ops = redisTemplate.opsForValue();
    Serializable value = ops.get(key);
    log.debug("从Redis中取出Key值为[{}]的数据",key);
    log.debug("结果:{}",value);// AlbumStandardVO(id=2022, name=测试相册00001, description=测试简介00001, sort=998)
    log.debug("结果的数据类型:{}",value.getClass().getName());
}
```

> ##### 在终端中get一个对象,输出的内容会包含该对象反射后的完全限定名
>
> ```java
> {"@class":"cn.tedu.csmall.product.pojo.vo.AlbumStandardVO","id":2022,"name":"测试相册00001","description":"测试简介00001","sort":998}
> ```

- #### 5.根据正则查询数据

  - ```java
    Set<String> keys(String pattern) >>> 根据模式pattern搜索Key
    ```

```java
// 根据正则查询数据
@Test
void keys(){
    String pattern = "*";// 查所有(正常在终端不推荐)

    Set<String> keys = redisTemplate.keys(pattern);
    log.debug("根据模式[{}]搜索Key,结果:{}",pattern,keys);// [username, album2022]
}
```

- #### 6.删除一条Redis数据

  - ```java
    Boolean delete(String key) >>> 根据Key删除数据，返回成功与否
    ```

```java
// 删除一条Redis数据
@Test
void delete(){
    String key = "username1";

    Boolean result = redisTemplate.delete(key);
    log.debug("根据key[{}]删除数据完成,结果为:{}",key,result);
}
```

- #### 7.删除多条Redis数据

  - ```java
    Long delete(Collection<String> keys) >>> 根据Key的集合批量删除数据，返回成功删除的数据的数量
    ```

```java
// 删除多条Redis数据
@Test
void deleteX(){
    Set<String> keys = new HashSet<>();
    keys.add("username2");
    keys.add("username3");
    keys.add("username4");

    Long count = redisTemplate.delete(keys);
    log.debug("根据Key集合[{}]删除数据完成,成功删除的数据的数量:{}",keys,count);
}
```

- #### 8.存入List集合

  - ```java
    ListOperations opsForList() >>> 获取ListOperations对象，操作Redis中的list类型时需要此对象
    ```

```java
// 存入List集合,通常从右侧压栈
@Test
public void rightPush(){
    String key = "stringList";
    List<String> stringList = new ArrayList<>();
    for (int i = 1; i <= 10; i++) {
        stringList.add("string-"+i);
    }

    ListOperations<String,Serializable> ops = redisTemplate.opsForList();
    for (String s : stringList) {
        ops.rightPush(key,s);
    }
}
```

- #### 9.取出List集合

##### 想要取出Redis中的List集合,需要借用工具,例如Another Redis Desktop Manager可视化软件

在Another Redis Desktop Manager中可以对Redis进行各种操作,并可以查询不同类型的value值

##### 下载地址:http://doc.canglaoshi.org/

![image-20221112214402729](images/image-20221112214402729.png)

------

关于Redis中的`list`类型的数据，它是一种先进后出、后进先出的栈结构：

![image-20221111174239963](images/image-20221111174239963.png)

在Redis中操作`list`时，允许从左侧或右侧进行操作（请将此栈结构想像为横着的）：

![image-20221111174343596](images/image-20221111174343596.png)

![image-20221111174416595](images/image-20221111174416595.png)