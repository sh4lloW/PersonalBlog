---
title: "JAVA - ArrayList的大小"
date: 2022-07-13T12:49:23+08:00
categories:
    - Java
tags:
    - 集合类
draft: false
---

&emsp;&emsp;相比于数组Array而言，数组列表ArrayList的特点在于它只能存放对象数据类型的数据，如果要存放基本类型数据会涉及到Java自动拆箱与装箱的内容，还有一个特点在于它的大小是动态的，不像Array在声明时就已经确定了自己的大小。\
&emsp;&emsp;即使如此，ArrayList的大小仍然有它自己的规则，查看源码可以看到，如果在创建时有传入大小，则会以声明的大小创建ArrayList。
```java
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```
如果没有传入，则默认的大小为10。
```java
private static final int DEFAULT_CAPACITY = 10;
```
在加入新的数据时，会判断是否已满，若满则会对其扩容。
```java
private void add(E e, Object[] elementData, int s) {
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e;
        size = s + 1;
    }
```
扩容的过程并不是逐个增大，而是创建一个新变量newCapacity，将oldCapacity加上其右移一位，也就是说，扩容的大小是原来的1.5倍再+1。\
值得注意的是，源码中的copyOf以扩容后的长度创建了一个新的数组，并将原数组的内容复制了过去。
```java
    private Object[] grow(int minCapacity) {
        int oldCapacity = elementData.length;
        if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            int newCapacity = ArraysSupport.newLength(oldCapacity,
                    minCapacity - oldCapacity, /* minimum growth */
                    oldCapacity >> 1           /* preferred growth */);
            return elementData = Arrays.copyOf(elementData, newCapacity);
        } else {
            return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
        }
    }

    private Object[] grow() {
        return grow(size + 1);
    }
```
由此可以举个例子
```java
public class Main {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList();
        /*默认的长度为10*/
        list.add("it is");
        /*...*/
        /*假设已经加了8个数据项*/
        list.add("a");  //将要满时会扩容，执行完后长度为10+(10>>1)+1=16
        /*...*/
        /*假设已经加了5个数据项*/
        list.add("test");   //扩容，执行完后长度为16+(16>>1)+1=25
    }
}
```