---
title: Java线程池详解
date: 2023-04-22 19:59:00 +0800
categories: [技术科普]
tags: [学习]
pin: true
author: YKFire

toc: true
comments: true

math: false
mermaid: true

typora-root-url: ../../YKFire.github.io

---

# Java线程池详解

## 一、作用

​	线程池（ThreadPool）是⼀种基于池化思想管理和使用线程的机制。它是将多个线程预先存储在⼀个“池子”内，当有任务出现时可以避免重新创建和销毁线程所带来性能开销，只需要从“池子”内取出相应的线程执行对应的任务即可。

线程我们可以使用 new 的方式去创建，但如果并发的线程很多，每个线程执行的时间又不长，这样频繁的创建线程会大大的降低系统处理的效率，因为创建和销毁进程都需要消耗资源，线程池就是用来解决类似问题。

线程池实现了一个线程在执行完一段任务后，不销毁，继续执行下一段任务。用《Java并发编程艺术》提到线程池的优点：

- 降低资源的消耗：使得线程可以重复使用，不需要在创建线程和销毁线程上浪费资源
- 提高响应速度：任务到达时，线程可以不需要创建即可以执行
- 线程的可管理性：线程是稀缺资源，如果无限制的创建会严重影响系统效率，线程池可以对线程进行管理、监控、调优。



>Java线程池的核心作用：通过复用线程，避免了线程创建和销毁的开销，提高了程序的性能和可靠性

##  二、组成部分

- 线程池管理器：用于创建并管理线程池
- 工作线程：线程池中的线程
- 任务接口：每个任务都必须实现的接口，用于工作线程调度其运行
- 任务队列：用于存放待处理的任务，提供一种缓冲机制



## 三、实现原理及使用

​	java线程池是通过Executor框架实现的，该框架中用到了 Executor，ExecutorsExecutorService，ThreadPoolExecutor ，Callable 和 Future、 FutureTask 这几个类

<img src="/assets/blog_res/2023-04-22-threadpool.assets/image-20230422221603113.png" alt="image-20230422221603113" style="zoom:150%;" />

> 主要是通过ThreadPoolExecutor类来实现 下面详细介绍Java线程池的使用方法与注意事项

### 1.创建线程池

创建线程池时，需要指定线程池的**核心线程数、最大线程数、线程空闲时间、任务队列**、**拒绝策略**等参数。例如：

```
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    corePoolSize, // 核心线程数
    maximumPoolSize, // 最大线程数
    keepAliveTime, // 线程空闲时间
    TimeUnit.SECONDS, // 时间单位
    new LinkedBlockingQueue<Runnable>() // 任务队列
    handler //拒绝策略
);
```

### 2.提交任务

将任务提交给线程池时，可以调用execute()方法或submit()方法。例如：

```
executor.execute(new Runnable() {
    @Override
    public void run() {
        // 执行任务
    }
});

Future<?> future = executor.submit(new Callable<Object>() {
    @Override
    public Object call() throws Exception {
        // 执行任务，返回结果
        return null;
    }
});
```

### 3.线程池状态

线程池会检查自身的状态，如果状态不是RUNNING，那么线程池会拒绝任务，或者直接抛出异常。线程池的状态有以下几种：

- RUNNING：线程池正在运行，可以接收任务。
- SHUTDOWN：线程池正在关闭，不再接收新任务，但会执行已提交的任务。
- STOP：线程池正在关闭，不再接收新任务，也不会执行已提交的任务。
- TIDYING：线程池正在清理任务队列。
- TERMINATED：线程池已经关闭，所有任务已经执行完毕。

### 4.任务队列

如果线程池中的线程都在执行任务，新的任务就会被放入任务队列中，等待线程空闲后再执行。常用的任务队列有以下几种：

- ArrayBlockingQueue：基于数组的有界队列，按照FIFO原则对任务进行排序。
- LinkedBlockingQueue：基于链表的无界队列，按照FIFO原则对任务进行排序。
- SynchronousQueue：不存储任务的阻塞队列，将任务直接交给线程执行。

### 5.线程池管理线程

线程池会根据任务数量和线程池状态来管理线程，如果任务数量超过了线程池的最大线程数，新的任务就会被拒绝或者等待。线程池管理线程的方法有以下几种：

- prestartCoreThread()：预启动一个核心线程。
- prestartAllCoreThreads()：预启动所有核心线程。
- allowCoreThreadTimeOut(boolean)：设置核心线程是否允许超时回收。
- setMaximumPoolSize(int)：设置线程池的最大线程数。
- setKeepAliveTime(long, TimeUnit)：设置线程空闲时间。
- setRejectedExecutionHandler(RejectedExecutionHandler)：设置任务拒绝策略。

### 6.关闭线程池

当不再需要线程池时，可以调用shutdown()方法关闭线程池，线程池会拒绝新任务，并等待已提交的任务执行完成。如果希望立即关闭线程池，可以调用shutdownNow()方法。例如：

```
executor.shutdown();
```

### 7.清理线程池

当所有任务执行完成后，线程池会清理所有线程，并释放资源。如果希望等待所有任务执行完成后再关闭线程池，可以调用awaitTermination()方法。例如：

```
executor.awaitTermination(Long.MAX_VALUE, TimeUnit.NANOSECONDS);
```

​	总的来说，Java线程池通过复用线程，避免了线程创建和销毁的开销，提高了程序的性能和可靠性。但是，在使用Java线程池时，需要注意避免线程安全问题，并合理设置线程池的参数，以达到最优的性能和可靠性。





## 四、工作流程

> Java线程池是一种并发编程技术，它允许开发者将多个任务分配给一组线程，从而提高程序的性能和可靠性。

1.创建线程池后，开始等待请求
2.当调用execute()方法添加一个请求任务时，线程池会做以下判断：

- 如果正在运行的线程数量小于corePoolSize，马上创建线程执行任务
- 如果正在运行的线程数量大于等于corePoolSize，将该任务放入等待队列
- 如果等待队列已满，但正在运行线程数量小于max，创建非核心线程执行任务
- 如果队列满了且正在运行的线程数量大于max，线程池会启动饱和拒绝策略

3.当一个线程完成任务时，会从等待队列中取下一个任务来执行

4.当空闲线程超过keepAliveTime定义时间，会判断：

- 如果当前运行线程大于corePoolSize，该线程销毁
- 所有线程执行完任务后，线程个数恢复到corePoolSize大小

图解：

![image-20230422223529048](/assets/blog_res/2023-04-22-threadpool.assets/image-20230422223529048.png)

![image-20230422222648952](/assets/blog_res/2023-04-22-threadpool.assets/image-20230422222648952.png)



## 五、拒绝策略

​	线程池中的线程已经用完了，无法继续为新任务服务，同时，等待队列也已经排满了，再也塞不下新任务了。这时候我们就需要**拒绝策略机制**合理的处理这个问题。

JDK 内置的拒绝策略如下:

- AbortPolicy : 直接抛出异常，阻止系统正常运行
- CallerRunsPolicy: 只要线程池未关闭，该策略直接在调用者线程中，运行当前被丢弃的任务。显然这样做不会真的丢弃任务，但是，任务提交线程的性能极有可能会急剧下降
- DiscardOldestPolicy : 丢弃最老的一个请求，也就是即将被执行的一个任务，并尝试再次提交当前任务。
- DiscardPolicy :该策略默默地丢弃无法处理的任务，不予任何处理。如果允许任务丢失，这是最好的一种方案


​	若以上策略仍无法满足实际以上内置拒策略均实现了 ReiectedExecutionHandler 接需要，完全可以自己扩展 ReiectedExecutionHandler 接口





