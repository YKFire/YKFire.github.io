---
title: ArrayList 可以完全替代数组吗？
date: 2023-01-21 15:50:00 +0800
categories: [数据结构]
tags: [学习]
pin: true
author: YKFire

toc: true
comments: true

math: false
mermaid: true

typora-root-url: ../../YKFire.github.io

---

# ArrayList 可以完全替代数组吗？

## 一、ArrayList 和 LinkedList 的区别

- **1、数据结构：** 在数据结构上，ArrayList 和 LinkedList 都是 “线性表”，都继承于 Java 的 `List` 接口。另外 LinkedList 还实现了 Java 的 `Deque` 接口，是基于链表的栈或队列，与之对应的是 `ArrayDeque` 基于数组的栈或队列；
- **2、线程安全：** ArrayList 和 LinkedList 都不考虑线程同步，不保证线程安全；
- **3、底层实现：** 在底层实现上，ArrayList 是基于动态数组的，而 LinkedList 是基于双向链表的。事实上，它们很多特性的区别都是因为底层实现不同引起的。比如说：
  - **在遍历速度上：** 数组是一块连续内存空间，基于局部性原理能够更好地命中 CPU 缓存行，而链表是离散的内存空间对缓存行不友好；
  - **在访问速度上：** 数组是一块连续内存空间，支持 O(1) 时间复杂度随机访问，而链表需要 O(n) 时间复杂度查找元素；
  - **在添加和删除操作上：** 如果是在数组的末尾操作只需要 O(1) 时间复杂度，但在数组中间操作需要搬运元素，所以需要 O(n)时间复杂度，而链表的删除操作本身只是修改引用指向，只需要 O(1) 时间复杂度（如果考虑查询被删除节点的时间，复杂度分析上依然是 O(n)，在工程分析上还是比数组快）；
  - **额外内存消耗上：** ArrayList 在数组的尾部增加了闲置位置，而 LinkedList 在节点上增加了前驱和后继指针。

## 二、ArrayList 源码分析

### 1、ArrayList 的属性

ArrayList 的属性很好理解，底层是一个 Object 数组，我要举手提问：

- **🙋🏻‍♀️疑问 1：** 为什么 elementData 字段不声明 `private` 关键字？
- **🙋🏻‍♀️疑问 2：** 为什么 elementData 字段声明 `transient` 关键字？
- **🙋🏻‍♀️疑问 3：** 为什么elementData 字段不声明为泛型类型 `E`？
- **🙋🏻‍♀️疑问 4：** 为什么 ArrayList 的最大容量是 `Integer.MAX_VALUE`，`Long.MAX_VALUE` 不行吗？
- **🙋🏻‍♀️疑问 5：** 为什么 ArrayList 的最大容量是 `MAX_VALUE - 8`，一定会减 `8` 吗？

这些问题我们在分析源码的过程中回答。疑问这么多，ArrayList 瞬间不香了。

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    // new ArrayList() 默认初始容量
    private static final int DEFAULT_CAPACITY = 10;

    // new ArrayList(0) 的全局空数组
    private static final Object[] EMPTY_ELEMENTDATA = {};

    // new ArrayList() 的全局空数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    // 修改次数记录
    protected transient int modCount = 0;

    // 数组的最大长度
    // 疑问 4：为什么 ArrayList 的最大容量是 Integer.MAX_VALUE，Long.MAX_VALUE 不行吗？
    // 疑问 5：为什么 ArrayList 的最大容量是 MAX_VALUE - 8，一定会减 8 吗？
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8

    // 疑问 1：为什么不声明 private（后文回答）
    // 疑问 2：为什么声明 transient（后文回答）
    // 疑问 3：为什么不声明为泛型类型 E
    // 底层数组
    transient Object[] elementData;

    // 数组的有效长度（不是 elementData.length）
    private int size;

    // size() 返回的是数组的有效长度（合理，底层数组我们不关心）
    public int size() {
        return size;
    }
}
```

### 2、ArrayList 的构造方法

ArrayList 有三个构造函数：

- **1、带初始容量的构造方法：** 如果初始容量大于 0，则创建指定长度的数组。如果初始容量是 0，则指向第 1 个全局空数组 ；`EMPTY_ELEMENTDATA`；
- **2、无参构造方法：** 指向第 2 个全局空数组 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA`；
- **3、带集合的构造方法：** 将集合转为数组，如果数组为空，则指向第 1 个全局空数组 `EMPTY_ELEMENTDATA`；

