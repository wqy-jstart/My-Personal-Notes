# JDK1.8新特性

### 1.Lambda表达式

Lambda表达式，也可称为闭包，它是推动Java 8发布的最重要的新特性。

Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中）。

使用Lambda表达式可以使代码变的更加简洁紧凑。

##### 作用：

1. 实现功能性接口（函数式接口）
2. 对集合进行遍历

##### 语法：

```java
// () -> {}
(parameters) -> expression
或
(parameters) -> { statements; }
```

以下是lambda表达式的重要特征:

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定表达式返回了一个数值。

##### Lambda表达式实例：

```java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```

##### 演示案例：

```java
package jdk8;

/**
 * 演示JDK1.8新特性--Lambda表达式
 *
 * @Author java@Wqy
 * @Version 0.0.1
 * @since 2023.1.5
 */
public class Lambda01 {
    public static void main(String args[]) {
        Lambda01 tester = new Lambda01();

        // 接口中方法的定义方式
        // 1.类型声明
        MathOperation addition = (int a, int b) -> a + b; // Integer::sum;

        // 2.不用类型声明
        MathOperation subtraction = (a, b) -> a - b;

        // 3.大括号中的返回语句
        MathOperation multiplication = (int a, int b) -> {
            return a * b;
        };

        // 4.没有大括号及返回语句
        MathOperation division = (int a, int b) -> a / b;

        System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
        System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
        System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
        System.out.println("10 / 5 = " + tester.operate(10, 5, division));

        // 用Lambda表达式返回输出语句
        // 1.不用括号
        GreetingService greetService1 = message ->
                System.out.println("Hello " + message);

        // 2.用括号
        GreetingService greetService2 = (message) ->
                System.out.println("Hello " + message);

        greetService1.sayMessage("Runoob");
        greetService2.sayMessage("Google");
    }

    interface MathOperation {
        int operation(int a, int b);
    }

    interface GreetingService {
        void sayMessage(String message);
    }

    /**
     * 该方法用来返回接口中实现的返回结果
     *
     * @param a             变量a
     * @param b             变量b
     * @param mathOperation 接口MathOperation
     * @return 返回接口中实现后的返回值
     */
    private int operate(int a, int b, MathOperation mathOperation) {
        return mathOperation.operation(a, b);
    }
}
```

输出结果：

```java
10 + 5 = 15
10 - 5 = 5
10 x 5 = 50
10 / 5 = 2
Hello Runoob
Hello Google
```

使用 Lambda 表达式需要注意以下两点：

- Lambda 表达式主要用来定义行内执行的方法类型接口（例如，一个简单方法接口）。在上面例子中，我们使用各种类型的 Lambda 表达式来定义 MathOperation 接口的方法，然后我们定义了 operation 的执行。
- Lambda 表达式免去了使用匿名方法的麻烦，并且给予 Java 简单但是强大的函数化的编程能力。

##### 变量作用域：

lambda 表达式只能引用标记了 final 的外层局部变量，这就是说不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误。

```java
public class Java8Tester {
 
   final static String salutation = "Hello! ";
   
   public static void main(String args[]){
      GreetingService greetService1 = message -> 
      System.out.println(salutation + message);
      greetService1.sayMessage("Runoob");
      // Hello！Runoob
   }
    
   interface GreetingService {
      void sayMessage(String message);
   }
}
```

我们也可以直接在 lambda 表达式中访问外层的局部变量：

```java
public class Java8Tester {
    public static void main(String args[]) {
        final int num = 1;
        Converter<Integer, String> s = (param) -> System.out.println(String.valueOf(param + num));
        s.convert(2);  // 输出结果为 3
    }
 
    public interface Converter<T1, T2> {
        void convert(int i);
    }
}
```

lambda 表达式的局部变量可以不用声明为 final，但是必须不可被后面的代码修改（即隐性的具有 final 的语义）

```java
int num = 1;  
Converter<Integer, String> s = (param) -> System.out.println(String.valueOf(param + num));
s.convert(2);
num = 5;  
//报错信息：Local variable num defined in an enclosing scope must be final or effectively 
 final
```

在 Lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量。

```java
String first = "";  
Comparator<String> comparator = (first, second) -> Integer.compare(first.length(), second.length());  //编译会出错
```

### 2.新的日期时间API

Java 8通过发布新的Date-Time API (JSR 310)来进一步加强对日期与时间的处理。

旧版的Java中，日期和API存在诸多问题，其中有：

1. 非线程安全：java.util.Date是非线程安全的，所有日期类都是可变的，这是Java日期类最大的问题之一。
2. 设计差：Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
3. 时区处理麻烦：日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但同样具有上述问题。

Java 8在java.time包下提供了很多新的API。一下两个比较重要的API：

