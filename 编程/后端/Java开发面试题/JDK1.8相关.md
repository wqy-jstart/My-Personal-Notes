# JDK1.8相关

### 1.Lambda表达式中的foreach能否中断？

可以，但没有必要！

集合可以调用foreach进行遍历，执行体是方法，并非循环，所以无法使用break/continue停止(语法报错)，若使用return(语法不报错)效果是忽略当前遍历，其余正常。

故要想停止，则需要抛异常或调用`System.exit(0)`强行终止任务即可！

```java
public class LambdaDemo {
    public static void main(String[] args) {
        String[] names = {"wqy","yaw","wj","wjx","sz","rl"};
        ArrayList<String> list = new ArrayList<>(Arrays.asList(names));
        // 首先forEach中执行的是方法的过程，不能使用break/continue来中断，如果使用return，效果和continue一致，跳过当前循环
        list.forEach(s ->{
            if ("wqy".equals(s)){
                return;
            }
            System.out.println(s);
        }); // System.out::println
        // 结果：yaw wj wjx sz rl
        // 想要目标效果有两种做法：1.抛异常 2.终止系统线程
        // 1.抛异常
        list.forEach(s ->{
            if ("wqy".equals(s)){
                throw new RuntimeException();
            }
            System.out.println(s);
        });
        // 2.终止系统线程任务
        list.forEach(s ->{
            if ("wqy".equals(s)){
                System.exit(0);
            }
            System.out.println(s);
        });
    }
}
```