**可以看到，除了指定大于 0 的初始容量外，ArrayList 在构造时不会创建数组，而是指向全局空数组，这是懒初始化的策略。**

构造器的源码不难，但小朋友总有太多的问号，举手提问 🙋🏻‍♀️：

- **🙋🏻‍♀️疑问 6：既然都是容量为 0 ，为什么 ArrayList 要区分出 2 个空数组？**

这个问题直接回答吧：ArrayList 认为无参构造函数应该使用默认行为，在首次添加数据时会创建长度为 `10（DEFAULT_CAPACITY）` 的默认初始数组；而显示设置初始容量为 0 是开发者的显式意图，所以不使用默认初始数组，在首次添加数据时只会创建长度为 `1 （size + 1）`的数组（可以结合后文源码理解下）。

- **🙋🏻‍♀️疑问 7：** 在带集合的构造方法中，为什么会存在集合转化后的数组类型不是 Object[].class 的情况？

```java
// 带初始容量的构造方法
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        // 创建 initialCapacity 长度的数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        // 指向第 1 个全局空数组
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        // 不合法的初始容量
        throw new IllegalArgumentException("Illegal Capacity: "+initialCapacity);
    }
}

// 无参构造方法
public ArrayList() {
    // 指向第 1 个全局空数组
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 带集合的构造方法
public ArrayList(Collection<? extends E> c) {
    // 将集合转为数组
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // 疑问 7：这一个条件语句好奇怪，toArray() 的返回值类型就是 Object[] 啊？
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        this.elementData = EMPTY_ELEMENTDATA;
    }
}

public Object[] toArray() {
    return Arrays.copyOf(elementData, size);
}
```



### 3、 ArrayList 的添加与扩容方法

ArrayList 可以在数组末尾或数组中间添加元素：

- 如果是在数组末尾添加，均摊时间只需要 O(1) 时间复杂度；
- 如果在数组中间添加，由于需要搬运数据，所以需要 O(n) 时间复杂度。

添加前会先检查数据容量，不足会先扩容：

- 在使用无参构造器初始化时，首次添加元素时会直接扩容到 10 的容量；
- 在其他情况，会直接扩容到旧容量的 1.5 倍，而不是扩容到最小容量要求。

**不管是扩容到 10 还是扩容到 1.5 倍，都是为了防止频繁扩容，避免每次 add 添加数据都要扩容一次。**

- **🙋🏻‍♀️疑问 4：为什么 ArrayList 的最大容量是 `Integer.MAX_VALUE`，`Long.MAX_VALUE` 不行吗？**

数组对象的长度是记录在对象头中的 “数组长度” 字段中，这个字段是 4 字节，正好就是 Integer 也是 4 个字节，所以限制为 `Integer.MAX_VALUE`，而不能使用 `Long.MAX_VALUE`。

不对啊，Java Integer 是有符号整数，所以 `Integer.MAX_VALUE` 只有 31 位有效位，还少了 1 位呢。没错，是少了一位。如果要榨干这 1 位容量，当然可以用 long 类型并且限制到 32 位能够表示的最大正整数上，并且在源码中到处加上数组越界判断，想想就不稳定的。相比之下，限制数组长度为 int 类型且最大长度为 `Integer.MAX_VALUE`，如果有超过 `Integer.MAX_VALUE` 存储容量的需求，那就创建两个数组呀：）你觉得哪种更好。

Java 对象内存布局:

![image-20230206164455908](/assets/blog_res/2023-01-21-ArrayList.assets/image-20230206164455908.png)

- **🙋🏻‍♀️疑问 5：为什么 ArrayList 的最大容量是 `MAX_VALUE - 8`，一定会减 `8` 吗？**

依然与对象的内存布局有关。在 Java 虚拟机垃圾回收算法中，需要计算对象的内存大小，计算结果是存储在 `jint` 类型变量（Java int 类型在 JNI 中的映射）中的。如果数组的长度是 `MAX_VALUE`，那么加上对象头之后就整型溢出了，所以 ArrayList 会预先减掉对象头可能占用的 8 个字节。对象头的具体大小取决于虚拟机实现，减 8 是相对保守的。

其实，ArrayList 的最大容量也不一定会减 8，如果最小容量要求是超过 `MAX_ARRAY_SIZE` 的，那么还是会扩容到 `MAX_VALUE` 。这有点摆烂的意思，会不会溢出运行时再说。

数组长度溢出:

```java
OutOfMemoryError: Requested array size exceeds VM limit
```

