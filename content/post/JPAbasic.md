---
title: "JAVA - JPA基础"
date: 2022-11-02T08:08:08+08:00
categories:
    - Java
tags:
    - 框架
draft: false
---

## JPA和MyBatis

​		在使用MyBatis的过程中，我们编写SQL语句将获取的数据映射为对应的Java对象，通过调用Mapper中的方法直接获得实体类，这样就完成了Java项目与数据库的交互。MyBatis是读取数据并封装为实体类，而JPA是直接将实体类对应到数据库表，一张表里有什么字段，对象就有什么属性，所有属性和数据库中的字段一一对应，并且它封装了SQL语句的编写，框架根据定义的映射关系自动生成相关的SQL语句，这样就不用手动编写SQL了（有点类似MyBatis-Plus）。

​		JPA实际是一种ORM(Object-Relationl Mapping)规范，并不是一种框架，Hibernate才是JPA的实现，它的学习难度相比MyBatis较高，时间也较为久远，所以现在结合SpringBoot都是使用SpringDataJPA，它是采用Hibernate作为底层实现，并对其加以封装。

​		官方文档：[SpringDataJPA](https://spring.io/projects/spring-data-jpa)

## 使用SpringDataJPA

### 前置工作

​	导入依赖：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

​		创建实体类以对应数据库中的表，通过注解添加数据库映射关系：

```java
@Data
@Entity   					//表示这个类是一个实体类
@Table(name = "student")    //对应的数据库中表名称
public class Student {

    @GeneratedValue(strategy = GenerationType.IDENTITY)   //生成策略，这里配置为自增
    @Column(name = "sid")    //对应表中sid这一列，下同
    @Id     				 //此属性为主键
    int sid;

    @Column(name = "name")
    String name;

    @Column(name = "grade")
    String grade;
}
```

​		接着需要在配置文件中修改JPA的配置：

```yaml
spring:
  jpa:
    show-sql: true	# 日志显示SQL语句执行
    hibernate:
      ddl-auto: update
      # ddl-auto属性用于设置表的定义，共有四种
      
      # create：启动时删除数据库中的表后创建，退出时不删除数据表
      # create-drop：启动时删除数据库中的表后创建，退出时删除数据表，如果表不存在会报错
      # update：如果启动时表格式不一致则更新表，原有数据保留
      # validate：项目启动表结构进行校验，如果不一致则报错
```

​		这时数据库与Java项目的基本映射就已经完成了，接下来就是需要一个Mapper来访问数据表，不过在JPA中它叫Repository：

```jAVA
@Repository		// 注解表明它是Repository
public interface StudentRepository extends JpaRepository<Student, Integer> {
    // Repository仍然是一个接口，它继承自JpaRepository，JpaRepository有两个泛型，第一个是对象实体，第二个是Id的类型
}
```

​		至此前置工作已经全部完成，SpringDataJPA根据表内容已经定义了一些初始的SQL语句供我们使用，包括增删改查等等：

```java
@SpringBootTest
class TestApplicationTests {

    @Resource
    StudentRepository repository;

    @Test
    void contextLoads() {
      	// 根据Id查找学生信息
        System.out.pringln(repository.findById(1));
        // 添加数据
        Student student = new Student();
    	student.setName("张凯");
    	student.setGrade("1");
    	repository.save(student);
        // 根据Id删除信息
        repository.deleteById(2);
    }

}
```

​		JPA依靠提供的注解信息自动完成了关系的映射与关联，并且没有出现任何的SQL语句，所以相比于MyBatis这样的半自动ORM框架来说，JPA几乎是一个全自动的ORM框架，它屏蔽了一些有关SQL的操作，让项目更加“面向对象”。

### 拼接自定义的SQL

​		JPA虽然提供了基础的数据库操作，但如果需要条件查询这种稍显复杂点的，还是需要自己在Repository定义的，只不过它同样不需要编写SQL语句，而是通过方法名称进行拼接。

​		具体拼接规则见官方SpringDataJPA教程的附录C：[SpringDataJPA官方教程 Version:2.7.5](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)

​		事实上也不需要对规则特别记忆，因为它的拼接逻辑和SQL语句关键字的拼接逻辑也是类似的，比如想要同时通过sid和name来查询学生的信息：

```java
@Repository
public interface StudentRepository extends JpaRepository<Student, Integer> {
    Student findByIdAndUsername(int id, String username);
}
```

### 关联查询

​		关联查询同样是较为常用的使用情况，在JPA中，实体类是表的映射，而表之间的关联关系，也可以看作是对象间的依赖关系。

​		此时还有一个表StudentDetail，里面记录了学生的详细信息，Student表中有字段作为StudentDetail的外键，那么只需要添加注解进行声明：

```java
@Data
@Entity
@Table(name = "student")
public class Student {

    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "sid")
    @Id
    int sid;

    @Column(name = "name")
    String name;

    @Column(name = "grade")
    String grade;
    
    @JoinColumn(name = "detail_sid")	// 指定存储外键的字段名称
    @OneToOne		// 声明一对一关系
    StudentDetail deatil;
}
```

​		提前建好StudentDetail实体类：

```java
@Data
@Entity
@Table(name = "student_detail")
public class AccountDetail {

    @Column(name = "sid")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Id
    int sid;

    @Column(name = "address")
    String address;

    @Column(name = "phoneNumber")
    String phoneNumber;
}
```

​		一对多与多对一的使用类似，他们的注解分别是：

```java
@OneToMany	// 一对多
@ManyToOne	// 多对一
```

​		多对多需要中间表来存储两张表的关联信息，所以使用的不是`@JoinColumn`注解，而是`@JoinTable`。

​		假设一个老师教多个学生，一个老师有多个学生：

```java
@ManyToMany   //多对多
@JoinTable(name = "teach_relation",     //中间关联表名字
        joinColumns = @JoinColumn(name = "sid"),    //当前实体主键在关联表中的字段名称
        inverseJoinColumns = @JoinColumn(name = "tid")   //教师实体主键在关联表中的字段名称
)
List<Teacher> teacher;
```

### 自定义SQL的编写

#### JPQL

​		如果需要复杂查询的场景，同样是需要手动编写很长的SQL语句，JPA同样可以直接编写SQL，不过JPA使用的是JPQL语言，它和SQL长得很像，不过是面向对象的。

```java
@Repository
public interface StudentRepository extends JpaRepository<Student, Integer> {

    @Transactional    //DML操作需要事务环境，也可以在使用时声明
    @Modifying     	 //表示这是一个DML操作
    @Query("update Student set grade = ?2 where sid = ?1") //这里操作的是一个实体类对应的表，参数使用?代表，后面接第n个参数
    int updateGradeBySid(int sid, String newGrade);
}
```

```java
@Test
void Test(){
    repository.updateGradeBySid(1, "2");
}
```

#### 原生SQL

​		如果仍然想用原生的SQL语句，只需要在`@Query`注解中设置nativeQuery属性即可：

```java
@Repository
public interface StudentRepository extends JpaRepository<Student, Integer> {

    @Transactional
    @Modifying
    @Query("update student set grade = :grade where sid = :sid" nativeQuery = true)
    int updateGradeBySid(@Param("sid") int sid, @Param("grade") String newGrade);
}
```

## JPA与MyBatis各有的优势

摘自https://blog.csdn.net/weixin_57393819/article/details/125515358。

**JPA**：

* **标准化** ：提供相同的访问API，保证了基于JPA开发的企业应用能够经过少量的修改就能够在不同的JPA框架下运行。

* **简单易用，集成方便** ：JPA的主要目标之一就是提供更加简单的编程模型：在JPA框架下创建实体和创建Java 类一样简单，没有任何的约束和限制，只需要使用 javax.persistence.Entity进行注释，JPA的框架和接口也都非常简单。

* **和JDBC的查询能力差不多** ：JPA的查询语言是面向对象而非面向数据库的，JPA定义了独特的JPQL（Java Persistence Query Language），JPQL是EJB QL的一种扩展，它是针对实体的一种查询语言，操作对象是实体，而不是关系数据库的表，而且能够支持批量更新和修改、JOIN、GROUP BY、HAVING 等通常只有 SQL 才能够提供的高级查询特性，甚至还能够支持子查询。

* **支持面向对象的高级特性** ：JPA 中能够支持面向对象的高级特性，如类之间的继承、多态和类之间的复杂关系，这样的支持能够让开发者最大限度的使用面向对象的模型。

  面向对象也避免了程序与数据库 SQL 语句耦合严重，比较适合跨数据源的场景（一会儿 MySQL，一会儿 Oracle 等）。

**MyBatis**

* **简单易学**：本身就很小且简单。没有任何第三方依赖，最简单安装只要两个jar文件+配置几个sql映射文件。
  灵活：mybatis不会对应用程序或者数据库的现有设计强加任何影响。
* **sql写在xml里，便于统一管理和优化** ： 通过sql语句可以满足操作数据库的所有需求。
* **解除sql与程序代码的耦合**：通过提供DAO层，将业务逻辑和数据访问逻辑分离，使系统的设计更清晰，更易维护，更易单元测试。sql和代码的分离，提高了可维护性。
* **提供映射标签，支持对象与数据库的ORM字段关系映射** ： 提供对象关系映射标签，支持对象关系组建维护。 提供xml标签，支持编写动态sql。









​		
