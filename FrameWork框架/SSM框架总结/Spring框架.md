# Spring框架

## 1. Spring框架的作用

Spring框架主要解决了创建对象、管理对象的问题。

## 2. Spring框架的依赖项

当项目中需要使用Spring框架时，需要添加的依赖项是：`spring-context`

![image-20221110170131833](images/image-20221110170131833.png)

## 3. Spring框架创建对象的做法

#### Spring框架创建对象有2种做法：

1. 在任何配置类（添加了`@Configuration`）中，自定义方法，返回某种类型的（你需要的）对象，并在方法上添加`@Bean`注解

   - 此方式创建出来的对象，在Spring容器中的名称就是方法名称
   - 此方法应该是`public`
   - 此方法的返回值类型，是你期望Spring框架管理的数据的类型
   - 此方法的参数列表，应该为空
   - 此方法的方法体，应该是自行设计的，没有要求

2. 配置组件扫描，并在组件类上添加组件注解

   - 此方式创建出来的对象，在Spring容器中的名称默认是将类名首字母改为小写

     - 例如：类名是`AdminController`，则对象在Spring容器中的名称为`adminController`
     - 此规则仅适用于类名的第1个字母大写，且第2个字母小写的情况，如果不符合此规则，则对象在Spring容器中的名称就是类名
     - 可以通过组件注解的参数来指定名称

   - 在任何配置类上，添加`@ComponentScan`，当加载此配置类时，就会激活组件扫描

   - 可以配置`@ComponentScan`的参数，此参数应该是需要被扫描的根包（会扫描所配置的包，及其所有子孙包），且此注解参数的值是数组类型的

     - 例如：

     - ```java
       @ComponentScan("cn.tedu")
       ```

   - 如果没有配置`@ComponentScan`的参数中的根包，则组件扫描的范围就是当前类的包及其子孙包

   - 需要在各组件类上添加组件注解，才会被创建对象，常见的组件注解有：

     - `@Component`：通用注解

     - ```
       @Controller
       ```

       ：控制器类的注解

       - `@RestController`：仅添加Spring MVC框架后可使用
       - `@ControllerAdvice`：仅添加Spring MVC框架后可使用(SpringMVC的全局异常处理)
       - `@RestControllerAdvice`：仅添加Spring MVC框架后可使用(SpringMVC的全局异常处理)

     - `@Service`：Service这种业务类的注解

     - `@Repository`：处理数据源中的数据读写的类的注解

   - 以上4种组件注解在Spring框架作用范围之内是完全等效的

   - 在Spring框架中，还有`@Configuration`注解，也是组件注解的一种，但是Spring对此注解的处理更加特殊（Spring框架对配置类使用了代理模式）

对于这2种创建对象的做法，通常：

- 如果是自定义的类，优先使用组件扫描的做法来创建对象

  - 例如配置类、Mapper层、Service层、控制器类。

- 如果不是自定义的类，无法使用组件扫描的做法，只能在配置类中通过`@Bean`方法来创建对象

  - 例如认证权限项目的Security配置类中利用@Bean注解创建密码编码器和认证信息接口

    ```java
    /**
     * 配置密码编译器,否则无法验证密码是否正确,服务端会报错
     * @return 密码编码器
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
    
    /**
     * 重写认证信息接口
     * @return AuthenticationManager
     * @throws Exception Exception
     */
    @Bean// 该注解便于Spring框架进行管理,自动调用,放入容器,利于自动装配
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
    ```


当Spring成功的创建了对象后，会将对象保存在Spring应用程序上下文（`ApplicationContext`）中，后续，当需要这些对象时，可以从Spring应用程序上下文中获取！

由于Spring应用程序上下文中持有大量对象的引用，所以，Spring应用程序上下文也通常被称之为“Spring容器”。

## 4. Spring框架管理的对象的作用域

默认情况下，Spring框架管理的对象都是单例的！

单例：在任何时间点，某个类的对象最多只有1个！

可以在类上添加`@Scope("prototype")`使得被Spring管理的对象是“非单例的”。

> 提示：`@Scope`注解还可以配置为`@Scope("singleton")`，此`singleton`表示“单例的”，是默认的。

默认情况下，Spring框架管理的**单例的**对象是“预加载的”，相当于设计模式中的单例模式的“饿汉式单例模式”。

> 提示：可以在类上添加`@Lazy`注解，使得此对象是“懒加载的”，相当于“懒汉式单例模式”，只会在第1次需要获取对象时才把对象创建出来！

注意：Spring框架并不是使用了设计模式中的“单例模式”，只是从对象的管理方面，对象的作用域表现与单例模式的极为相似而已。

## 5. 自动装配

自动装配：当某个量需要值时，Spring框架会自动的从容器中找到合适的值，为这个量赋值。

自动装配的典型表现是在属性上添加`@Autowired`注解，例如：

```java
@RestController
public class AlbumController {

    // ↓↓↓↓↓  自动装配的典型表现  ↓↓↓↓↓
    @Autowired
    private IAlbumService albumService;
}
```

