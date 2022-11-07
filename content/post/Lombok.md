---
title: "JAVA - Lombok"
date: 2022-07-29T14:08:29+08:00
categories:
    - Java
tags:
    - Web
draft: false
---

## Lombok概述
&emsp;&emsp;“Lombok项目是一个java库，它可以自动插入到编辑器和构建工具中，增强java的性能。不需要再写getter、setter或equals方法，只要有一个注解，你的类就有一个功能齐全的构建器、自动记录变量等等。”&emsp;&emsp;&emsp;&emsp;——百度百科\
&emsp;&emsp;通俗来说，Lombok是一个可以大幅减少模板代码的工具。\
&emsp;&emsp;比如说我们要写一个Student类，它里面有id,name等等变量，我们需要对这些变量写上get,set等等方法，还需要给类加上构造函数：
```java
public class Student {
    private int id;
    private String name;
    public Student(int id, String name)
    {
        this.id = id;
        this.name = name;
    }
    public int getId()
    {
        return id;
    }
    public void setId(int id)
    {
        this.id = id;
    }
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
}
```
&emsp;&emsp;但如果用上Lombok以后，只需要添加几个注解就可以解决掉代码冗长的问题：
```java
@Getter
@Setter
@AllArgsConstructor
public class Student {
    private int id;
    private String name;
}
```

