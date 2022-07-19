---
title: "JAVA - HashMap的大小"
date: 2022-07-19T23:32:12+08:00
categories:
    - Java
tags:
    - 集合类
draft: false
---

&emsp;&emsp;哈希表的本质是一个用于存放后续结点的头结点的数组，数组下标就是数据计算得出的哈希值，在HashMap中，源码定义了结构table。
```
    /**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    transient Node<K,V>[] table;
```
&emsp;&emsp;在注释中可以看到，HashMap会在初次定义时进行初始化，并在必要时对自身进行扩容，其扩容的大小始终为2的倍数。
```
    /**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
其默认大小为$2^4=16$。
