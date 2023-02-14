# asList()、toArray()、suList()

### 1.数组转集合:

```java
//★源码如下:
public static <T> List<T> asList(T... a) {// 需要包装类型的数组
        return new ArrayList<>(a);
}
```

- ```java
  public class ListDemo3 {
      public static void main(String[] args) {
          //数组没有重写toString()方法,故需调用Arrays类中的toString()方法来输出
          String[] array = {"one","two","three","four","five"};
          System.out.println("array:"+ Arrays.toString(array));
  
          //将数组转化为集合，存储了数组的引用，指向了同一个对象
          List<String> list = Arrays.asList(array);
          System.out.println("list:"+list);
  
          //修改数组，会影响集合
          array[2] = "six";
          System.out.println("array:"+Arrays.toString(array));
          System.out.println("list:"+list);
  
          //修改集合，同样会影响数组
          list.set(3,"seven");
          System.out.println("array:"+Arrays.toString(array));
          System.out.println("list:"+list);
      }
  }
  ```

### 2.集合转数组: 

```java
//★源码如下:
<T> T[] toArray(T[] a);
```

- ```java
  public class CollectionToArrayDemo {
      public static void main(String[] args) {
          Collection<String> c = new ArrayList<>();
          c.add("one");
          c.add("two");
          c.add("three");
          c.add("four");
          c.add("five");
          System.out.println(c);
          //若参数数组元素个数==集合元素个数，那就正常转换
          //若参数数组元素个数>集合元素个数，则正常转换，同时末尾补默认值null
          //若参数数组元素个数<集合元素个数，则会按照集合大小转给数组
          String[] arr = new String[c.size()];
          //toArray()方法中需要传一个数组以供转换
          String[] array = c.toArray(arr);
          //String[] array = c.toArray(new String[c.size()]);也可以
          System.out.println(Arrays.toString(array));
      }
  }
  ```

### 3.获取集合的子集: 

```java
//★源码如下:
List<E> subList(int fromIndex, int toIndex);//范围,含头不含尾
```

- ```java
  public class ListDemo2 {
      public static void main(String[] args) {
          List<Integer> list = new ArrayList<>();//该集合只能装Integer类型
          for (int i = 0; i < 10; i++) {
              list.add(i*10); //自动装箱
          }
          System.out.println(list);//[0, 10, 20, 30, 40, 50, 60, 70, 80, 90]
  
          //获取下标3到7的子集
          List<Integer> subList = list.subList(3,8);
          System.out.println(subList);//[30, 40, 50, 60, 70]
  
          for (int i = 0; i < subList.size(); i++) {
              subList.set(i,subList.get(i)*10);
          }
          System.out.println(subList);//[300, 400, 500, 600, 700]
  
          //注意：对子集的操作就是对原集合对应元素操作
          System.out.println(list);//[0, 10, 20, 300, 400, 500, 600, 700, 80, 90]
  
          list.remove(0);
          System.out.println(list);
          //一旦做了子集，则原集合不能做修改，否则与子集元素不对应会报错，但可以重新获取子集
          //System.out.println(subList);//发生不支持操作的编译异常
  
      }
  }
  ```

### 4.字符串转char[]

```java
String num = "java";
char[] chars = num.toCharArray();
```

### 5.int[]数组转Integer[]包装类数组：

在Java 8中，`int[]`可以轻松转换为`Integer[]`：

> 当出现传递int[]数组，但需要使用集合必须传递包装类型的对象时可使用：

```java
public static void main(String[] args) {
    int[] data = {1,2,3,4,5,6,7,8,9,10};

    // To boxed array
    Integer[] what = Arrays.stream( data ).boxed().toArray( Integer[]::new );
    Integer[] ever = IntStream.of( data ).boxed().toArray( Integer[]::new );

    // To boxed list
    List<Integer> you  = Arrays.stream( data ).boxed().collect( Collectors.toList() );
    List<Integer> like = IntStream.of( data ).boxed().collect( Collectors.toList() );
}
```
