# Collection-ArrayList

## 概述

ArrayList实现了List接口，是有序容器，即元素存放的数据与放进去的顺序相同，允许放入Null元素，底层通过**数组实现**。除该类未实现同步外，其余跟Vector大致相同。

每个ArrayList都有一个容量(capacity)，表示底层数组的实际大小，容器内存储元素的个数不能多于当前容量。当向容器中添加元素时，如果容量不足，容器会自动增大数组的大小，前面已经提过，Java泛型只是编译器提供的语法糖，所以这里的数组是一个Object数组，以便能够容纳任何类型的对象。

![ArrayList_base.png](https://pdai.tech/images/collection/ArrayList_base.png)

1. size(), isEmpty(), get(), set()方法均能在常数时间内完成
2. add()方法的时间开销跟插入位置有关
3. addAll()方法的时间开销跟添加元素的个数成正比。

其余方法大都是线性时间。

**为追求效率，ArrayList没有实现同步(synchronized)**，如果需要多个线程并发访问，用户可以手动同步，也可使用Vector替代。

## ArrayList的实现

### 1.底层数据结构

```java
/**
 *存储ArrayList元素的数组缓冲区。
 *ArrayList的容量是此数组缓冲区的长度。任何
 *具有elementData==DEFAULTCAPACITY_empty_elementData的空ArrayList
 *将在添加第一个元素时扩展为DEFAULT_CAPACITY。
 */
transient Object[] elementData; // 非私有以简化嵌套类访问

/**
 * ArrayList的大小（它包含的元素数）。
 *
 * @serial
 */
private int size;
```

### 2.构造函数

```java
/**
 * 构造具有指定初始容量的空列表。
 *
 * @param  initialCapacity  列表的初始容量
 * @throws IllegalArgumentException 如果指定的初始容量为负值
 *
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+initialCapacity);
    }
}

/**
 * 构造一个初始容量为10的空列表。
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * 构造一个包含指定集合，按照集合的返回顺序迭代器。
 *
 * @param c 要将其元素放置到此列表中的集合
 * @throws NullPointerException 如果指定的集合为空
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // 替换空数组
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

### 3.自动扩容

每当向数组中添加元素时，都要去检查添加后元素的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。数组扩容通过一个公开的方法`ensureCapacity(int minCapacity)`来实现。在实际添加大量元素前，我也可以使用`ensureCapacity`来手动增加ArrayList实例的容量，以减少递增式再分配的数量。

**数组进行扩容时，会将老数组中的元素重新拷贝一份到新的数组中，每次数组容量的增长大约是其原容量的1.5倍**。

这种操作的代价是很高的，因此在实际使用时，我们应该尽量避免数组容量的扩张。

当我们可预知要保存的元素的多少时，要在构造ArrayList实例时，就指定其容量，以避免数组扩容的发生。或者根据实际需求，通过调用`ensureCapacity`方法来手动增加ArrayList实例的容量。

```java
/**
 * 增加实例ArrayList的容量
 * 如果必要，以确保它至少可以容纳一定数量的元素由最小容量参数决定
 *
 * @param   minCapacity   所需的最小容量
 */
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)? 0 : DEFAULT_CAPACITY;
    // 如果不是默认元素表，则为任意大小
    // 大于默认空表的默认值。它已经被认为是默认大小。
    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

/**
 * 要分配的数组的最大大小。
 * 一些虚拟机在数组中保留一些标题字。
 * 尝试分配更大的阵列可能会导致
 * OutOfMemoryError：请求的数组大小超过VM限制
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 * 增加容量，以确保它至少可以容纳最小容量参数指定的元素数。
 *
 * @param minCapacity 所需的最小容量
 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 复制
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // 溢出
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?Integer.MAX_VALUE : MAX_ARRAY_SIZE;
}
```

![ArrayList_grow.png](https://pdai.tech/images/collection/ArrayList_grow.png)

### 4.add(), addAll() 

跟C++ 的*vector*不同，*ArrayList*没有`push_back()`方法，对应的方法是`add(E e)`，*ArrayList*也没有`insert()`方法，对应的方法是`add(int index, E e)`。

这两个方法都是向容器中添加新元素，这可能会导致*capacity*不足，因此在添加元素之前，都需要进行剩余空间检查，如果需要则自动扩容。扩容操作最终是通过`grow()`方法完成的。

```java
/**
 * 将指定的元素追加到此列表的末尾。
 *
 * @param e 要附加到此列表的元素
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // 增加 modCount!!
    elementData[size++] = e;
    return true;
}

/**
 * 在此中的指定位置插入指定元素列表。移动当前位于该位置的元素（如果有）
 * 右侧的任何后续元素（在其索引中添加一个）。
 *
 * @param index 插入指定元素的索引
 * @param element 要插入的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // 增加 modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,size - index);
    elementData[index] = element;
    size++;
}
```

![ArrayList_add.png](https://pdai.tech/images/collection/ArrayList_add.png)

`add(int index, E e)`需要先对元素进行移动，然后完成插入操作，也就意味着该方法有着线性的时间复杂度。

`addAll()`方法能够一次添加多个元素，根据位置不同也有两个版本，一个是在末尾添加的`addAll(Collection<? extends E> c)`方法，一个是从指定位置开始插入的`addAll(int index, Collection<? extends E> c)`方法。

跟`add()`方法类似，在插入之前也需要进行空间检查，如果需要则自动扩容；如果从指定位置插入，也会存在移动元素的情况。 `addAll()`的时间复杂度不仅跟插入元素的多少有关，也跟插入的位置相关。

```java
/**
 * 将指定集合中的所有元素追加到此列表，按返回顺序指定集合的迭代器。此操作的行为是
 * 如果在操作期间修改了指定的集合，则未定义正在进行中。
 *（这意味着此调用的行为是如果指定的集合是此列表，则未定义列表不是空的。）
 *
 * @param c 包含要添加到此列表中的元素的集合
 * @return <tt>true</tt> 如果此列表因调用而更改
 * @throws NullPointerException 如果指定的集合为空
 */
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // 扩容
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

