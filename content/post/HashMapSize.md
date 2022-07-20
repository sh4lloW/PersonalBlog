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
&emsp;&emsp;在注释中可以看到，HashMap会在初次定义时进行初始化，并在必要时对自身进行扩容，其扩容的大小始终为2的倍数。\
&emsp;&emsp;每次扩容的大小为上次大小的2倍。
```
if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
```
&emsp;&emsp;其默认大小为$2^4=16$。
```
    /**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
\
&emsp;&emsp;这时涉及到一个问题，就是在什么样的情况下会对数组进行扩容。\
&emsp;&emsp;在HashMap的源码中有一个装填因子的变量，它用来表示当前情况是否需要扩容的标准，其默认值为0.75。
```
    /**
     * The load factor for the hash table.
     *
     * @serial
     */
    final float loadFactor;

    
    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
```
&emsp;&emsp;在这里，这个0.75表示的意思是当前数组的占用率达到75%时便会对其扩容。扩容会对所有的数据重新计算其哈希值，得到新的数组下标并组成新的哈希表。\
\
&emsp;&emsp;这样解决了数组扩容的问题，但链表过长的问题并没有得到解决。\
&emsp;&emsp;对此，在JDK1.8后，引入了红黑树这个数据结构，**使得链表长度达到8以后将链表自动转为红黑树**，当有了红黑树以后，查询数据便可以利用二分搜索的思想，使查询效率大幅提高。
```
    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;
    //链表长度达到8以后转为红黑树

    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
```
![图片](https://s1.328888.xyz/2022/07/20/lpG9F.png#pic_center)