# 关于RESTful

### RESTful是一种软件的设计风格（不是规定，也不是规范）。

> RESTFUL是一种网络应用程序的设计风格和开发方式，基于HTTP，可以使用XML格式定义或JSON格式定义。RESTFUL适用于移动互联网厂商作为业务接口的场景，实现第三方OTT调用移动网络资源的功能，动作类型为新增、变更、删除所调用资源。

### RESTful的典型表现有：

- #### 一定是响应正文的

  - 服务器端处理完请求后将响应数据，不会由服务器响应页面到客户端

- #### 通常会将具有唯一性的请求参数设计到URL中，成为URL的一部分

  - `https://blog.csdn.net/gghbf45219/article/details/1045245854`
  - `https://blog.csdn.net/m4465sfd46/article/details/1042276671`

- #### 严格区分4种请求方式，

  **在许多业务系统中并不这样设计**

  - 尝试增加数据，使用`POST`请求方式
  - 尝试删除数据，使用`DELETE`请求方式
  - 尝试修改数据，使用`PUT`请求方式
  - 尝试查询数据，使用`GET`请求方式

Spring MVC框架很好的支持了RESTful风格的设计，当需要在URL中使用变量值时，可以使用`{自定义名称}`作为占位符，并且，在方法的参数列表中，自定义参数接收此变量值，在参数前还需要添加`@PathVariable`注解，例如：

```java
// http://localhost:8080/album/9527/delete
@RequestMapping("/{id}/delete")
public String delete(@PathVariable String id) {
    String message = "尝试删除id值为【" + id + "】的相册";
    log.debug(message);
    return message;
}
```

注意：通常会将占位符中的名称和方法的参数名称保持一致，例如以上的`{id}`和`String id`都使用`id`作为名字，如果因为某些原因无法保持一致，则需要配置`@PathVariable`注解的参数，此注解参数值与占位符中的名称一致即可，方法的参数名称就不重要了。

在开发实践中，可以将处理请求的方法的参数类型设计为期望的类型，例如将`id`设计为`Long`类型的，但是，如果这样设计，必须保证请求中的参数值是可以被正确转换为`Long`类型的，否则会出现`400`错误！

为了尽量保证匹配的准确性、保证参数值可以正常转换，在设计占位符时，可以在占位符名称右侧添加冒号，并在冒号右侧使用**正则表达式**来限制占位符的值的格式，例如：

```java
@RequestMapping("/{id:[0-9]+}/delete")
```

##### 注意，一旦使用正则表达式后，多个不冲突的占位符的设计是允许共存在的，例如：

```java
// http://localhost:8080/album/9527/delete
@RequestMapping("/{id:[0-9]+}/delete")
public String delete(@PathVariable Long id) {
    String message = "尝试删除id值为【" + id + "】的相册";
    log.debug(message);
    return message;
}

// http://localhost:8080/album/hello/delete
@RequestMapping("/{name:[a-z]+}/delete")
public String delete(@PathVariable String name) {
    String message = "尝试删除名称值为【" + name + "】的相册";
    log.debug(message);
    return message;
}
```

##### 另外，没有使用占位符的设计，与使用了占位符的设计，也是允许共存的，例如：

```java
// http://localhost:8080/album/hello/delete
@RequestMapping("/{name:[a-z]+}/delete")
public String delete(@PathVariable String name) {
    String message = "尝试删除名称值为【" + name + "】的相册";
    log.debug(message);
    return message;
}

// http://localhost:8080/album/test/delete
@RequestMapping("/test/delete")
public String delete() {
    String message = "尝试测试删除相册";
    log.debug(message);
    return message;
}
```

##### 最后，关于RESTful风格的URL设计，如果没有明确的要求，或没有更好的选择，可以设计为：

- 获取数据列表：`/数据类型的复数`
  - 例如：`/albums`
- 根据id获取数据：`/数据类型的复数/id值`
  - 例如：`/albums/1`
- 根据id对数据执行某种操作：`/数据类型的复数/id值/命令`
  - 例如：`/albums/1/delete`