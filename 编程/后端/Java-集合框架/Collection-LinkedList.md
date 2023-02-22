# Collection-LinkedList

## 概述

LinkedList同时实现了List和Queue接口，也就是说它既可以看作一个有序容器，也可以看作一个队列(Queue)，一个栈(Stack)。

当你考虑使用队列和栈时，可以考虑使用LinkedList。

原因是因为Java官方已经声明不建议使用Stack类。

<u>关于栈和队列，现在的首选是ArrayQueue，它有着比LinkedList更好的性能！</u>

![img](https://pdai.tech/images/collection/LinkedList_base.png)

<u>*LinkedList*的实现方式决定了所有跟下标相关的操作都是线性时间</u>，而在首段或者末尾删除元素只需要常数时间。为追求效率*LinkedList*没有实现同步(synchronized)，如果需要多个线程并发访问，可以先采用`Collections.synchronizedList()`方法对其进行包装。

## LinkedList实现

### 1.底层数据结构

LinkedList底层通过**双向链表**实现。

双向链表的每个节点用内部类*Node*表示。*LinkedList*通过`first`和`last`引用分别指向链表的第一个和最后一个元素。注意这里没有所谓的哑元，当链表为空的时候`first`和`last`都指向`null`。

```java
transient int size = 0;

/**
 * 指针指向第一个节点
 * Invariant: (first == null && last == null) ||
 *            (first.prev == null && first.item != null)
 */
transient Node<E> first;

/**
 * 指针指向末尾节点
 * Invariant: (first == null && last == null) ||
 *            (last.next == null && last.item != null)
 */
transient Node<E> last;
```

其中Node是一个私有内部类：

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### 2.构造函数

```java
/**
 * 构造一个空的集合
 */
public LinkedList() {
}

/**
 * 构造一个包含指定元素的集合
 * 按照集合返回的顺序迭代
 *
 * @param  c 要将其元素放置到此列表中的集合
 * @throws NullPointerException 如果指定的集合为空
 */
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

### 3.getFirst(),getLast()

获取第一个元素，获取最后一个元素。

```java
/**
 * 返回集合中的第一个元素
 *
 * @return 集合中第一个元素
 * @throws NoSuchElementException 如果这个集合为空
 */
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

/**
 * 返回集合中最后一个元素
 *
 * @return 集合中最后一个元素
 * @throws NoSuchElementException 如果集合为空
 */
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
```

### 4.remove(...)

`removeFirst()`、 `removeLast()`、`remove(e)`、`remove(index)`

`remove()`方法也有两个版本，一个是删除跟指定元素相等的第一个元素`remove(Object o)`，另一个是删除指定下标处的元素`remove(int index)`。

![img](https://pdai.tech/images/collection/LinkedList_remove.png)

删除元素 - 指的是删除**第一次出现的这个元素**, 如果没有这个元素，则返回false；

判断的依据是equals方法， 如果equals，则直接unlink这个node；

由于LinkedList可存放null元素，故也可以删除第一次出现null的元素；

```java
/**
 * 从该列表中删除第一次出现的指定元素，如果它存在的话
 * 如果此列表不包含该元素，则不变
 * 更正式的，删除索引最小的元素
 * {@code i} 使得
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
 * (如果存在这样的元素).  如果此列表为空，则返回｛@code true｝
 * 包含指定的元素（或等效地，如果此列表作为呼叫的结果而改变）。
 *
 * @param o 要从此列表中删除的元素（如果存在）
 * @return {@code true} 如果此列表包含指定的元素
 */
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

/**
 * 取消链接非空节点x。
 */
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {// 第一个元素
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {// 最后一个元素
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null; // GC
    size--;
    modCount++;
    return element;
}
```

`remove(int index)`使用的是下标计数， 只需要判断该index是否有元素即可，如果有则直接unlink这个node。

```java
/**
 * 删除此列表中指定位置的元素。移动所有左边的后续元素（从其索引中减去一个）。
 * 返回从列表中删除的元素。
 *
 * @param index 要删除的元素的索引
 * @return 先前位于指定位置的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

删除head元素：`removeFirst()`

```java
/**
 * 移除并返回集合列表中的第一个元素
 *
 * @return 列表中的第一个元素
 * @throws NoSuchElementException 如果list为空
 */
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}


/**
 * 取消链接非空的第一个节点f。
 */
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```

删除last元素：`removeLast()`

```java
/**
 * 移除并返回集合列表中的最后一个元素
 *
 * @return 列表中的最后一个元素
 * @throws NoSuchElementException 如果列表为空
 */
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

/**
 * 取消链接非空的最后一个节点l。
 */
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    last = prev;
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
}
```

### 5.add()

add()方法有两个版本，一个是`add(E e)`，该方法在LinkedList的末尾插入元素，**因为有`last`指向链表末尾**，在末尾插入元素的花费是常数时间。只需要简单修改几个相关引用即可；

另一个是`add(int index, E element)`，该方法是在指定下表处插入元素，需要先通过线性查找找到具体位置，然后修改相关引用完成插入操作。

```java
/**
 * 在列表的末尾追加元素
 *
 * <p>此方法等效于{@link #addLast}.
 *
 * @param e 追加到列表末尾的元素
 * @return {@code true} (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    linkLast(e);
    return true;
}

/**
 * 链接末尾元素
 */
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

