# Java中default关键字

## 1.简介

★:default关键字和Java中的public、[private](https://so.csdn.net/so/search?q=private&spm=1001.2101.3001.7020)等关键字一样，都属于修饰符关键字，可以用来修饰属性、方法以及类，**但是default一般用来修饰接口中的方法**。

## 2.原因

出现该关键字的原因是，由于接口在Java中定义之初，有一个缺点，那就是，如果定义了一个接口，接口中又定义了N个方法，那么某个具体的类在实现该接口时，需要实现接口中的所有方法，不管是否需要用到接口中的方法。如果接口中的某个方法被default关键字修饰了，那么具体的实现类中可以不用实现方法。

## 3.举例说明

```java
public interface Person {
    default void show(){
        System.out.println("this is show");
    }
}
```

> #### Student类可以不用实现Person接口中的show()方法。

```java
class Student implements Person{
   //可以不用实现show()方法
}
```

> #### 若实现了接口中的方法.

```java
class Student implements Person{
   //实现时
    @Override
    public void show() {
        System.out.println(Student.class.getName());
    }
} 
```



## 4.解决冲突

#### 如果实现类实现了个多个接口，假如不同的接口中有同名的被default修饰的方法，那么此时，实现类就必须重写这个方法，否则会编译出错。

#### Person1接口:

```java
public interface Person1{
    default void prinN(){
        System.out.println(Person1.class.getName());
    }
}
```

#### Person2接口:

```java
public interface Person2{
    default void prinN(){
        System.out.println(Person2.class.getName());
    }
}
```

#### Student实现类:

```java
public class Student implements Person1,Person2{
    @Override
    public void prinN(){
        System.out.println(Student.class.getName());
    }
}
```