- **🙋🏻‍♀️疑问 8：不应该是 elementData.length - minCapacity > 0 吗？** 这是考虑到整型溢出的情况，minCapacity 溢出就变成负数了

```java
// 在数组末尾添加元素
public boolean add(E e) {
    // 先确保底层数组容量足够容纳 size + 1，不足会扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 在 size + 1 的位置赋值
    elementData[size++] = e;
    return true;
}

// 在数组中间插入元素
public void add(int index, E element) {
    // 范围检查
    rangeCheckForAdd(index);
    // 先确保容量足够容纳 size + 1，不足会扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 先搬运数据腾出空位
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    // 在 index 的位置赋值
    elementData[index] = element;
    // 长度加一
    size++;
}

// 在数组末尾添加集合
public boolean addAll(Collection<? extends E> c) {
    // 集合转数组
    Object[] a = c.toArray();
    // 先确保底层数组容量足够容纳 size + numNew，不足会扩容
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    // 搬运原数据
    System.arraycopy(a, 0, elementData, size, numNew);
    // 长度加 numNew
    size += numNew;
    return numNew != 0;
}

// 在数组中间插入集合
public boolean addAll(int index, Collection<? extends E> c) {
    // 略，原理类似
}

// 尝试扩容
// （提示：源码调用了 calculateCapacity() 函数，这里做内联简化）
private void ensureCapacityInternal(int minCapacity) {
    // 使用无参构造器初始化时，指定扩容为 10 
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity =  Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // 疑问 8：不应该是 elementData.length - minCapacity > 0 吗？
    // 如果底层数组长度不够 minCapacity 最小容量要求，则需要扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // 旧容量
    int oldCapacity = elementData.length;
    // 新容量 = 旧容量 * 1.5 倍（有可能整型溢出）
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果新容量小于最小容量要求，则使用最小容量（addAll 大集合的情况）
    if (newCapacity - minCapacity < 0) {
        newCapacity = minCapacity;
    }
    // （提示：上一个 if 的 newCapacity 有可能是溢出的）
    // 如果新容量超出最大数组长度限制，说明无法扩容 1.5 倍，回归到 minCapacity 上
    // （提示：源码调用了 hugeCapacity() 函数，这里做内联简化）
    if (newCapacity - MAX_ARRAY_SIZE > 0) {
        // 最小容量要求发生整型溢出，无法满足要求，只能直接抛出 OOM
        if (minCapacity < 0) throw new OutOfMemoryError();
        // 如果最小容量要求超出最大数组长度限制，则扩容到 MAX_VALUE（说明不一定会减 8）
        // 否则，扩容到最大数组长度限制（MAX_VALUE - 8）
        newCapacity = (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
    }
    // 扩容到 newCapacity 长度
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    // 已经内联简化到 grow 方法中
}
```

除了扩容之外，ArrayList 还支持缩容，将底层数组的容量缩小到实际元素的数量：

```java
// 缩容
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0) ? EMPTY_ELEMENTDATA : Arrays.copyOf(elementData, size);
    }
}
```

另外，因为扩容涉及到数据搬运操作，所以如果能事先知道数据的容量，最好在创建 ArrayList 时就显式指定数据量大小。

### 4、 ArrayList 的迭代器

Java 的 foreach 是语法糖，本质上也是采用 iterator 的方式。ArrayList 提供了 2 个迭代器：

- **iterator():Iterator()：** 单向迭代器
- **ListIterator listIterator()：** 双向迭代器

在迭代器遍历数组的过程中，有可能出现多个线程并发修改数组的情况，造成数据不一致甚至数组越界，所以 Java 很多容器类的迭代器中都有 fail-fast 机制。

如果在迭代的过程中发现 expectedModCount 变化，说明数据被修改，此时就会提前抛出 `ConcurrentModificationException` 异常（当然也不一定是被其他线程修改）。

```java
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    // 创建迭代器是会记录外部类的 modCount
    int expectedModCount = modCount;
    ...

    Itr() {}

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        // 检查
        checkForComodification();
        ...
    }

    public void remove() {
        ...
        // 更新
        expectedModCount = modCount;
    }
}
```

- **🙋🏻‍♀️疑问 1：为什么 elementData 字段不声明 `private` 关键字？**

在注释中的解释是：“non-private to simplify nested class access”。但我们知道在 Java 中，内部类是可以访问外部类的 private 变量的，所以这就说不通的。我的理解是：因为内部类在编译后会生成独立的 Class 文件，如果外部类的 elementData 字段是 private 类型，那么编译器就需要在 ArrayList 中插入 getter / setter，并通过方法调用，而 non-private 字段就可以直接访问字段。

