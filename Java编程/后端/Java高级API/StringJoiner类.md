# StringJoiner类

### 所在包:

- ```java
  java.util.StringJoiner
  ```

### 1.这也是一个处理字符串的类,内部是通过StringBuilder来实现的,线程不安全

### 内部源码实现:

```java
public final class StringJoiner {
    private final String prefix;
    private final String delimiter;
    private final String suffix;

    /*
     * StringBuilder value -- at any time, the characters constructed from the
     * prefix, the added element separated by the delimiter, but without the
     * suffix, so that we can more easily add elements without having to jigger
     * the suffix each time.
     */
    private StringBuilder value;

    /*
     * By default, the string consisting of prefix+suffix, returned by
     * toString(), or properties of value, when no elements have yet been added,
     * i.e. when it is empty.  This may be overridden by the user to be some
     * other value including the empty String.
     */
    private String emptyValue;
	......
}
```

### 2.需要对字符串的结果进行拼接时使用

### 参数:

- ##### delimiter分隔符

- ##### prefix前缀 

- ##### suffix后缀

### 3.举例(处理异常信息时):

```java
@ExceptionHandler
    public JsonResult handlerBindException(BindException e){// 因为该异常的信息不是我们自己定的,而状态信息是我们定的,故调用两参的fail()
        log.debug("开始处理BindException----检查不通过的异常");
        // 第一种方式(将所有异常一同获取并遍历)
        // 需要拼接输出结果时使用(delimiter分隔符 prefix前缀 suffix后缀)
        StringJoiner stringJoiner = new StringJoiner(",","请求参数格式错误,","!!!");
        List<FieldError> fieldErrors = e.getFieldErrors();// 获取该异常的结果集合
        for (FieldError fieldError : fieldErrors){ // 遍历获取的结果集合
            String defaultMessage = fieldError.getDefaultMessage();// 获取validation框架中参数传递错误时@NotNull(message = "?")注解中提示的信息
            stringJoiner.add(defaultMessage);// 将获取的提示信息add到stringJoiner中
        }
        return JsonResult.fail(ServiceCode.ERR_BAD_REQUEST,stringJoiner.toString());
        // 结果:请求参数格式错误,XXX,XXX...,!!!
    }
```

