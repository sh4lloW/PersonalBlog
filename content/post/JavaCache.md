---
title: "JAVA - Integer的常量存储池"
date: 2022-07-12T23:00:23+08:00
categories:
    - Java
tags:
    - Cache
    - 基本类型包装类
draft: false
---

JAVA可以依靠自动装箱与拆箱机制实现赋值创建对象，如：
```java
Integer i = 1
```
当出现如下代码时：
```java
Integer i1 = 127;
Integer i2 = 127;
Integer i3 = 128;
Integer i4 = 128;
System.out.println(i1 == i2);
System.out.println(i1 == i3);
System.out.println(i3 == i4);
```
输出结果为：
```
true
false
false
```
当打开Integer包装类的源码时，我们可以看到：
```java
public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
```java
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer[] cache;
        static Integer[] archivedCache;

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    h = Math.max(parseInt(integerCacheHighPropValue), 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(h, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            // Load IntegerCache.archivedCache from archive, if possible
            CDS.initializeFromArchive(IntegerCache.class);
            int size = (high - low) + 1;

            // Use the archived cache if it exists and is large enough
            if (archivedCache == null || size > archivedCache.length) {
                Integer[] c = new Integer[size];
                int j = low;
                for(int i = 0; i < c.length; i++) {
                    c[i] = new Integer(j++);
                }
                archivedCache = c;
            }
            cache = archivedCache;
            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```
可以看到在源码中对Integer的缓冲做了约束，**在类加载时将[-128.127]区间的值保存在cache数组中，一旦调用valueOf方法则会直接在cache中取Integer对象。**\
**若不在区间内，则会new一个Integer对象来存放值。**\
\
在其他包装类中，有一些同样具有缓冲池：\
Character：<=127\
Short:[-128,127]\
Long:[-128,127]\
Boolean,Byte,Float,Double没有对应的缓冲池