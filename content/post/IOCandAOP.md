---
title: "JAVA - IOC和AOP"
date: 2022-09-17T13:45:19+08:00
categories:
    - Java
tags:
    - Spring
    - 框架
draft: false
---

​		IOC和AOP是Spring框架的两大核心概念，但这两个概念并不是由Spring提出来的，它们在Spring之前就已经存在，不过Spring在技术层面将这两个理论做了很好的实现。

# IOC

## 什么是IOC

​		IOC(Inversion of control)多数时候译成**控制反转**，它是一种面向对象编程的思想，不是技术实现。

​		在面向对象的编程中，底层都是由多的对象构成，各个对象相互合作实现业务逻辑，就像齿轮一样同时工作，密不可分，这里可以用上经典齿轮图来描述对象间的耦合关系：

![图片](https://s1.328888.xyz/2022/09/17/o4V2k.png)

​		这样的体系结构虽然清晰，但是存在一个很严重的问题就是耦合度过高。在软件项目高速迭代的今天，一个项目之前实现的功能需要修改甚至全部推翻的情况经常出现，但在这样的体系结构下，一个模块的修改很容易造成牵一发而动全身的结果。在项目比较庞大时，这样造成的后果是很严重的，因此，为了解决这种高耦合度的问题，我们必须将各个模块进行解耦，让各个模块的直接依赖性不那么强。对此，就引入了IOC理论。

​		对于实现类的对象，我们不再通过自己创建来获得对象，而是通过IOC容器来帮我们实例化对象，当需要某个类时，直接从IOC容器中取出。

​		所以，**控制反转中的控制指的就是对象管理的权力，反转指的是将控制权由开发者交给外部环境（IOC容器）**。

​		IOC容器相当于第三方，这样就达到了耦合度降低的目的：

![图片](https://s1.328888.xyz/2022/09/17/o3C6r.png)

​		高内聚，低耦合是现代软件的开发目标之一，Spring框架提供了一个IOC容器进行对象的管理。

## IOC容器的使用

### 使用Spring框架

​		首先在Maven项目中导入Spring依赖：

```xml
<dependency>
	<groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.22</version>
</dependency>
```

​		在resources中创建Spring的xml配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

​		最后在主方法中编写：

```java
public static void main(String[] args) {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-config.xml");	// 括号中是配置文件的名字
    
}
```

​		这时一个最基本的Spring项目就创建完了。

### IOC容器管理JavaBean

​		创建一个Student类：

```java
public class Student {
    String name;
    int age;
}
```

​		在Spring配置文件中注册为Bean：

```xml
<bean name="student" class="com.test.bean.Student"/>
<!-- name是这个Bean的名字，可以随意起，不一定和类一样 -->
<!-- class是类的路径 -->
```

​		这样就可以在主方法中使用类了，注意这时不是直接new一个Student类，而是由IOC容器提供类：

```java
public static void main(String[] args) {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-config.xml");
    Student student = context.getBean(Student.class);
    // 也可以写成Student student = (Student) context.getBean("student");	注意引号内写的是Bean的名字
    System.out.println(student);
}
```

​		通过IOC容器管理的对象默认是单例模式的，但可以在配置文件中修改其作用域：

```xml
<bean name="student" class="com.test.bean.Student" scope="prototype"/>
<!-- 通过将作用域设为prototype原型模式 -->
```

​		在单例模式中，Bean一开始就会被创建，被IOC容器存储，只要容器不被销毁，对象就会一直存在，而在原型模式中，获取才会new一个新的对象，并不会被保存。



​		还可以通过配置文件，给一个对象添加初始化方法和销毁方法，首先先在类中编写初始化的方法和销毁方法，然后在配置文件中添加对应的方法即可：

```java
public class Student {
    String name;
    int age;

    private void init(){
        System.out.println("init");
    }

    private void destroy(){
        System.out.println("destory");
    }

    public Student(){
        System.out.println("Student");
    }
}
```

```xml
<bean name="student" class="com.test.bean.Student" init-method="init" destroy-method="destroy"/>
<!-- 在init-method和destroy-method属性中添加类中编写的初始化和销毁方法 -->
<!-- 初始化方法会在Bean被创建时就执行，销毁方法会在对象销毁时执行 -->
<!-- 如果是在默认单例模式下，IOC容器关闭时就是对象销毁的时候 -->
```

```java
public static void main(String[] args) {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-config.xml");
    Student student = context.getBean(Student.class);
    System.out.println(student);
    context.close();  //手动销毁容器
}
```

### 依赖注入

​		依赖注入DI(Dependency Injection)可以理解为IOC思想的实现，IOC容器在创建对象时，需要将预先的属性注入到对象中，这时需要在对象中提前写好`set`方法，然后在配置文件中使用`property`标签实现：

```java
public class Student {
    String name;
    int age;

    public void setName(String name) {
        this.name = name;
    } // 使用Lombok的会自动添加set方法
}
```

```xml
<bean name="student" class="com.test.bean.Student">
    <property name="name" value="张凯"/>
</bean>
```

​		如果注入的属性不是基本类型，而是一个对象，那可以使用`ref`属性：

```xml
<bean name="teacher" class="com.test.bean.Teacher"/>
<!-- 这时有另外一个Teacher类 -->
<bean name="student" class="com.test.bean.Student">
    <property name="name" value="张凯"/>
    <property name="teacher" ref="teacher"/>
    <!-- 注意引用的是Bean的名字而不是类的名字 -->
</bean>
```

​		除了利用`set`方法实现以外，还可以指定构造方法，首先在Student类中写好带参构造方法：

```java
public class Student {
    String name;
    int age;

    public Student(String name, int age){
        this.name = name;
        this.age = age;
    }
}
```

​		然后在配置文件中使用`constructor-arg`标签：

```xml
<bean name="student" class="com.test.bean.Student">
	<constructor-arg name="name" value="张凯"/>
	<constructor-arg name="age" value="18"/>
	<!-- 同样可以通过下标指定，比如说name为下标为0的参数注入值，就可以写： -->
    <!-- index="0" value="张凯" -->
</bean>
```

​		此外还有第三种：接口注入，不过这种方式会使对象实现不必要的接口，带有侵入性，所以现在并不提倡使用。

# AOP

## 什么是AOP

​		AOP(Aspect Oriented Programming)通常译作面向切面编程，是一种延续自OOP的编程思想，通过预编译和动态代理的方式将代码切入到类的指定位置，在方法执行前或执行后做某些操作。

​		如果将整个项目看作是一个圆柱体，AOP的思想就是将圆柱体在某个位置切开，在切开的位置添加上额外的内容，然后再将原本的切面无缝严合上。

## 利用Spring实现AOP

​		Spring是支持AOP的框架，在使用之前先导入依赖：

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-aspects</artifactId>
	<version>5.3.22</version>
</dependency>
```

​		找到需要切入的类：

```java
public class Student {
    String name;
    int age;
    
	//分别在test方法执行前后切入
    public void test() {
        System.out.println("it is a test");
    }
}
```

​		单独建立一个AOP操作的类，在其中写切入的方法：

```java
public class AopTest {
    //在test()执行之前执行该方法
    public void before(){
        System.out.println("before running...");
    }

    //在test()执行之后执行该方法
    public void after(){
        System.out.println("after running");
    }
}
```

​		现在被切入的类和切入的操作都已经写好了，接下来就是如何告诉Spring执行切入，在Spring配置文件添加AOP配置（当然在这之前先把这两个类注册为Bean）：

```xml
<aop:config>
    <aop:pointcut id="test" expression="execution(* com.test.bean.Student.test())"/>
    <!-- execution中的表达式为execution(修饰符 包名.类名.方法名称(方法参数)) -->
    <!-- *可以代表任意修饰符或任意方法任意参数 -->
    <aop:aspect ref="aopTest">
    	<aop:before method="before" pointcut-ref="test"/>
    	<aop:after-returning method="after" pointcut-ref="test"/>
        <!-- pointcut-ref引用的是上面切点的名字 -->
	</aop:aspect>
</aop:config>
```

​		除此以外也可以使用`@annotation`注解来标记哪些被注解的方法被切入。

​		接下来就可以执行test()方法进行测试了。



​		除了before和after以外，常用的还有环绕方法around，使用环绕方法相当于完全代理该方法，需要手动调用才会执行该方法：

```java
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    // 通过JoinPoint形参可以获取一个实现类对象
    System.out.println("before running...");
    Object value = joinPoint.proceed();
    // 将该实现类对象赋值给value
    System.out.println("the value is: " + value);
    System.out.println("after running...");
    return value;
}
```



​		除了这种方式以外还可以通过`Advice`接口来实现AOP操作，这里不写了。

# 利用注解实现

​		前面利用配置文件来实现IOC和AOP，但好像并没有达到简化开发的目的，编写大量的配置仍然需要很多精力和时间，所以，相比较来说，注解是更加简便高效的方式，也是现在常用的方式。

## 注解代替配置文件

​		在使用注解的情况下就不用Spring的配置文件了，只需要新建一个配置类，加上`@Configuration`：

```java
@Configuration
public class MainConfiguration {
    
}
```

​		它取代了Spring配置中的内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

​		需要注册为Bean的类也只需要在类上加上`@Bean`注解就可以了：

```java
@Configuration
public class MainConfiguration {
	@Bean
    public Student student() {
        return new student();
    }
}
```

​		在主方法中加载配置类：

```java
public class Main {
    public static void main(String[] args) {
        // 之前使用ClassPathXmlApplicationContext代表是用配置文件
        // 这里使用AnnotationConfigApplicationContext代表是用注解配置
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
        // 指定配置类
        Student student = context.getBean(Student.class);  // 容器用法和之前一样
        System.out.println(student);
    }
}
```

## 注解实现Bean注册和依赖注入

​		相比于使用`@Bean`注解，`@Component`注解是更常见的一种Bean注册方法，在使用这个注解前需要给配置类添加`@ComponentScan`注解，用于对某个包下所有添加了`@Component`注解的类进行扫描。

```java
@ComponentScan("com.test.bean")
@Configuration
public class MainConfiguration {

}
```

​		给某个类上添加`@Component`注解，表明这个类作为Bean交给Spring提供的IOC容器进行管理：
```java
@Component
public class Student {
    String name;
    int age;
}
```

​		将属性注入到Bean中需要用到`@Resource`或`@Autowired`注解：

```java
@Component
public class Student {
    String name;
    int sid;
    
    @Resource
    Teacher teacher;
}
```

* @Resource默认**ByName**如果找不到则**ByType**，可以添加到set方法、字段上。
* @Autowired默认是**ByType**，可以添加在构造方法、set方法、字段、方法参数上。

​		IDEA不推荐将`@Autowired`使用在字段上，但可以放在方法上。

## 注解实现AOP

​		注解实现AOP同样不需要配置文件，只需要在配置类前添加`@EnableAspectJAutoProxy`注解：

```java
@EnableAspectJAutoProxy
@ComponentScan("com.test.bean")
@Configuration
public class MainConfiguration {
    
}
```

​		接下来只需要在执行AOP操作的类上加上`@Aspect`注解，当然，同样需要将其注册为Bean：

```java
@Component
@Aspect
public class AopTest {

}
```

​		同样，切入的方法也可以用注解来完成：
```java
@Component
@Aspect
public class AopTest {
    @Before("execution(* com.test.bean.Student.test())")
    public void before(){
        System.out.println("before running...");
    }

    @AfterReturning("execution(* com.test.bean.Student.test(..))"")
    public void after(){
        System.out.println("after running");
    }
}
```