1. Local（本地）：简化日期时间的处理，不存在时区的问题。
2. Zoned（时区）：通过制定的时区处理日期和时间。

##### 本地化时间API：

LocalDate/LocalTime 和 LocalDateTime 类可以在处理时区不是必须的情况。代码如下：

```java
package jdk8;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.Month;

/**
 * 演示JDK1.8新日期：LocalDateTime等相关API
 *
 * @author java@Wqy
 * @version 0.0.1
 * @since 2023.1.5
 */
public class LocalDateTime01 {
    public static void main(String args[]){
        LocalDateTime01 java8tester = new LocalDateTime01();
        java8tester.testLocalDateTime();
    }

    public void testLocalDateTime(){

        // 获取当前的日期时间
        LocalDateTime currentTime = LocalDateTime.now();
        System.out.println("当前时间: " + currentTime);// 2023-01-05T12:04:45.295

        // 转化本地时间: toLocalDate()方法
        LocalDate date1 = currentTime.toLocalDate();
        System.out.println("date1: " + date1); // 2023-01-05

        // 获取月份: getMonth()方法
        Month month = currentTime.getMonth();
        // 获取日: getDayOfMonth()方法
        int day = currentTime.getDayOfMonth();
        // 获取秒: getSecond()方法
        int seconds = currentTime.getSecond();

        System.out.println("月: " + month +", 日: " + day +", 秒: " + seconds);

        // 根据指定年月获取东八区时间: withDayOfMonth()/withYear()
        LocalDateTime date2 = currentTime.withDayOfMonth(10).withYear(2012);
        System.out.println("date2: " + date2);

        // 12 december 2014
        LocalDate date3 = LocalDate.of(2014, Month.DECEMBER, 12);
        System.out.println("date3: " + date3);

        // 22 小时 15 分钟
        LocalTime date4 = LocalTime.of(22, 15);
        System.out.println("date4: " + date4);

        // 解析字符串
        LocalTime date5 = LocalTime.parse("20:15:30");
        System.out.println("date5: " + date5);
    }
}
```

##### 使用时区的日期时间API：

如果我们考虑到时区，就可以使用下面的API：

```java
package jdk8;

import java.time.ZoneId;
import java.time.ZonedDateTime;

/**
 * 演示面对时区时的问题: ZonedDateTime
 *
 * @author Java@Wqy
 * @version 0.0.1
 * @since 2023.1.5
 */
public class LocalDateTime02 {
    // 如果我们需要考虑到时区，就可以使用时区的日期时间API
    public static void main(String args[]){
        LocalDateTime02 java8tester = new LocalDateTime02();
        java8tester.testZonedDateTime();
    }

    public void testZonedDateTime(){

        // 获取当前时间日期
        ZonedDateTime date1 = ZonedDateTime.parse("2015-12-03T10:15:30+05:30[Asia/Shanghai]");
        System.out.println("date1: " + date1);// 2015-12-03T10:15:30+08:00[Asia/Shanghai]

        ZoneId id = ZoneId.of("Europe/Paris");
        System.out.println("ZoneId: " + id);// Europe/Paris

        ZoneId currentZone = ZoneId.systemDefault();
        System.out.println("当期时区: " + currentZone);// Asia/Shanghai
    }
}
```

### 3.方法的引用：`::`

方法的引用通过方法的名字来指向一个方法。

方法引用可以使语言的构造更加紧凑简洁，减少冗余代码。

方法的引用可以使用一对冒号来实现：`::`

```java
package jdk8;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.function.Supplier;

/**
 * 演示JDK1.8方法的引用-- ::
 *
 * @author java@Wqy
 * @version 0.0.1
 * @since 2023.1.5
 */
public class NewMethodUse {
    public static void main(String[] args) {
        // 构造器引用 Class::new
        final Car car = Car.create(Car::new);
        final List<Car> cars = Arrays.asList(car);
        // 静态方法的引用 Class::static_method
        cars.forEach(Car::collide);
        // 特定类的任意对象的方法引用 Class::method
        cars.forEach(Car::repair);
        // 特定对象的方法引用 instance::method
        final Car police = Car.create(Car::new);
        cars.forEach(police::follow);

        // 方法引用实例
        List<String> names = new ArrayList<>();
        names.add("Google");
        names.add("Runoob");
        names.add("Taobao");
        names.add("Baidu");
        names.add("Sina");
        names.forEach(System.out::println);
    }
}

// 该类中定义各种方法
class Car {
    public static Car create(final Supplier<Car> supplier) {
        return supplier.get();
    }

    public static void collide(final Car car) {
        System.out.println("Collided " + car.toString());
    }

    public void follow(final Car another) {
        System.out.println("Following the " + another.toString());
    }

    public void repair() {
        System.out.println("Repaired " + this.toString());
    }
}
```