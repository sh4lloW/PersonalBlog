---
title: "JAVA - 注解"
date: 2022-07-25T22:47:32+08:00
categories:
    - Java
tags:
    - 注解
draft: false
---

## 注解

&emsp;&emsp;在之前接触源码的过程中我们就已经接触到了很多注解，比如`@Override`表示重写父类方法。

&emsp;&emsp;注解可以被标注在任意地方，包括类名上、方法上、注解本身等等，就像注释一样，它相当于我们对某样东西的一个标记。而与注释不同的是，注解可以通过反射在运行时获取，注解也可以选择是否保留到运行时。

### 预设注解

&emsp;&emsp;JDK预设了以下注解，作用于代码：

- @Override - 检查（仅仅是检查，不保留到运行时）该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。
- @Deprecated - 标记过时方法。如果使用该方法，会报编译警告。
- @SuppressWarnings - 指示编译器去忽略注解中声明的警告（仅仅编译器阶段，不保留到运行时）
- @FunctionalInterface - Java 8 开始支持，标识一个匿名函数或函数式接口。
- @SafeVarargs - Java 7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。

### 元注解

&emsp;&emsp;元注解是作用于注解上的注解，一般用于我们编写的注解：

- @Retention - 标识这个注解怎么保存，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问。
- @Documented - 标记这些注解是否包含在用户文档中。
- @Target - 标记这个注解应该作用于什么类型。
- @Inherited - 标记这个注解是继承于哪个注解类(默认 注解并没有继承于任何子类)
- @Repeatable - Java 8 开始支持，标识某注解可以在同一个声明上使用多次。

注解`@Override`是如何定义的：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

&emsp;&emsp;`@Target`表示该注解只能作用于方法上，ElementType是一个枚举类型，用于表示此枚举的作用域，一个注解可以有多个作用域。\
&emsp;&emsp;`@Retention`表示该注解的只保留在代码中。\
&emsp;&emsp;一般情况下，自定义的注解需要定义1个`@Retention`和1-n个`@Target`。\
\
&emsp;&emsp;在这里定义一个属于自己的注解：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
}
```

&emsp;&emsp;将Test注解放在类和方法上：

```java
@Test
public class Main {
    @Test
    public static void main(String[] args) {
        
    }
}
```

### 注解的使用

&emsp;&emsp;我们可以在注解中定义一些属性，注解的属性也叫做成员变量，注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型：

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
    String value();
}
```

&emsp;&emsp;当只有一个属性时，默认的属性名字为value，若使用value作为属性名则在使用时不用指定：

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
    String test();
}
```

```java
public class Main {
    @Test(test = "")
    public static void main(String[] args) {

    }
}
```

&emsp;&emsp;当属性存在默认值时，使用注解的时候可以不用传入属性值，可以使用default关键字来为这些属性指定默认值：

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
    String value() default "annotation for test";
}
```

\
&emsp;&emsp;如果属性为数组，在使用注解传参时，若数组中只有一个值，可以直接传入，否则要用大括号括起来：
```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
    String[] value();
}
```
```java
    @Test("nice try")
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
```
```java
    @Test({"value1", "value2"})   //多个值时
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
```

### 反射获取注解

&emsp;&emsp;无论是方法、类、还是字段，都可以使用`getAnnotations()`方法来快速获取标记的注解。\
&emsp;&emsp;利用反射机制获取注解：

```java
    public static void main(String[] args) {
        Class<Student> clazz = Student.class;
        for (Annotation annotation : clazz.getAnnotations()) 
        {
            System.out.println(annotation.annotationType());   //获取注解类型
            System.out.println(annotation instanceof Test);   //直接判断是否为Test
        }
    }
```

&emsp;&emsp;同样可以通过这种方式获取方法上的注解：

```java
    public static void main(String[] args) throws NoSuchMethodException {
        Class<Student> clazz = Student.class;
        for (Annotation annotation : clazz.getMethod("test").getAnnotations())
        {
            System.out.println(annotation.annotationType());   //获取注解类型
            System.out.println(annotation instanceof Test);   //直接判断是否为Test
        }
    }
```