## Lombok原理
&emsp;&emsp;Lombok是一种插件化注解API，在javac进行编译时进行处理。\
&emsp;&emsp;Java的编译可以分成以下阶段：
![java编译.png](http://tva1.sinaimg.cn/large/008kE3f3gy1h7wmbcve9nj30go0233yw.jpg)
* 源文件被解析成语法树。
* 调用注解处理器。
* 语法树被分析为class文件。\
\
在第二步时，Lombok会根据自己的注解处理器，动态修改语法树，最终生成字节码。

## Lombok的使用
&emsp;&emsp;**以下内容节选自QQ:3083155908文章。**\
&emsp;&emsp;**原文链接为：https://blog.csdn.net/xiao297328/article/details/124715133**
### @Getter与@Setter
&emsp;&emsp;`@Getter` 或 `@Setter` 注解作用在类或字段上，Lombok 会自动生成默认的 getter/setter 方法。\
&emsp;&emsp;使用示例
```java
@Getter
@Setter
public class GetterAndSetterDemo {
    String firstName;
    String lastName;
    LocalDate dateOfBirth;
}
```
&emsp;&emsp;以上代码经过 Lombok 编译后，会生成如下代码：
```java
public class GetterAndSetterDemo {
    String firstName;
    String lastName;
    LocalDate dateOfBirth;
    public GetterAndSetterDemo() {
    }
    // 省略其它setter和getter方法
    public String getFirstName() {
        return this.firstName;
    }
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
}
```

### @AllArgsConstructor，@NoArgsConstructor与@RequiredArgsConstructor
&emsp;&emsp;使用 `@AllArgsConstructor` 注解可以为指定类，生成包含所有成员的构造函数。\
&emsp;&emsp;使用示例
```java
@AllArgsConstructor
public class AllArgsConstructorDemo {
    private long id;
    private String name;
    private int age;
}
```
&emsp;&emsp;以上代码经过 Lombok 编译后，会生成如下代码：
```java
public class AllArgsConstructorDemo {
    private long id;
    private String name;
    private int age;
    public AllArgsConstructorDemo(long id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
}
```
\
&emsp;&emsp;`@NoArgsConstructor` 注解可以为指定类，生成默认的构造函数。\
&emsp;&emsp;使用示例
```java
@NoArgsConstructor(staticName = "getInstance")
public class NoArgsConstructorDemo {
    private long id;
    private String name;
    private int age;
}
```
&emsp;&emsp;以上代码经过 Lombok 编译后，会生成如下代码：
```java
public class NoArgsConstructorDemo {
    private long id;
    private String name;
    private int age;
    private NoArgsConstructorDemo() {
    }
    public static NoArgsConstructorDemo getInstance() {
        return new NoArgsConstructorDemo();
    }
}
```
\
&emsp;&emsp;`@RequiredArgsConstructor` 注解可以为指定类必需初始化的成员变量，如 final 成员变量，生成对应的构造函数。\
&emsp;&emsp;使用示例
```java
@RequiredArgsConstructor
public class RequiredArgsConstructorDemo {
    private final long id;
    private String name;
    private int age;
}
```
&emsp;&emsp;以上代码经过 Lombok 编译后，会生成如下代码：
```java
public class RequiredArgsConstructorDemo {
    private final long id;
    private String name;
    private int age;
    public RequiredArgsConstructorDemo(long id) {
        this.id = id;
    }
}
```

### @EqualsAndHashCode
&emsp;&emsp;使用 `@EqualsAndHashCode` 注解可以为指定类生成 equals 和 hashCode 方法。\
&emsp;&emsp;使用示例
```java
@EqualsAndHashCode
public class EqualsAndHashCodeDemo {
    String firstName;
    String lastName;
    LocalDate dateOfBirth;
}
```
&emsp;&emsp;以上代码经过 Lombok 编译后，会生成如下代码：
```java
public class EqualsAndHashCodeDemo {
    String firstName;
    String lastName;
    LocalDate dateOfBirth;
    public EqualsAndHashCodeDemo() {
    }
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof EqualsAndHashCodeDemo)) {
            return false;
        } else {
            EqualsAndHashCodeDemo other = (EqualsAndHashCodeDemo)o;
            if (!other.canEqual(this)) {
                return false;
            } else {
              // 已省略大量代码
        }
    }
    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $firstName = this.firstName;
        int result = result * 59 + ($firstName == null ? 43 : $firstName.hashCode());
        Object $lastName = this.lastName;
        result = result * 59 + ($lastName == null ? 43 : $lastName.hashCode());
        Object $dateOfBirth = this.dateOfBirth;
        result = result * 59 + ($dateOfBirth == null ? 43 : $dateOfBirth.hashCode());
        return result;
    }
}
```

### @ToString
&emsp;&emsp;使用`@ToString`来为当前类生成预设的toString方法。\
&emsp;&emsp;使用示例
```java
@ToString(exclude = {"dateOfBirth"})
public class ToStringDemo {
    String firstName;
    String lastName;
    LocalDate dateOfBirth;
}
```
&emsp;&emsp;以上代码经过 Lombok 编译后，会生成如下代码：
```java
public class ToStringDemo {
    String firstName;
    String lastName;
    LocalDate dateOfBirth;
    public ToStringDemo() {
    }
    public String toString() {
        return "ToStringDemo(firstName=" + this.firstName + ", lastName=" +
          this.lastName + ")";
    }
}
```

### @Data
&emsp;&emsp;`@Data` 注解与同时使用以下的注解的效果是一样的：\
1.`@ToString`\
2.`@Getter`\
3.`@Setter`\
4.`@RequiredArgsConstructor`\
5.`@EqualsAndHashCode`\
&emsp;&emsp;值得注意的是，一旦使用`@Data`就不建议此类有继承关系，因为equal方法可能不符合预期结果（尤其是仅比较子类属性）。\
&emsp;&emsp;此处省略代码，具体见上面的。

### @SneakyThrows
&emsp;&emsp;`@SneakyThrows` 注解用于自动抛出已检查的异常，而无需在方法中使用 throw 语句显式抛出（相当于自动生成try-catch代码块）。\
&emsp;&emsp;使用示例
```java
public class SneakyThrowsDemo {
    @SneakyThrows
    @Override
    protected Object clone() {
        return super.clone();
    }
}
```
&emsp;&emsp;以上代码经过 Lombok 编译后，会生成如下代码：
```java
public class SneakyThrowsDemo {
    public SneakyThrowsDemo() {
    }
    protected Object clone() {
        try {
            return super.clone();
        } catch (Throwable var2) {
            throw var2;
        }
    }
}
```

### @Clean
&emsp;&emsp;`@Clean` 注解用于自动管理资源，用在局部变量之前，在当前变量范围内即将执行完毕退出之前会自动清理资源，自动生成 try-finally 这样的代码来关闭流，并在最后调用其close()方法。\
&emsp;&emsp;使用示例：
```java
public class CleanupDemo {
    public static void main(String[] args) throws IOException {
        @Cleanup InputStream in = new FileInputStream(args[0]);
        @Cleanup OutputStream out = new FileOutputStream(args[1]);
        byte[] b = new byte[10000];
        while (true) {
            int r = in.read(b);
            if (r == -1) break;
            out.write(b, 0, r);
        }
    }
}
```
&emsp;&emsp;以上代码经过 Lombok 编译后，会生成如下代码：
```java
public class CleanupDemo {
    public CleanupDemo() {
    }
    public static void main(String[] args) throws IOException {
        FileInputStream in = new FileInputStream(args[0]);
        try {
            FileOutputStream out = new FileOutputStream(args[1]);
            try {
                byte[] b = new byte[10000];
                while(true) {
                    int r = in.read(b);
                    if (r == -1) {
                        return;
                    }
                    out.write(b, 0, r);
                }
            } finally {
                if (Collections.singletonList(out).get(0) != null) {
                    out.close();
                }
            }
        } finally {
            if (Collections.singletonList(in).get(0) != null) {
                in.close();
            }
        }
    }
}
```

### @Builder
&emsp;&emsp;用 `@Builder` 注解可以为指定类实现建造者模式，该注解可以放在类、构造函数或方法上。\
&emsp;&emsp;使用示例
```java
@Builder
public class BuilderDemo {
    private final String firstname;
    private final String lastname;
    private final String email;
}
```
&emsp;&emsp;以上代码经过 Lombok 编译后，会生成如下代码：
```java
public class BuilderDemo {
    private final String firstname;
    private final String lastname;
    private final String email;
    BuilderDemo(String firstname, String lastname, String email) {
        this.firstname = firstname;
        this.lastname = lastname;
        this.email = email;
    }
    public static BuilderDemo.BuilderDemoBuilder builder() {
        return new BuilderDemo.BuilderDemoBuilder();
    }
    public static class BuilderDemoBuilder {
        private String firstname;
        private String lastname;
        private String email;
        BuilderDemoBuilder() {
        }
        public BuilderDemo.BuilderDemoBuilder firstname(String firstname) {
            this.firstname = firstname;
            return this;
        }
        public BuilderDemo.BuilderDemoBuilder lastname(String lastname) {
            this.lastname = lastname;
            return this;
        }
        public BuilderDemo.BuilderDemoBuilder email(String email) {
            this.email = email;
            return this;
        }
        public BuilderDemo build() {
            return new BuilderDemo(this.firstname, this.lastname, this.email);
        }
        public String toString() {
            return "BuilderDemo.BuilderDemoBuilder(firstname=" + this.firstname + ", lastname=" + this.lastname + ", email=" + this.email + ")";
        }
    }
}
```

## 使用Lombok的优缺点
&emsp;&emsp;优点：
* 能通过注解的形式自动生成构造器，提高了一定的开发效率。
* 让代码变得简洁，不用过多的去关注相应的方法。
* 属性做修改时，也简化了维护为这些属性所生成的getter/setter方法等。

&emsp;&emsp;缺点：
* 代码可读性，可调试性低。
* 破坏封装性，无脑使用 getter 和 setter 不如不使用。
* Lombok 违反了 Java annotation processor 的规定，使用 HACK 字节码的方式实现这些方法，可能会导致一些不好发现的问题，在多个 JVM 语言混用时容易出一些离奇的错误。