### 5、ArrayList 的序列化过程

- **🙋🏻‍♀️疑问 2：为什么 elementData 字段声明 `transient` 关键字？**

ArrayList 重写了 JDK 序列化的逻辑，只把 elementData 数组中有效元素的部分序列化，而不会序列化整个数组。

```java
// 序列化和反序列化只考虑有效元素
private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // 写入数组长度
    s.writeInt(size);

    // 写入有效元素
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```



### 6、 ArrayList 的 clone() 过程

ArrayList 中的 elementData 数组是引用类型，因此在 clone() 中需要实现深拷贝，否则原对象与克隆对象会相互影响：

```java
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();
        // 拷贝数组对象
        v.elementData = Arrays.copyOf(elementData, size);
        // 修改计数归零
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```



### 7、为什么阿里巴巴要求谨慎使用 subList API？

在 《阿里巴巴 Java 开发手册》中，有关于 `ArrayList#subList` API 的规定。为什么阿里巴巴要做这样的限制呢？

- 【强制】ArrayList 的 subList 结果不可强转成 ArrayList，否则会抛出 ClassCastException 异常；
- 【强制】在 subList 场景中，高度注意对原集合元素的增加或删除，均会导致子列表的遍历、增加、删除产生 ConcurrentModificationException 异常。

这是因为 subList API 只是提供通过起始索引 fromIndex 和终止索引 toIndex 包装了一个原 ArrayList 的 **“视图窗口”** ，并不是真的截取并创建了一个新的 ArrayList，所以强制类型转换就会抛出 ClassCastException 异常。

此时，在 ArrayList 或 SubList 上做修改，要注意相互之间的影响：

- 在 ArrayList 或 SubList 上修改元素，都会同步更新到对方（因为底层都是 ArrayList 本身）；
- 在 SubList 上增加或删除元素，会影响到 ArrayList；
- 在 ArrayList 上增加或删除元素，会导致 SubList 抛出 ConcurrentModificationException 异常。

ArrayList.java

```java
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}

private class SubList extends AbstractList<E> implements RandomAccess {
    // 原 ArrayList
    private final AbstractList<E> parent;
    private final int parentOffset;
    private final int offset;
    int size;

    SubList(AbstractList<E> parent, int offset, int fromIndex, int toIndex) {
        this.parent = parent;
        this.parentOffset = fromIndex;
        this.offset = offset + fromIndex;
        this.size = toIndex - fromIndex;
        // modCount 记录
        this.modCount = ArrayList.this.modCount;
    }

    public E set(int index, E e) {
        rangeCheck(index);
        // 在 ArrayList 上增加或删除元素，会导致 SubList 抛出 ConcurrentModificationException 异常
        checkForComodification();
        // 在 SubList 上增加或删除元素，会影响到 ArrayList；
        E oldValue = ArrayList.this.elementData(offset + index);
        ArrayList.this.elementData[offset + index] = e;
        return oldValue;
    }
}
```

### 8、ArrayList 如何实现线程安全？

有 4 种方式：

- **方法 1 - 使用 Vector 容器：** Vector 是线程安全版本的数组容器，它会在所有方法上增加 synchronized 关键字；
- **方法 2 - 使用 Collections.synchronizedList 包装类：** 原理也是在所有方法上增加 synchronized 关键字；
- **方法 3 - 使用 CopyOnWriteArrayList 容器：** 基于加锁的 “读写分离” 和 “写时复制” 实现的动态数组，适合于读多写少，数据量不大的场景。
- **方法 4 - 使用 ArrayBlockingQueue 容器：** 基于加锁的阻塞队列，适合于带阻塞操作的生产者消费者模型。

## 三、Arrays#ArrayList

事实上，在 Java 环境中有两个 ArrayList，这或许是一个隐藏的彩蛋（坑）：

- **ArrayList：** 一般认为的 ArrayList，是一个顶级类；
- **Arrays#ArrayList：** Arrays 的静态内部类，和上面这个 ArrayList 没有任何关系。

其实，Arrays#ArrayList 的定位就是在数组和和 List 直接切换而已。Arrays 提供了数组转 List 的 API，而 Arrays#ArrayList 也提供了 List 转数组的 API（这些 API 第一个 ArrayList 中也都有…）

回过头看剩下的 2 个问题：

