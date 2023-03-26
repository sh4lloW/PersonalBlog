---
title: "JAVA - Stream流"
date: 2023-03-19T14:45:16+08:00
categories:
    - Java
tags:
    - 集合类
draft: false
---

​		JDK8新增了许多语法糖和新特性，其中Lambda表达式和Stream流就是常用的两个，它们可以极大减少代码的编写量，也可以使代码更加简洁。

## Lambda表达式

​		Lambda表达式是对匿名内部类的简化写法，将函数作为一个方法的参数传入，也是一种语法糖，尤其在后面Stream流中使用的非常多。

​		使用Lambda表达式替代匿名内部类需要**接口只有一个必须被实现的方法**，不是接口只有一个方法，而是接口中只能有一个必须被实现的方法，比如`default`修饰的方法有默认实现。

​		Lambda表达式的基本表达形式为：

```
(参数, [..]) -> {代码}
```

​		以下是一个Lambda表达式取代匿名内部类的例子：

```java
// 创建线程并启动
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("test");
    }
}).start();

// 用Lambda表达式来写
new Thread(() -> {
    System.out.println("test");
}).start();
```

### 基本使用

​		Lambda的基本使用在上面的例子中已经得到体现，下面列举了更多有关Lambda表达式的用法。

```java
// 无参，直接返回值
() -> {return 5;}

// 1个参数，返回其2倍
(int x) -> {return x * 2;}

// 2个参数，返回其和
(int x, int y) -> {return x + y;}

// 字符串参数，输出
(String s) -> {System.put.println(s);}
```

### 简化规则

​		实际上，Lambda表达式可以更加简化，总共有以下几个规则：

* 参数类型可以省略，比如`(int x) -> {return x * 2;}`可以简化为`(x) -> {return x * 2;}`
* 当方法体内只有一句代码时，方法体的大括号和语句的分号可以省略，并且return也可以省略，比如`(x) -> {return x * 2;}`可以简化为`(x) -> x * 2`
* 当只有一个参数时，小括号可以省略，比如`(x) -> x * 2`可以简化为`x -> x * 2`，但是多个参数不能省略小括号，比如`(int x, int y) -> {return x + y}`最多只能简化为`(x, y) -> x + y`

​		使用上面的规则，上面例子都可以写为更简洁的写法。

	## Stream流

​		Stream流同样是JDK8的新特性，使用函数式编程模式，一般用于对集合进行链式的调用与操作来处理集合，可以看作对数据操作的封装。

## Stream流流程

​		使用Stream流一般包含三部分：

* 创建流，将集合或数组转换为流的形式
* 中间操作，对集合流进行操作
* 终结操作，这一步必不可少，不进行终结操作的话无法得到中间操作的结果

​		一个集合转换成Stream流，经过中间操作的处理，最后由终结操作得到处理的结果。

​		以下是一个简单使用Stream流的例子。

```java
// 输出List中所有为偶数的值
List<Integer> list = new ArrayList<>();
for (int i = 1; i < 10; i++) {
    list.add(i);
}
list.stream()                        // 创建流
    .filter(x -> x % 2 == 0)        // 中间操作，把偶数过滤出来
    .forEach(System.out::println);  // 终结操作，挨个输出
```

​		下面列举了各个步骤的各种API。

### 创建流

#### Stream创建

​		通过创建Stream对象来创建Stream流。

```java
Stream<Integer> stream = Stream.of(1, 2, 3);
```

#### 集合创建

​		最常用的一种，直接将集合转为Stream流。

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(3);
Stream<Integer> stream = list.stream();
```

#### 数组创建

​		通过`Arrays.stream()`方法生成Stream流。

```java
int[] a = {1, 2, 3};
IntStream stream = Arrays.stream(a);
```

### 中间操作

​		一个流可以有0个也可以有多个中间操作，中间操作一般是对集合数据做处理，返回一个新流，交给下一个操作使用，但中间操作并没有进行流的遍历，仅仅只是调用了对应的方法，真正流的遍历要等到终结操作来执行。

#### filter

​		通过设置条件过滤元素。

```java
List<Person> personList = new ArrayList<>();
// 输出所有年龄大于18的person
personList.stream()
    .filter(person -> person.getAge() > 18)
    .foreach(System.out::println);