![img](https://pdai.tech/images/collection/LinkedList_add.png)

`add(int index, E element)`, 当index==size时，等同于add(E e); 

如果不是，则分两步: 

1.先根据index找到要插入的位置,即node(index)方法；

2.修改引用，完成插入操作。

```java
/**
 * 在此列表中的指定位置插入指定元素。
 * 移动当前位于该位置的元素（如果有）右侧的后续元素（在其索引中添加一个）。
 *
 * @param index 插入指定元素的索引
 * @param element 要插入的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element); // 相当于末尾添加：linkLast()
    else
        linkBefore(element, node(index));
}
```

上面代码中的`node(int index)`函数有一点小小的trick，因为链表双向的，可以从开始往后找，也可以从结尾往前找，具体朝那个方向找取决于条件`index < (size >> 1)`，也即是index是靠近前端还是后端。

**从这里也可以看出，linkedList通过index检索元素的效率没有arrayList高。**

```java
/**
 * 返回指定元素索引处的（非空）节点。
 */
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

### 6.addAll()

addAll(index, c) 实现方式并不是直接调用add(index,e)来实现，主要是因为效率的问题，另一个是fail-fast(快速失败)中modCount只会增加1次；

```java
/**
 * 将指定集合中的所有元素追加到此列表,按指定的集合的迭代器。
 * 如果在操作过程中修改了指定的集合，则此操作的行为未定义.（请注意，如果指定的集合是这个列表，它不是空的。）
 *
 * @param c 包含要添加到此列表中的元素的集合
 * @return {@code true} 如果此列表因调用而更改
 * @throws NullPointerException 如果指定的集合为空
 */
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}

/**
 * 将指定集合中的所有元素插入到这个列表，从指定位置开始。移动元素
 * 将当前位于该位置的元素（如果有）和任何后续元素向右移动（增加其索引）。
 * 新元素将按指定集合的迭代器返回的顺序出现在列表中
 *
 * @param index 从指定集合中插入第一个元素的索引
 * @param c 包含要添加到此列表中的元素的集合
 * @return {@code true} 如果此列表因调用而更改
 * @throws IndexOutOfBoundsException {@inheritDoc}
 * @throws NullPointerException 如果指定的集合为空
 */
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;

    Node<E> pred, succ;
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }

    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

### 7.clear()

为了让GC更快可以回收放置的元素，需要将node之间的引用关系赋空。

```java
/**
 * 删除此列表中的所有元素。
 * 此调用返回后，列表将为空。
 */
public void clear() {
    // 清除节点之间的所有链接是“不必要的”，但是：
    // - 如果丢弃的节点驻留在一代以上，则帮助生成GC
    // - 即使存在可访问的迭代器，也确保释放内存
    for (Node<E> x = first; x != null; ) {
        Node<E> next = x.next;
        x.item = null;
        x.next = null;
        x.prev = null;
        x = next;
    }
    first = last = null;
    size = 0;
    modCount++;
}
```

### 8.位置访问方法

通过index获取元素

```java
/**
 * 返回此列表中指定位置的元素。
 *
 * @param index 需要返回的元素索引
 * @return the 返回此列表中指定位置的元素。
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

将某个位置的元素重新赋值:

```java
/**
 * 用指定的元素替换此列表中指定位置的元素。
 *
 * @param index 要替换的元素下标
 * @param element 要设置的元素
 * @return 先前位于指定位置的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```

将元素插入到指定index位置:

```java
/**
 * 在此列表中的指定位置插入指定元素。
 * 将当前位于该位置的元素（如果有）和任何后续元素向右移动（将一个元素添加到其索引中）。
 *
 * @param index 插入指定元素的索引
 * @param element 要插入的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
```

删除指定位置的元素:

```java
/**
 * 删除此列表中指定位置的元素。移动所有左边的后续元素（从其索引中减去一个）。
 * 返回从列表中删除的元素。
 *
 * @param index 要删除的元素的索引
 * @return 先前位于指定位置的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

其它位置的方法:

```java
/**
 * 告知是否存在该下标
 */
private boolean isElementIndex(int index) {
    return index >= 0 && index < size;
}

/**
 * 告诉参数是不是迭代器或加法操作的有效位置的索引
 */
private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;
}

/**
 * 构造IndexOutOfBoundsException详细消息。
 * 在许多可能的错误处理代码重构中，这种“概述”在服务器和客户端VM上都表现得最好。
 */
private String outOfBoundsMsg(int index) {
    return "Index: "+index+", Size: "+size;
}

/**
 * 检查元素下标
 */