或者：

```java
@RestController
public class AlbumController {

    private IAlbumService albumService;
    
    //                     ↓↓↓↓↓↓↓  自动装配  ↓↓↓↓↓↓↓
    public AlbumController(IAlbumService albumService) {
        this.albumService = albumService;
    }
}
```

> 提示：Spring创建对象时需要调用构造方法，如果类中仅有1个构造方法（如上所示），Spring会自动调用，如果这唯一的构造方法是有参数的，Spring也会自动从容器中找到合适的对象来调用此构造方法，如果容器没有合适的对象，则无法创建！如果类中有多个构造方法，默认情况下，Spring会自动调用添加了`@Autowired`注解的构造方法，如果多个构造方法都没有添加此注解，则Spring会自动调用无参数的构造方法，如果也不存在无参数构造方法，则会报错！

另外，在配置类中的`@Bean`方法也可以在需要的时候自行添加参数，如果Spring容器中有合适的值，Spring也会从容器中找到值来调用方法。

关于“合适的值”，Spring对于`@Autowired`的处理机制是：查找在Spring容器中**匹配类型的对象**的数量：

- 1个：直接装配，且装配成功
- 0个：取决于`@Autowired`注解的`required`属性
  - `true`（默认值）：装配失败，在加载时即报错----为true说明必须装配
  - `false`：放弃装配，则此量的值为`null`，在接下来的使用过程中可能导致NPE（`NullPointerException`）
- 超过1个：取决于是否存在某个Spring Bean（Spring容器中的对象）的名称**与当前量的名称匹配**
  - 存在：成功装配
  - 不存在：装配失败，在加载时即报错

关于通过名称匹配：

- 默认情况下，要求量（全局变量、方法参数等）的名称与对象在Spring容器中名称完全相同，视为匹配
- 可以在量（全局变量、方法参数等）的声明之前添加`@Qualifier`注解，**通过此注解参数来指定名称**，以匹配某个Spring容器的对象
  - `@Qualifier`注解是用于配合自动装配机制的，单独使用没有意义

其实，还可以使用`@Resource`注解实现自动装配，但不推荐！

Spring框架对`@Resource`注解的自动装配机制是：先根据名称再根据类型来实现自动装配。

`@Resource`是`javax`包中的注解，根据此注解的声明，此注解只能添加在类上、属性上、方法上，不可以添加在构造方法上、方法的参数上。

## 6. 关于IoC与DI

**IoC**：**I**nversion **o**f **C**ontrol，控制反转，即将对象的创建、管理的权力（控制能力）交给框架

**DI**：**D**ependency **I**njection，依赖注入，即为依赖项注入值

**关系:** Spring框架通过 DI 实现/完善 了IoC。

## 7. 关于Spring AOP

------

# 注解:

| 注解             | 所属框架 | 作用                                                         |
| ---------------- | -------- | ------------------------------------------------------------ |
| `@ComponentScan` | Spring   | 添加在配置类上，开启组件扫描。<br />如果没有配置包名，则扫描当前配置类所在的包，<br />如果配置了包名，则扫描所配置的包及其子孙包 |
| `@Component`     | Spring   | 添加在类上，标记当前类是组件类，可以通过参数配置Spring Bean名称 |
| `@Controller`    | Spring   | 添加在类上，标记当前类是控制器组件类，用法同`@Component`     |
| `@Service`       | Spring   | 添加在类上，标记当前类是业务逻辑组件类，用法同`@Component`   |
| `@Repository`    | Spring   | 添加在类上，标记当前类是数据访问组件类，用法同`@Component`   |
| `@Configuration` | Spring   | 添加在类上，仅添加此注解的类才被视为配置类，通常不配置注解参数 |
| `@Bean`          | Spring   | 添加在方法上，标记此方法将返回某个类型的对象，<br />且Spring会自动调用此方法，并将对象保存在Spring容器中 |
| `@Autowired`     | Spring   | 添加在属性上，使得Spring自动装配此属性的值 <br />添加在构造方法上，使得Spring自动调用此构造方法<br />添加在Setter方法上，使得Spring自动调用此方法 |
| `@Qualifier`     | Spring   | 添加在属性上，或添加在方法的参数上， <br />配合自动装配机制，用于指定需要装配的Spring Bean的名称 |
| `@Scope`         | Spring   | 添加在组件类上，或添加在已经添加了`@Bean`注解的方法上，<br />用于指定作用域，注解参数为`singleton`（默认）时为“单例”，注解参数为`prototype`时为“非单例” |
| `@Lazy`          | Spring   | 添加在组件类上，或添加在已经添加了`@Bean`注解的方法上， <br />用于指定作用域，当Spring Bean是单例时，注解参数为`true`（默认）时为“懒加载”，注解参数为`false`时为“预加载” |
| `@Resource`      | Spring   | 此注解是`javax`包中的注解，<br />添加在属性上，使得Spring自动装配此属性的值， <br />通常不推荐使用此注解 |