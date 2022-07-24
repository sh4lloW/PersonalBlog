---
title: "JAVA - 线程锁"
date: 2022-07-24T16:41:16+08:00
categories:
    - Java
tags:
    - 多线程
    - 锁
draft: false
---

## 一个例子
&emsp;&emsp;先给出一个样例：
```java
    private static int value = 0;
    public static void main(String[] args) throws InterruptedException{
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) 
            {
                value++;
            }
            System.out.println("thread 1 end!");
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) 
            {
                value++;
            }
            System.out.println("thread 2 end!");
        });
        t1.start();
        t2.start();
        Thread.sleep(1000);  //主线程休眠1秒，确保两个线程执行完成
        System.out.println(value);
    }
```
&emsp;&emsp;按照预期，等待两个线程执行完后，value的值应该是20000，但在运行几次后发现，结果是一个不断变化的值：
```
thread 2 end!
thread 1 end!
11346
```
```
thread 1 end!
thread 2 end!
17040
```
&emsp;&emsp;实际上，**当两个线程同时读取value值时，可能会拿到相同的值**，这样自增操作后写回主内存，逻辑上两次的自增操作实际只会执行一次。\
&emsp;&emsp;要解决这种问题，就必须采取某种同步机制来限制不同线程对共享变量的访问！\
\
&emsp;&emsp;在之前学习数据库原理的过程中，了解到事务有其原子性的特性。**事务的原子性指的是一个事务中的所有操作，要么全部执行，要么全部都不执行，不会结束在中间环节**。如果事务在执行的过程中发生错误，要回滚(rollback)到事务前的开始状态。\
&emsp;&emsp;同理，这里就要保证共享变量value自增操作的原子性，使自增操作执行的过程中不被其他线程所打断。

## 线程锁
&emsp;&emsp;在Java中，通过synchronized关键字来创建一个线程锁，它需要在括号中填入一个对象或是一个类，现在在value自增操作外套上synchronized代码块：
```java
    private static int value = 0;
    public static void main(String[] args) throws InterruptedException{
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) 
            {
                synchronized (Main.class)   //创建线程锁
                {  
                    value++;
                }
            }
            System.out.println("thread 1 end!");
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++)
             {
                synchronized (Main.class)
                {
                    value++;
                }
            }
            System.out.println("thread 2 end!");
        });
        t1.start();
        t2.start();
        Thread.sleep(1000);  //主线程休眠1秒，确保两个线程执行完成
        System.out.println(value);
    }
```
&emsp;&emsp;这样可以得到预期的结果：
```
thread 2 end!
thread 1 end!
20000
```
&emsp;&emsp;传入synchronized的如果是对象，就是对象锁，不同的对象代表不同的对象锁。如果是类，就是类锁，类锁只有一个，实际上类锁也是对象锁，是Class类的实例。**两个线程必须使用同一把锁**。\
&emsp;&emsp;当一个线程进入到同步代码块时，会获取到当前的锁，而这时如果其他使用同样的锁的同步代码块也想执行内容，就必须等待当前同步代码块的内容执行完毕，执行完毕后会自动释放这把锁，而其他的线程才能拿到这把锁并开始执行同步代码块里面的内容。\
\
&emsp;&emsp;synchronized关键字也可以作用于方法上，当调用此方法时也会获取锁：
```java
    private static int value = 0;
    private static synchronized void add()  //锁作用于方法上
    {
        value++;
    }
    public static void main(String[] args) throws InterruptedException{
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++)
            {
                add();
            }
            System.out.println("thread 1 end!");
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++)
            {
                add();
            }
            System.out.println("thread 2 end!");
        });
        t1.start();
        t2.start();
        Thread.sleep(1000);  //主线程休眠1秒，确保两个线程执行完成
        System.out.println(value);
    }
```
&emsp;&emsp;synchronized关键字作用于方法上时，如果是静态方法，使用的就是类锁，如果是普通成员方法，使用的就是对象锁。

## 死锁
&emsp;&emsp;早在操作系统中就有接触过死锁的概念，两个或以上的进程由于竞争资源或彼此通信问题而造成进程阻塞，若没有外力的推动，两个进程将无法推进下去。\
&emsp;&emsp;当两个线程互相需要对面释放锁，进入无限等待的状态：
![图片](https://s1.328888.xyz/2022/07/24/md19h.png#pic_center)\
&emsp;&emsp;可以看到线程A与线程B被无限期阻塞，因此程序不可能正常终止。\
&emsp;&emsp;我们试运行下面的代码：
```java
    public static void main(String[] args) throws InterruptedException {
        Object o1 = new Object();
        Object o2 = new Object();
        Thread t1 = new Thread(() -> {
            synchronized(o1)        //先给线程1上锁1
            {
                try
                {
                    Thread.sleep(1000);
                    synchronized(o2)    //获取锁2
                    {
                        System.out.println("thread 1");
                    }
                }catch(InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
        });
        Thread t2 = new Thread(() -> {
            synchronized(o2){       //先给线程2上锁2
                try
                {
                    Thread.sleep(1000);
                    synchronized(o1)    //获取锁1
                    {
                        System.out.println("thread 2");
                    }
                }catch(InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
        });
        t1.start();
        t2.start();
    }
```
&emsp;&emsp;线程陷入了无限期等待，直到我们手动终止程序的运行。

## 简述悲观锁与乐观锁
&emsp;&emsp;悲观锁就是悲观思想，随时都认为有其他线程在对数据进行修改，这样别人想读写数据就会block，直到拿到锁为止，上文所提的synchronized就是一种悲观锁。\
\
&emsp;&emsp;乐观锁是一种乐观思想，操作数据时不会对操作的数据上锁，只有到数据提交时才通过某种机制来验证是否冲突（一般实现方式是添加版本号并进行比对）。