- **🙋🏻‍♀️疑问 3：为什么 elementData 字段不声明为泛型类型 `E`？**

泛型擦除后等于 Object[] elementData，没有区别。

- **🙋🏻‍♀️疑问 7：在带集合的构造方法中，为什么会存在集合转化后的数组类型不是 Object[].class 的情况？**

这是因为有些 List 集合的底层数组不是 Object[] 类型，有可能是 String[] 类型。而在 ArrayList#toArray() 方法中，返回值的类型是 Object[] 类型，有类型错误风险。

例如：在这段代码中，ArrayList 接收一个由 String 数组转化的 List，最后在 ArrayList#toArray() 返回的 Object 数组中添加一个 Object 对象，就出现异常了：

```java
// 假设没有特殊处理
List<String> list = new ArrayList<>(Arrays.asList("list"));
// class java.util.ArrayList
System.out.println(list.getClass());

Object[] listArray = list.toArray();
// 如果过没有特殊处理，实际类型是 [Ljava.lang
System.out.println(listArray.getClass());
// 如果过没有特殊处理，将抛出 ArrayStoreException 异常
listArray[0] = new Object();
```

源码摘要:

Arrays.java

```java
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}

private static class ArrayList<E> extends AbstractList<E> implements RandomAccess, java.io.Serializable {
    // 泛型擦除后：Object[] a;
    private final E[] a;

    // 泛型擦除后：Object[] array;
    // Java 数组是协变的，能够接收 String[] 类型的数组
    ArrayList(E[] array) {
        // 赋值
        a = Objects.requireNonNull(array);
    }

    // 实际返回的数组可能是 Object[] 类型，也可能是 String[] 类型
    @Override
    public Object[] toArray() {
        return a.clone();
    }
}
```

ArrayList.java

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable
    transient Object[] elementData;

    // 带集合的构造方法
    public ArrayList(Collection<? extends E> c) {
        // 将集合转为数组
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // 疑问 7：这一个条件语句好奇怪，toArray() 的返回值类型就是 Object[] 啊？
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }
}
```









## 四、ArrayList 这么好用，可以完全替代数组吗

大多数场景可以，但不能完全替代。

ArrayList 是基于 Object 数组封装的动态数组，我们不需要关心底层数组的数据搬运和扩容等逻辑，因此在大多数业务开发场景中，除非是为了最求极致的性能，否则直接使用 ArrayList 代替数组是更好的选择。

那么，ArrayList 有哪些地方上比数组差呢？

- 举例 1 - ArrayList 等容器类不支持 int 等基本数据类型，所以必然存在装箱拆箱操作；
- 举例 2 - ArrayList 默认的扩容逻辑是会扩大到原容量的 1.5 倍，在大数据量的场景中，这样的扩容逻辑是否多余，需要打上问题；
- 举例 3 - ArrayList 的灵活性不够。ArrayList 不允许底层数据有空洞，所有的有效数据都会 “压缩” 到底层数组的首部。因此，当需要基于数组二次开发容器时，ArrayList 并不是一个好选择。
  - 例如，使用 ArrayList 开发栈的结构或许合适，可以在数组的尾部操作数据。但使用 ArrayList 开发队列就不合适，因为在数组的首部入队或出队需要搬运数据；
  - 而数组没有这些约束，我们可以将数组设计为 “环形数组”，就可以避免入队和出队时搬运数据。例如 Java 的 `ArrayBlockingQueue`  和 [ArrayDeque](https://juejin.cn/post/7162819765361672199) 就是基于数组的队列。



## 五、总结

1、ArrayList 是基于数组封装的动态数组，封装了操作数组时的搬运和扩容等逻辑；

2、在构造 ArrayList 时，除了指定大于 0 的初始容量外，ArrayList 在构造时不会创建数组，而是指向全局空数组，这是懒初始化的策略；

3、在添加数据时会先检查数据容量，不足会先扩容。首次添加默认会扩容到 10 容量，后续会扩容到旧容量的 1.5 倍，这是为了避免反复扩容；

4、因为扩容涉及到数据搬运操作，所以如果能事先知道数据的容量，最好在创建 ArrayList 时就显式指定数据量大小；

5、ArrayList 重写了序列化过程，只处理数组中有效的元素；

6、ArrayList 的 subList API 只是提供视图窗口，并不是创建新列表；

7、ArrayList 在大多数场景中可以代替数组，但在高性能和二次封装的场景中，ArrayList 无法替代数组。

