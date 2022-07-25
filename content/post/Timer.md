---
title: "JAVA - 定时器"
date: 2022-07-25T12:22:16+08:00
categories:
    - Java
tags:
    - 多线程
draft: false
---

&emsp;&emsp;我们有时会有定时执行任务的需求，比如需要在3秒后执行任务，这当然可以用Thread.sleep()并封装类来实现，但Java提供了一套自己的方法。
```java
    public static void main(String[] args) throws InterruptedException {
        Timer timer = new Timer();  //创建定时器
        timer.schedule(new TimerTask() {    //这是一个抽象类而非接口
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());   //打印当前线程的名字
            }
        },1000);    //延迟1s执行
    }
```
&emsp;&emsp;通过Timer类可以进行定时任务调度，包括定时任务，循环定时任务等等。\
&emsp;&emsp;但在执行上述代码的过程中，发现虽然任务执行完了，但程序并没有停止运行，这是因为Timer内存维护了一个任务队列和一个工作线程：
```java
    /**
     * The timer task queue.  This data structure is shared with the timer
     * thread.  The timer produces tasks, via its various schedule calls,
     * and the timer thread consumes, executing timer tasks as appropriate,
     * and removing them from the queue when they're obsolete.
     */
    private final TaskQueue queue = new TaskQueue();

    /**
     * The timer thread.
     */
    private final TimerThread thread = new TimerThread(queue);
```
&emsp;&emsp;其中TimerThread继承自Thread，而它的run()方法会循环读取队列，检测是否还有任务，若没有则会暂时处于休眠状态：
```java
public void run() {
        try {
            mainLoop();
        } finally {
            // Someone killed this Thread, behave as if Timer cancelled
            synchronized(queue) {
                newTasksMayBeScheduled = false;
                queue.clear();  // Eliminate obsolete references
            }
        }
    }
        /**
     * The main timer loop.  (See class comment.)
     */
    private void mainLoop() {
        while (true) {
            try {
                TimerTask task;
                boolean taskFired;
                synchronized(queue) {
                    // Wait for queue to become non-empty
                    while (queue.isEmpty() && newTasksMayBeScheduled)
                        queue.wait();
                    if (queue.isEmpty())
                        break; // Queue is empty and will forever remain; die
                    ...
                    ...
```
&emsp;&emsp;可以通过调用cancel()方法来关闭其工作线程：
```java
public void cancel() {
        synchronized(queue) {
            queue.clear();
            cleanup.clean();
        }
    }
```
```java
    public static void main(String[] args) throws InterruptedException {
        Timer timer = new Timer();  //创建定时器
        timer.schedule(new TimerTask() {    //这是一个抽象类而非接口
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());   //打印当前线程的名字
                timer.cancel();
            }
        },1000);    //延迟1s执行
    }
```