private void checkElementIndex(int index) {
    if (!isElementIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

/**
 * 检查元素位置下标
 */
private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

### 9.查找操作

查找操作的本质是查找元素的下标:

查找某元素第一次出现的index, 如果找不到返回-1；

```java
/**
 * 返回指定元素在列表中第一次出现的索引
 * 如果此列表不包含元素，则为-1。
 * 更正式地说，返回最小索引 {@code i} 这样
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * 如果没有这样的索引，返回-1。
 *
 * @param o 要查找索引的元素
 * @return 此列表中指定元素第一次出现的索引，如果此列表不包含该元素，则为-1
 */
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```

查找元素最后一次出现的index, 如果找不到返回-1；

```java
/**
 * 返回指定元素在列表中最后一次出现的索引
 * 如果此列表不包含元素，则为-1。
 * 更正式地说，返回最小索引 {@code i} 这样
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * 如果没有这样的索引，返回-1。
 *
 * @param o 要查找索引的元素
 * @return 此列表中指定元素最后一次出现的索引，如果此列表不包含该元素，则为-1
 */
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}
```

### 10.Queue 方法

```java
/**
 * 检索但不删除此列表的头（第一个元素）。
 *
 * @return 这个列表的头，或者如果列表为空返回 {@code null} 
 * @since 1.5
 */
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}

/**
 * 检索但不删除此列表的头（第一个元素）。
 *
 * @return 这个列表的头
 * @throws NoSuchElementException 如果列表为空
 * @since 1.5
 */
public E element() {
    return getFirst();
}

/**
 * 检索并删除此列表的头（第一个元素）。
 *
 * @return 此列表的开头，或者｛@code null｝（如果此列表为空）
 * @since 1.5
 */
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

/**
 * Retrieves and removes the head (first element) of this list.
 *
 * @return the head of this list
 * @throws NoSuchElementException if this list is empty
 * @since 1.5
 */
public E remove() {
    return removeFirst();
}

/**
 * 将指定的元素添加到此列表的尾部（最后一个元素）。
 *
 * @param e 添加的元素
 * @return ｛@code true｝（由｛@link Queue#offer｝指定）
 * @since 1.5
 */
public boolean offer(E e) {
    return add(e);
}
```

### 11.Deque 方法

```java
/**
 * 将元素添加到列表第一位
 *
 * @param e 要添加的元素
 * @return {@code true} (as specified by {@link Deque#offerFirst})
 * @since 1.6
 */
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

/**
 * 将元素添加到列表最后一位
 *
 * @param e 要添加的元素
 * @return {@code true} (as specified by {@link Deque#offerLast})
 * @since 1.6
 */
public boolean offerLast(E e) {
    addLast(e);
    return true;
}

/**
 * 检索但不删除此列表的第一个元素，
 * 如果此列表为空，则返回｛@code null｝。
 *
 * @return 此列表的第一个元素，或｛@code null｝如果此列表为空
 * @since 1.6
 */
public E peekFirst() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
 }

/**
 * 检索但不删除此列表的最后一个元素，
 * 如果此列表为空，则返回｛@code null｝。
 *
 * @return 此列表的最后一个元素，或｛@code null｝如果此列表为空
 * @since 1.6
 */
public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}

/**
 * 检索并删除此列表的第一个元素，
 * 如果此列表为空，则返回｛@code null｝。
 *
 * @return 此列表的第一个元素，或｛@code null｝如果此列表为空
 * @since 1.6
 */
public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

/**
 * 检索并删除此列表的最后一个元素，
 * 如果此列表为空，则返回｛@code null｝。
 *
 * @return 此列表的最后一个元素，或｛@code null｝如果此列表为空
 * @since 1.6
 */
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}

/**
 * 将元素推送到此列表所表示的堆栈上。将元素插入此列表的前面。
 * <p>这方法等同于{@link #addFirst}.
 *
 * @param e 要推送的元素
 * @since 1.6
 */
public void push(E e) {
    addFirst(e);
}

/**
 * 从该列表表示的堆栈中弹出一个元素。换句话说，删除并返回此列表的第一个元素。
 *
 * <p>这方法等同于 {@link #removeFirst()}.
 *
 * @return 此列表前面的元素（它是此列表表示的堆栈的顶部）
 * @throws NoSuchElementException 如果集合为空
 * @since 1.6
 */
public E pop() {
    return removeFirst();
}

/**
 * 删除此列表中第一个出现的指定元素（从头到尾遍历列表时）。
 * 如果列表中不包含该元素，则它将保持不变。
 *
 * @param o 列表中要删除的元素(第一个)
 * @return {@code true} 如果包含该元素
 * @since 1.6
 */
public boolean removeFirstOccurrence(Object o) {
    return remove(o);
}

/**
 * 删除此列表中最后一个出现的指定元素（倒过来遍历）。
 * 如果列表中不包含该元素，则它将保持不变。
 *
 * @param o 列表中要删除的元素(最后一个)
 * @return {@code true} 如果包含该元素
 * @since 1.6
 */
public boolean removeLastOccurrence(Object o) {
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

## 参考

Java LinkedList源码剖析 结合源码对LinkedList进行讲解 http://www.cnblogs.com/CarpenterLee/p/5457150.html