/**
 * 将指定集合中的所有元素插入列表，从指定位置开始。
 * 移动元素当前在该位置（如果有）以及任何后续元素右侧（增加其指数）。
 * 新元素将出现在列表中按指定集合的迭代器。
 *
 * @param index 从中插入第一个元素的索引指定的集合
 * @param c 包含要添加到此列表中的元素的集合
 * @return <tt>true</tt> 如果此列表因调用而更改
 * @throws IndexOutOfBoundsException {@inheritDoc}
 * @throws NullPointerException 如果指定的集合为空
 */
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // 扩容

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

### 5.set()

既然底层是一个数组*ArrayList*的`set()`方法也就变得非常简单，直接对数组的指定位置赋值即可。

```java
public E set(int index, E element) {
    rangeCheck(index);//下标越界检查
    E oldValue = elementData(index);
    elementData[index] = element;//赋值到指定位置，复制的仅仅是引用
    return oldValue;
}
```

### 6.get()

`get()`方法同样很简单，唯一要注意的是由于底层数组是Object[]，得到元素后需要进行类型转换。

```java
public E get(int index) {
    rangeCheck(index);
    return (E) elementData[index];//注意类型转换
}
```

### 7.remove()

`remove()`方法也有两个版本，一个是`remove(int index)`删除指定位置的元素，另一个是`remove(Object o)`删除第一个满足`o.equals(elementData[index])`的元素。

删除操作是`add()`操作的逆过程，需要将删除点之后的元素向前移动一个位置。需要注意的是为了让GC起作用，必须显式的为最后一个位置赋`null`值。

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; //清除该位置的引用，让GC起作用
    return oldValue;
}
```

关于Java GC这里需要特别说明一下，**有了垃圾收集器并不意味着一定不会有内存泄漏**。对象能否被GC的依据是是否还有引用指向它，上面代码中如果不手动赋`null`值，除非对应的位置被其他元素覆盖，否则原来的对象就一直不会被回收

### 8.trimToSize()

ArrayList还给我们提供了将底层数组的容量调整为当前列表保存的实际元素的大小的功能。它可以通过trimToSize方法来实现。代码如下:

```java
/**
 * Trims the capacity of this <tt>ArrayList</tt> instance to be the
 * list's current size.  An application can use this operation to minimize
 * the storage of an <tt>ArrayList</tt> instance.
 */
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0) ? EMPTY_ELEMENTDATA : Arrays.copyOf(elementData, size);
    }
}
```

### 9.indexOf(), lastIndexOf()

获取元素的第一次出现的index:

```java
/**
 * 返回指定元素第一次出现的索引,如果此列表不包含元素，则为-1。
 * 更正式地说，返回最低索引<tt>i</tt>，这样
 * <tt>（o==null？get（i）==null：&nbsp；o.equals（get（i）））</tt>，
 * 或-1，如果没有这样的索引。
 */
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

获取元素的最后一次出现的index:

```java
/**
 * 返回指定元素最后一次出现的索引, 如果此列表不包含元素，则为-1。
 * 更正式地说，返回最低索引<tt>i</tt>，这样
 * <tt>（o==null？get（i）==null：&nbsp；o.equals（get（i）））</tt>，
 * 或-1，如果没有这样的索引。
 */
public int lastIndexOf(Object o) {
    if (o == null) {
        // 颠倒遍历
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

### 10.Fail-Fast机制

ArrayList也采用了快速失败的机制，通过记录`modCount`参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

## 参考

- 深入Java集合学习系列: ArrayList的实现原理 http://zhangshixi.iteye.com/blog/674856

- Java ArrayList源码剖析 结合源码对ArrayList进行讲解 http://www.cnblogs.com/CarpenterLee/p/5419880.html