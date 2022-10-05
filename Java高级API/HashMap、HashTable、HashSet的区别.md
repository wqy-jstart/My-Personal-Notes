# HashMap、HashTable、HashSet的区别:

- ## HashMap

  - HashMap实现了Map接口，Map接口对键-值对进行映射。Map中不允许重复的键。
  - HashMap里面存入的键-值对在取出的时候没有固定的顺序。
  - HashMap允许键和值为null。HashMap是非synchronized的(非线程安全)，但collection框架提供了方法Collections.synchronizedMap()能保证HashMap synchronized，这样多个线程同时访问HashMap时，能保证只有一个线程更改Map。
  - public Object put(Object Key,Object value)方法用来将元素添加到map中。

- ## 哈希表Hashtable

  - 哈希表（Hashtable）又称为“散置”，Hashtable是会根据索引键的哈希程序代码组织成的索引键（Key）和值（Value）配对的集合。
  - Hashtable 对象是由包含集合中元素的哈希桶（Bucket）所组成的。而Bucket是Hashtable内元素的虚拟子群组，可以让大部分集合中的搜寻和获取工作更容易、更快速。

- ## HashSet

  - HashSet实现了Set接口，它不允许集合中出现重复元素。当我们提到HashSet时，第一件事就是在将对象存储在HashSet之前，要确保重写hashCode（）方法和equals（）方法，这样才能比较对象的值是否相等，确保集合中没有储存相同的对象。

## HashMap与HashTable的比较:

![image-20221005145533156](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221005145533156.png)

## HashMap与HashSet的比较:

![image-20221005145739540](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221005145739540.png)