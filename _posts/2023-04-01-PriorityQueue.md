---
title: Java优先队列PriorityQueue
date: 2023-04-01 20:54:00 +0800
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

# Java优先队列PriorityQueue 

## 一、概述

​	Java中PriorityQueue 实现的是Queue 接口，可以使用Queue的方法和自定义方法；其通过完全二叉树构造的小顶堆实现（任意一个非叶子节点的权值，都不大于其左右子节点的权值）。对于可比较的元素（natural ordering）直接进行比较；对于自定义类的比较，通过构造时传入的比较。

> 优先队列的默认空间大小为11

​	**优先队列的作用是能保证每次取出的元素都是队列中权值最小的。**这里牵涉到了大小关系，元素大小的评判可以通过元素本身的自然顺序（natural ordering），也可以通过构造时传入的比较（Comparator，类似于C++的仿函数）



## 二、接口实现

PriorityQueue 实现的是 Queue 接口 ，可以使用 Queue 提供的方法，以及自带的方法

![image-20230404111955516](/assets/blog_res/2023-04-01-PriorityQueue.assets/image-20230404111955516.png)



## 三、常用方法

```java
public boolean add(E e); //在队尾插入元素，插入失败时抛出异常，并调整堆结构
public boolean offer(E e); //在队尾插入元素，插入失败时抛出false，并调整堆结构

public E remove(); //获取队头元素并删除，并返回，失败时前者抛出异常，再调整堆结构
public E poll(); //获取队头元素并删除，并返回，失败时前者抛出null，再调整堆结构

public E element(); //返回队头元素（不删除），失败时前者抛出异常
public E peek()；//返回队头元素（不删除），失败时前者抛出null

public boolean isEmpty(); //判断队列是否为空
public int size(); //获取队列中元素个数
public void clear(); //清空队列
public boolean contains(Object o); //判断队列中是否包含指定元素（从队头到队尾遍历）
public Iterator<E> iterator(); //迭代器
```



## 四、堆结构调整

每次插入或删除元素后，都对队列进行调整，使得队列始终构成最小/大堆。

具体调整如下：

- **插入**元素后，从**堆底到堆顶**调整堆；
- **删除**元素后，将队尾元素复制到队头，并从**堆顶到堆底**调整堆。

**小根堆结构调整**

插入（ add()和offer()方法 ）元素后，向上调整堆：

```java
//siftUp()
private void siftUp(int k, E x) {
    while (k > 0) {
        int parent = (k - 1) >>> 1;//parentNo = (nodeNo-1)/2
        Object e = queue[parent];
        if (comparator.compare(x, (E) e) >= 0)//调用比较器的比较方法
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = x;
}
```

删除（ remove()和poll()方法 ）元素后，向下调整堆：

```java
//siftDown()
private void siftDown(int k, E x) {
    int half = size >>> 1;
    while (k < half) {
        //首先找到左右孩子中较小的那个，记录到c里，并用child记录其下标
        int child = (k << 1) + 1;//leftNo = parentNo*2+1
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            comparator.compare((E) c, (E) queue[right]) > 0)
            c = queue[child = right];
        if (comparator.compare(x, (E) c) <= 0)
            break;
        queue[k] = c;//然后用c取代原来的值
        k = child;
    }
    queue[k] = x;
}
```



## 五、自定义类的比较

PriorityQueue采用：Comparble和Comparator两种方式：

### 1、实现Comparble接口

该方法是默认的内部比较方式，**如果用户插入自定义类型对象时，该类对象必须要实现Comparble接口**，**并重载compareTo方法**

```java
import java.util.Comparator;
import java.util.PriorityQueue;
public class CompTest {
    class Comp implements Comparable<Comp> {
        int val;
        ListNode ptr;
        Comp(int val, ListNode ptr) {
            this.val = val;
            this.ptr = ptr;
        }
        public int compareTo(Comp b) {
            return this.val - b.val;
        }
    }
    public static void main(String[] args) {
        PriorityQueue<Comp> queue = new PriorityQueue<Comp>();
    }
}
```



### 2、实现Comparator接口

实现Comparator比较器并重载compare方法：

```java
import java.util.Comparator;
import java.util.PriorityQueue;
public class CompTest {
    public static void main(String[] args) {
         PriorityQueue<ListNode> queue = new PriorityQueue<>(new Comparator<ListNode>() {
            @Override
            public int compare(ListNode o1, ListNode o2) {
                return o1.val-o2.val;
            }
        });
        }
    }
}
```



> **由于源码中新入队元素x是在第1个参数的位置，因此最大/最小优先队列主要根据第1个参数的大小关系来判断**
>
> 可以直接使用下面方法进行实现
>
> 最小堆：return o1.compareTo(o2); /return o1-02;  (队头为最小值,后面乱序)
>
> 最大堆：return o2.compareTo(o1); /return o2-o1; (队头为最大值,后面乱序)

## 六、应用场景

**前K个高频元素**

> 给定一个整数数组nums和一个整数k，请返回其中出现频率前k高的元素，可以接受任意顺序返回元素

```java
//本题采用数组代替hashmap
public class TopK {
    public int[] topKFrequent(int[] nums, int k) {
        //创建hashmap用于记录数组每个数出现的次数
        HashMap<Integer,Integer> map = new HashMap<Integer,Integer>();
        //遍历数组 记录次数
        for (int num : nums) {
            map.put(num,map.getOrDefault(num,0)+1); //对不存在的值提供默认值
        }

        //创建优先队列构建小顶堆  存储数组 int[] 的第一个元素代表数组的值，第二个元素代表了该值出现的次数
        //这里创建了一个匿名内部类作为比较器，实现了 Comparator 接口中的 compare 方法。
        // 在 compare 方法中，根据两个元素的第二个元素（索引为 1）的大小关系来进行比较，
        // 返回一个负数表示 m 的第二个元素小于 n 的第二个元素，返回 0 表示 m 的第二个元素
        // 等于 n 的第二个元素，返回一个正数表示 m 的第二个元素大于 n 的第二个元素。
        // 这样，优先队列中队首元素的第二个元素就是最小的。
        PriorityQueue<int[]> queue = new PriorityQueue<>(new Comparator<int[]>() {
            public int compare(int[] m, int[] n) {
                return m[1] - n[1];
            }
        });

        //对一个 Map<Integer, Integer> 对象中的每个键值对进行遍历操作
        for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
            //获取map的键和值 键代表数组内的数 值代表其出现的次数
            int num = entry.getKey(), count = entry.getValue();
            //如果队列内的元素等于k，那么就需要判断当前元素是否可以加入队列
            if (queue.size() == k){
                if (queue.peek()[1] < count){
                    queue.poll();
                    queue.offer(new int[]{num,count});
                }
            }
            //否则 直接加入队列
            else{
                queue.offer(new int[]{num,count});
            }
        }
        //创建结果数组
        int[] res = new int[k];
        for (int i = 0; i < k; i++) {
            res[i] = queue.poll()[0];
        }
        return res;
    }
}
```

