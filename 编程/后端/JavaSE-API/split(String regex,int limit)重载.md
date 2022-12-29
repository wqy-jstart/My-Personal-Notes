## split(String regex,int limit)重载:

#### ★字符串拆分split(String regex,int limit)方法重载的演示

#### 效果:由前往后将字符串按照regex拆为limit项

#### 代码如下:

```java
package com.webserver.test;

import java.util.Arrays;

/**
 * ★字符串拆分split(String regex,int limit)方法重载的演示
 *  --------★由前往后将字符串按照regex拆为limit项
 */
public class SplitDemo {
    public static void main(String[] args) {
        String line = "1=2=3=4=5=6=======";
        String[] array = line.split("=");//直接按"="拆,末尾若无内容则省略
        System.out.println(Arrays.toString(array));
        //[1, 2, 3, 4, 5, 6]

        //limit=2 表达仅拆分为两项
        array = line.split("=",2);//由前往后拆2项
        System.out.println(Arrays.toString(array));
        //[1, 2=3=4=5=6=======]

        //limit=3  表达仅拆分为三项
        array = line.split("=",3);//由前往后拆3项
        System.out.println(Arrays.toString(array));
        //[1, 2, 3=4=5=6=======]

        //★当可拆分项不足limit时,仅保留所有可拆分项
        array = line.split("=",100);
        System.out.println(Arrays.toString(array));
        //[1, 2, 3, 4, 5, 6, , , , , , , ]

        //当limit为0时,作用与split(String regex)一致
        array = line.split("=",0);
        System.out.println(Arrays.toString(array));
        //[1, 2, 3, 4, 5, 6]

        //★limit为负数时为应拆尽拆并且全保留(与可拆分项不足时相同)
        array = line.split("=",-1);
        System.out.println(Arrays.toString(array));
        //[1, 2, 3, 4, 5, 6, , , , , , , ]
    }
}
```

