---
title: "JAVA - 线程池"
date: 2022-03-02T16:57:16+08:00
categories:
    - Java
tags:
    - 多线程
draft: false
---

## 阻塞队列

​		JUC包中存在各种各样的阻塞队列，用于不同的场景。

​		阻塞队列是基于队列与`ReentrantLock`实现的，适用于多线程环境下，它的`put`方法可以实现入队，并且在队列已满的情况下阻塞线程，直到能入队为止。

​		比如此时有一个容量为5的阻塞队列，线程1入队了5个元素，线程2还要入队3个元素，但此时队列已满，第二个线程会被直接阻塞。

​		此时有线程3使队列出队2个元素，那线程2停止阻塞，入队两个元素，带着最后一个元素继续阻塞。

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202303021713925.png)

​		阻塞队列有多种实现，其中常用的有五种：

* `ArrayBlockingQueue`：有界阻塞队列，队列有容量限制，需要在初始化时指定
* `SynchronousQueue`：只能有单个元素的阻塞队列
* `LinkedBlockingQueue`：有界阻塞队列，由链表构成，但实际默认容量是`Integer.MAX_VALUE`
* `PriorityBlockingQueue`：无界阻塞队列，支持优先级排序
* `DelayQueue`：无界阻塞队列，有延时出队属性，同样支持优先级排序

## 线程池

​		线程池就是管理一系列线程的资源池，线程池中的线程在使用后不会销毁，而是将线程复用，利用池化技术，反复利用这些线程。

​		线程池的好处：

* 降低资源消耗
* 提高响应速度
* 提高线程的可管理性

## 线程池的创建

### ThreadPoolExecutor

#### 参数

​		可以通过ThreadPoolExecutor直接新建一个线程池对象，它已经实现好了线程的调度机制。

​		它共有四个构造方法，直接来看全参构造。

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
        null :
    AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

​		参数的含义如下：

* `corePoolSize`：**核心线程池大小**，每向线程池提交一个多线程任务时，都会创建一个新的核心线程，无论是否存在其他空闲线程，直到到达核心线程池大小为止，之后会尝试复用线程资源。当然也可以在一开始就全部初始化好，调用` prestartAllCoreThreads()`即可。
* `maximumPoolSize`：**最大线程池大小**，当目前线程池中所有的线程都处于运行状态，并且等待队列已满，那么就会直接尝试继续创建新的非核心线程运行，但是不能超过最大线程池大小。
* `keepAliveTime`：**线程最大空闲时间**，当一个非核心线程空闲超过一定时间，会自动销毁。
* `unit`：**线程最大空闲时间的时间单位**
* `workQueue`：**线程任务队列**，当线程池中核心线程数已满时，就会将任务暂时存到任务队列中，直到有线程资源可用为止，实现就是上面的阻塞队列。
* `threadFactory`：**线程创建工厂**，可以干涉线程池中线程的创建过程，进行自定义。
* `handler`：**拒绝策略**，当任务队列和线程池都没有空间，不能再来新的任务时，来了个新的多线程任务，就会根据当前设定的拒绝策略进行处理。

#### 创建实例

​		这里给出一个创建实例：

```java
public static void main(String[] args) throws InterruptedException {
    ThreadPoolExecutor executor =
        new ThreadPoolExecutor(2, 4,   //2个核心线程，最大线程数为4个
                               3, TimeUnit.SECONDS,        //最大空闲时间为3秒钟
                               new ArrayBlockingQueue<>(2));     //使用容量为2的ArrayBlockingQueue队列

    for (int i = 0; i < 6; i++) {   //开始6个任务
        int finalI = i;
        executor.execute(() -> {
            try {
                System.out.println(Thread.currentThread().getName()+" 开始执行！（"+ finalI);
                TimeUnit.SECONDS.sleep(1);
                System.out.println(Thread.currentThread().getName()+" 已结束！（"+finalI);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }

    TimeUnit.SECONDS.sleep(1);    //看看当前线程池中的线程数量
    System.out.println("线程池中线程数量："+executor.getPoolSize());
    TimeUnit.SECONDS.sleep(5);     //等到超过空闲时间
    System.out.println("线程池中线程数量："+executor.getPoolSize());

    executor.shutdownNow();    //立即关闭线程池
}
```

### Executors

​		通过使用`Executor`框架的工具类`Executors`来创建线程池。

​		但在《阿里巴巴Java开发手册》中强制不允许使用`Executors`创建线程池，因为有概率造成OOM，这里并不补充详细内容。

## 线程池原理

​		根据上面的参数含义和实例基本可以推出线程池的工作原理，这里做一些源码与流程的补充。

### ctl

​		ctl变量在`execute`方法中有使用，这里先介绍一下。

```java
// 原子类AtomicInteger，用于同时保存线程池运行状态和线程数量
// 通过拆分32个bit位来保存数据，前3位保存线程池状态，后29位保存工作线程数量
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;    // 后29位
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;   //计算得出最大容量（1左移29位，最大容量为2的29次方-1）

// 线程池状态，都是只占用前3位，不会占用后29位
// 接收新任务，并等待执行队列中的任务
private static final int RUNNING    = -1 << COUNT_BITS;   //111 | 0000... (后29数量位，下同)
// 不接收新任务，但是依然等待执行队列中的任务
private static final int SHUTDOWN   =  0 << COUNT_BITS;   //000 | 数量位
// 不接收新任务，也不执行队列中的任务，并且还要中断正在执行中的任务
private static final int STOP       =  1 << COUNT_BITS;   //001 | 数量位
// 所有的任务都已结束，线程数量为0，即将完全关闭
private static final int TIDYING    =  2 << COUNT_BITS;   //010 | 数量位
// 完全关闭
private static final int TERMINATED =  3 << COUNT_BITS;   //011 | 数量位
```

### execute

​		`ThreadPoolExecutor`类的`execute`方法，里面描述了线程池的工作流程。

```java
public void execute(Runnable command) {
    // 如果传入的任务为空，直接抛出空指针异常
    if (command == null)
        throw new NullPointerException();
    // 获取ctl的值
    int c = ctl.get();
    // 如果工作线程数 < 核心线程数
    if (workerCountOf(c) < corePoolSize) {
        // 直接添加线程
        if (addWorker(command, true))
            return;
        // 如果添加线程失败了就要更新ctl的值
        c = ctl.get();
    }
    // 如果线程池是运行状态且阻塞队列未满则往阻塞队列中添加任务
    if (isRunning(c) && workQueue.offer(command)) {
        // 再次获取ctl的值
        int recheck = ctl.get();
        // 再次确认线程池是否还在运行状态，如果关闭了就要把刚刚加进阻塞队列的任务再拿出来
        if (! isRunning(recheck) && remove(command))
            // 根据拒绝策略拒绝当前任务
            reject(command);
        // 如果确实还在运行那就检查工作线程是否为0
        else if (workerCountOf(recheck) == 0)
            // 添加新线程执行
            addWorker(null, false);
    }
    // 如果不在运行状态则拒绝，这种情况要么是线程池关了或阻塞队列满了且没法再添加非核心线程
    else if (!addWorker(command, false))
        reject(command);
}
```

### 流程

​		根据`execute`方法的源码可以总结出线程池的工作流程，如下图所示。

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202303041512929.png)





