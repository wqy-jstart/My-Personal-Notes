# DateTimeFormatter格式化时间器

#### 用于打印和分析日期时间对象的格式化程序。

```java
/**
 * 根包下的工具类(SOC关注点分离)
 *
 * @Author java@Wqy
 * @Version 0.0.1
 */
public class IdUtils {

    private static DateTimeFormatter dateTimeFormatter
            = DateTimeFormatter.ofPattern("yyMMddHHmmssSSS");// 创建一个格式化时间器

    private static Random random = new Random();

    /**
     * 获取ID值
     *
     * @return 返回id值
     */
    public static long getId() {
        // 【临时策略】使用 yyMMddHHmmssSSS 时间值并拼接在 [1000, 9999] 的随机数字
        LocalDateTime now = LocalDateTime.now(); // 2022-11-17T14:17:33.079 创建一个时间器调用now()返回当前时间
        String dateTimeString = dateTimeFormatter.format(now); // 20221117141959085 调用format()传入时间进行格式化
        int randomNumber = random.nextInt(8999) + 1000; // 获取随机数
        String result = dateTimeString + randomNumber; // 拼接结果
        long id = Long.parseLong(result); // 利用包装类将结果转换成基本类型赋给id
        return id; // 作出返回
    }
}
```

