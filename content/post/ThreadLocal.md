---
title: "JAVA - 线程变量ThreadLocal"
date: 2022-07-24T21:34:17+08:00
categories:
    - Java
tags:
    - 多线程
draft: false
---

&emsp;&emsp;每个线程都有一个私有的工作内存（本地内存），工作内存中存放了该线程的实例副本。\
&emsp;&emsp;使用ThreadLocal类可以创建工作内存中的变量，它将变量值存储在内部（只能存储一个变量），不同的变量访问到ThreadLocal对象时，都只能获取到自己线程所属的变量。

```java
    public static void main(String[] args) throws InterruptedException {
        ThreadLocal<String> local = new ThreadLocal<>();
        Thread t1 = new Thread(() -> {
            local.set("it is a var!");   //将变量的值给予ThreadLocal
            System.out.println(local.get());   //尝试获取ThreadLocal中存放的变量
        });
        Thread t2 = new Thread(() -> {
            System.out.println(local.get());
        });
        t1.start();
        Thread.sleep(3000);    //间隔三秒
        t2.start();
}
```
&emsp;&emsp;执行的结果为：
```
it is a var!
null
```
&emsp;&emsp;尝试将第一个线程存入后，第二个线程也存放，是否会覆盖第一个线程存放的内容：
```java
    public static void main(String[] args) throws InterruptedException {
        ThreadLocal<String> local = new ThreadLocal<>();
        Thread t1 = new Thread(() -> {
            local.set("it is a var!");   //将变量的值给予ThreadLocal
            System.out.println(local.get());   //尝试获取ThreadLocal中存放的变量
        });
        Thread t2 = new Thread(() -> {
            local.set("it is var2!");
            System.out.println(local.get());
        });
        t1.start();
        Thread.sleep(3000);    //间隔三秒
        t2.start();
    }
```
&emsp;&emsp;执行的结果为：
```
it is a var!
it is var2!
```
&emsp;&emsp;由此可得，不同线程向ThreadLocal存放数据，只会存放在线程自己的工作空间中，而不会直接存放到主内存中，因此各个线程直接存放的内容互不干扰。\
\
&emsp;&emsp;在线程中创建的子线程，无法获得父线程工作内存中的变量：

```java
public static void main(String[] args) {
    ThreadLocal<String> local = new ThreadLocal<>();
    Thread t = new Thread(() -> {
       local.set("try child thread!");
        new Thread(() -> {
            System.out.println(local.get());
        }).start();
    });
    t.start();
}
```

&emsp;&emsp;对此可以使用InheritableThreadLocal来解决：

```java
public static void main(String[] args) {
    ThreadLocal<String> local = new InheritableThreadLocal<>();
    Thread t = new Thread(() -> {
       local.set("lbwnb");
        new Thread(() -> {
            System.out.println(local.get());
        }).start();
    });
    t.start();
}
```

&emsp;&emsp;在InheritableThreadLocal存放的内容，会自动向子线程传递。