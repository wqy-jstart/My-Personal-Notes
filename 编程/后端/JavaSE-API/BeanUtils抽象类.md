# BeanUtils抽象类

#### 所在包为Spring提供的类

```java
org.springframework.beans.BeanUtils
```

#### 浅拷贝:

- 对基本数据类型进行值传递,对引用数据类型,使用其引用地址,不拷贝其内容,此为浅拷贝

#### 深拷贝:

- 对基本数据类型进行值传递,对引用数据类型,创建一个新的对象,并复制其内容,此为深拷贝.

### 建议使用Spring提供的BeanUtils(无需捕获异常)

#### 该类主要提供了对于JavaBean进行各种操作,比如对象,属性复制等.

|                           主要方法                           |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                    cloneBean(Object bean)                    |                          对象的克隆                          |
|           copyProperties(Object dest, Object orig)           |                   把对象origcopy到对象dest                   |
|     copyProperty(Object bean, String name, Object value)     |                 拷贝指定的属性值到指定的bean                 |
|     setProperty(Object bean, String name, Object value)      |                       设置指定属性的值                       |
| populate(Object bean, Map<String,? extends Object> properties) | 将map数据拷贝到javabean中（map的key要与javabean的属性名称一致） |

