# Java集合框架

## 1.Collection接口

### List集合：

##### （1）ArrayList

- 存储结构：
  1. 基于数组存储，java中的数组是定长的

- 应用特点：
  1. 允许元素重复
  2. 可以保证元素的顺序

- 优势：
  1. 随机访问速度快，因为基于数组访问，有下标
  2. 尾部插入速度比较快

- 劣势：
  1. 线程不安全
  2. 在内存中需要连续
  3. 随机插入数据的速度比较慢，插入、删除时需要进行移动，麻烦！

##### （2）LinkedList

- 存储结构：
  1. 基于链表实现存储，序列的
  2. jdk1.7使用双向循环链表，1.8使用双向链表

- 应用特点：
  1. 允许元素重复
  2. 可以保证元素的添加顺序，双向链表的每一个元素前后都有Node节点，也有对应的下标

- 优势：
  1. 随机插入和删除元素的速度快
  2. 头和尾插入速度更快

- 劣势：
  1. 线程不安全
  2. 支持顺序读取，但是查询效率比较慢，不支持随机读取，内部需要找到前后的节点来定位元素

##### （3）Vector

- 历史遗留，现在基本已经不实用了
- 线程安全的ArrayList对象，对整个集合添加排他锁，效率低

##### （4）CopyOnWriteArrayList

- 存储结构：
  1. 数组

- 应用特点：
  1. 允许元素重复
  2. 可以保证元素的添加顺序
  3. 支持并发环境下的多线程共享应用

- 优点：
  1. 基于乐观锁实现线程安全：更新时先拷贝数组，在自己的线程中进行更新，最后将更新的数据写到主内存！
  2. 底层使用CAS（比较和交换）算法：减少阻塞，减少线程间的切换

- 劣势：
  1. 并发较多时效率低

### Set集合：

##### （1）HashSet

- 存储结构：
  1. 散列表，将值存储到HashMap对象的key位置

- 应用特点：
  1. 不能保证元素的添加顺序
  2. 不允许元素重复

- 优势：
  1. 读取效率相对较高

- 劣势：
  1. 线程不安全

##### （2）TreeSet

- 存储结构：
  1. 底层实现红黑树数据结构

- 应用特点：
  1. 不允许元素重复
  2. 可以对添加的元素进行排序

- 优势：
  1. 可以对元素进行排序

- 劣势：
  1. 相对HashSet可能慢一些

##### （3）LinkedHashSet

- 添加链表的HashSet对象

- 基于链表记录元素的添加顺序

#### 参考：https://www.processon.com/view/link/635c86b86376897317161c22#map