```

#### map

​		对流中的元素进行转换或计算。

```java
List<Person> personList = new ArrayList<>();
// 输出所有人的姓名
personList.stream()
    .map(person -> person.getName())
    .foreach(System.out::println);

// 对所有人的年龄进行+10输出
personList.stream()
    .map(person -> person.getAge() + 10)
    .foreach(System.out::println);
```

#### distinct

​		去除流中的重复元素。

```java
List<Person> personList = new ArrayList<>();
// 输出所有人的姓名，不输出重名的
personList.stream()
    .map(person -> person.getName())
    .distinct()
    .foreach(System.out::println);
```

#### sorted

​		对流中的元素进行排序。

```java
List<Person> personList = new ArrayList<>();
// 根据年龄倒序排序输出
personList.stream()
    .sorted((o1, o2) -> o2.getAge() - o1.getAge())
    .foreach(person -> System.out.println(person.getAge()));
```

#### limit

​		限制流的最大长度。

```java
List<Person> personList = new ArrayList<>();
// 输出年龄最大的前两个人的名字
personList.stream()
    .sorted((o1, o2) -> o2.getAge() - o1.getAge())
    .limit(2)
    .foreach(person -> System.out.println(person.getName()));
```

#### skip

​		跳过流中的前n个元素，输出剩下的。

```java
List<Person> personList = new ArrayList<>();
// 输出除了年龄最大的三个人的名字
personList.stream()
    .sorted((o1, o2) -> o2.getAge() - o1.getAge())
    .skip(3)
    .foreach(person -> System.out.println(person.getName()));
```

#### flatMap

​		与map不同的是,flatMap可以将一个对象转换为多个对象作为流中的元素，最后合并起来。

### 终结操作

​		一个流只能有一个终结操作，这个操作会真正进行流的遍历，并且操作执行后流结束，无法复用。

#### forEach

​		遍历流。

#### collect

​		将当前流转换为集合。

```java
List<Person> personList = new ArrayList<>();
// 将personList转换为Set
personList.stream()
    .collect(Collectors.toSet());
```

#### count

​		获取当前流中元素的个数。

```java
List<Person> personList = new ArrayList<>();
// 获取年龄大于18的人的个数
int cnt = personList.stream()
    .filter(person -> person.getAge() > 18)
    .count();
```

#### max && min

​		获取流中元素的最大/最小值。

```java
List<Person> personList = new ArrayList<>();
// 获得年龄最大的人的年龄
int maxx = personList.stream()
    .max(Comparator.comparingInt(User::getAge))
    .get()
    .getAge();
```

#### anyMatch

​		返回boolean，检查是否匹配至少一个元素。

```java
List<Person> personList = new ArrayList<>();
// 是否有叫张三的人
boolean flag = personList.stream()
    .anyMatch(person -> "张三".equals(person.getName()));
```

#### allMatch

​		返回boolean，检查是否匹配所有元素。

```java
List<Person> personList = new ArrayList<>();
// 是否全叫张三
boolean flag = personList.stream()
    .allMatch(person -> "张三".equals(person.getName()));
```

#### noneMatch

​		返回boolean，检查是否没有匹配所有元素。

```java
List<Person> personList = new ArrayList<>();
// 是否没人叫张三
boolean flag = personList.stream()
    .noneMatch(person -> "张三".equals(person.getName()));
```

#### findFirst

​		返回第一个元素。

```java
List<Person> personList = new ArrayList<>();
Person p = personList.stream()
    .findFirst()
    .get();
```

#### findAny

​		返回任意元素。

```
List<Person> personList = new ArrayList<>();
Person p = personList.stream()
    .findAny()
    .get();
```

#### reduce

​		对流中的数据按照自定的计算方式计算出结果。
