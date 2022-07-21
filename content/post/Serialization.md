---
title: "JAVA - 序列化"
date: 2022-07-21T17:17:11+08:00
categories:
    - Java
tags:
    - I/O
draft: false
---

&emsp;&emsp;序列化 (Serialization)是将对象的状态信息转换为可以存储或传输的形式的过程。\
&emsp;&emsp;以通俗的方式理解，假设我们需要将一个铁块从运往另一个地方，此时的做法是将铁块融化成一桶铁水，将铁水运送到目的地以后再将铁水重铸为铁块。铁块融为铁水的过程，就是**序列化**，铁水还原为铁块的过程，就是**反序列化**。\
&emsp;&emsp;以程序的角度看，**序列化**就是将一个对象变成所有计算机都能识别的字节流，**反序列化**就是将字节流还原为程序所能识别的对象。\
\
&emsp;&emsp;在Java当中，实现序列化的方式十分简单，只需要实现Serializable接口就可以了，
比如说：
```
class Person implements Serializable{
    String name;
    String id;
    public Person(String name, String id)
    {
        this.name = name;
        this.id = id;
    }
    public String toString()
    {
        return "name:"+name+"\t"+"id:"+id;
    }
}
```
&emsp;&emsp;现在利用**对象流**对Person这个对象进行序列化与其对应二进制数据进行反序列化。
```
    public static void main(String[] args) {
        serialize();
        deserialize();
    }
    private static void serialize(){    //序列化
        try(ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream("test.txt"))){
            Person p = new Person("kuma","100101");
            System.out.println(p.toString());
            System.out.println("serializing");
            outputStream.writeObject(p);
            outputStream.flush();
        }catch (IOException e)
        {
            e.printStackTrace();
        }
    }
    private static void deserialize(){      //反序列化
        try(ObjectInputStream inputStream = new ObjectInputStream(new FileInputStream("test.txt"))){
            System.out.println("deserializing");
            Person p = (Person)inputStream.readObject();
            System.out.println(p.toString());
        }catch (IOException | ClassNotFoundException e)
        {
            e.printStackTrace();
        }
    }
```
&emsp;&emsp;运行的输出结果为：
```
name:kuma	id:100101
serializing
deserializing
name:kuma	id:100101
```
\
&emsp;&emsp;在许多JDK的内部源码中，可以看到其中存在大量的**transient**关键字，它的作用就是标识这些属性不参与序列化，可以节省数据空间的占用并减少整体序列化的时间。\
&emsp;&emsp;如果我们对上面Person对象的变量name添加transient关键字：
```
class Person implements Serializable{
    transient String name;
    public Person(String name, String id)
    {
        this.name = name;
    }
    public String toString()
    {
        return "name:"+name;
    }
}
```
&emsp;&emsp;并执行上面序列化与反序列的代码，可以得到输出结果为：
```
name:kuma
serializing
deserializing
name:null
```
&emsp;&emsp;虽然得到了对象Person，但其中name并没有被序列化，所以没有被保存，结果为null。\
\
&emsp;&emsp;值得注意的是，实现了Serializable接口的类都会有一个版本号，**若没有指定版本号则JDK会根据对象的属性自动生成对应的版本号**，当然我们可以给其定义序列化版本号：
```
class Person implements Serializable{
    private static final long serialVersionUID = 9527;  //定义版本号
    String name;
    public Person(String name, String id)
    {
        this.name = name;
    }
    public String toString()
    {
        return "name:"+name;
    }
}
```
&emsp;&emsp;我们先序列化Person对象，此时指定版本号为9527，但在反序列号时将版本号手动改为95279527，显然当版本不匹配时无法反序列化为对象：
```
java.io.InvalidClassException: Person; local class incompatible: stream classdesc serialVersionUID = 9527, local class serialVersionUID = 95279527
	at java.base/java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:728)
	at java.base/java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:2086)
	at java.base/java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1933)
	at java.base/java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2259)
	at java.base/java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1768)
	at java.base/java.io.ObjectInputStream.readObject(ObjectInputStream.java:543)
	at java.base/java.io.ObjectInputStream.readObject(ObjectInputStream.java:501)
	at Main.deserialize(Main.java:25)
	at Main.main(Main.java:8)
```
\
&emsp;&emsp;
