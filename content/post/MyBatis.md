---
title: "JAVA - MyBatis"
date: 2022-08-13T18:27:19+08:00
categories:
    - Java
tags:
    - Web
    - 框架
    - Mybatis
draft: false
---
&emsp;&emsp;MyBatis算是我接触的第一个框架，学习这个的过程有些漫长，认识到了xml这种以前见过但是不知道是什么的东西，也第一次看某样东西官方的文档，不管怎么样还是有所收获的。\
&emsp;&emsp;下面是MyBatis的中文官方文档：\
&emsp;&emsp;[MyBatis文档](https://mybatis.org/mybatis-3/zh/index.html)

## Mybatis是什么
&emsp;&emsp;用文档的话来说：\
&emsp;&emsp;MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。\
&emsp;&emsp;简单来说，在与数据库交互方面，相比于JDBC，使用MyBatis是一种更加简洁高效的方式。

## 简述XML语言
&emsp;&emsp;XML 指可扩展标记语言（EXtensible Markup Language）。一般来说长这样：
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!-- xml声明，定义xml的版本和使用的编码，非必须 -->
<outer>
  <name>Jane</name>
  <inner type="1">
    <sex>male</sex>
    <age>18</age>
  </inner>
</outer>
```
&emsp;&emsp;它和HTML长得很像，但它们的意义完全不同，**XML被设计用来传输和存储数据，HTML被设计用来显示数据。**\
&emsp;&emsp;一个xml文件有以下格式规范：
* 必须存在一个根结点，将所有的子标签全部包含。比如在上面的XML文件中，`<outer>`就是根结点，`<name>`，`<inner>`等等就是子标签。
* 所有的标签必须成对出现，比如`<name>Jane</name>`，标签之间可以嵌套但是不能交叉嵌套
* 区分大小写。
* 标签中可以存在属性，比如type="1"就是inner标签的一个属性，属性的值由单引号或双引号包括。\
\
更详细的教程：[XML教程](https://www.runoob.com/xml/xml-tutorial.html)\
在MyBatis中，重点是如何使用XML来作为MyBatis的配置文件。

## MyBatis的配置
&emsp;&emsp;首先要去官方Github上面下载jar依赖包（不用Maven的情况）：[mybatis/mybatis-3](https://github.com/mybatis/mybatis-3)\
&emsp;&emsp;根据官方文档的流程，先在项目的根目录下创建xml文件，该xml文件用于MyBatis的配置，填写以下内容：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <!-- dtd文件是文档类型的定义，提前帮助我们规定了一些标签，用这些标签使得MyBatis可以正确识别 -->
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/> <!-- 驱动类 -->
        <property name="url" value="${url}"/> <!-- 数据库连接URL -->
        <property name="username" value="${username}"/> <!-- 用户名 -->
        <property name="password" value="${password}"/> <!-- 密码 -->
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="mapper/TestMapper"/> <!-- Mapper的路径，下面会说 -->
  </mappers>
</configuration>
```
&emsp;&emsp;配置完成后在Java代码中创建MyBatisUtil类，用以使用MyBatis：
```java
public class MybatisUtil {
    private static SqlSessionFactory sqlSessionFactory;
    // 创建SqlSessionFactory对象
    static
    {
        try{
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(new FileInputStream("mybatis-config.xml"));
            // 读取xml文件
        }catch(FileNotFoundException e){
            e.printStackTrace();
        }
    }
    public static SqlSession getSession(boolean autoCommit){ // 该方法用于获取新会话
        return sqlSessionFactory.openSession(autoCommit); // autoCommit为自动提交，关闭的话就会变为事务操作
    }
}
```
&emsp;&emsp;每个基于`MyBatis`的应用都是以一个`SqlSessionFactory`的实例为核心的，我们可以通过`SqlSessionFactory`来创建多个新的会话，每个会话就相当于从不同的地方登陆一个账号去访问数据库，也可以认为这就是之前JDBC中的`Statement`对象，会话之间相互隔离，没有任何关联。\
![SqlSession.png](http://tva1.sinaimg.cn/large/008kE3f3gy1h7wmcehi1xj30c609hmxs.jpg)
&emsp;&emsp;`SqlSessionFactory`一般只用创建一次，所以创建MybatisUtil类作为工具类来管理，这样就可以直接在主程序当中调用了。\
&emsp;&emsp;在此之前，先创建一个测试用的实体类用于读取（建议新建一个entity文件夹然后放在里面，便于统一管理)：
```java
@Data
public class Student {
    int sid;   //名称最好和数据库字段名称保持一致，不然可能会映射失败导致查询结果丢失
    String name;    //当然如果不对应也有解决的办法，后面会提及
    String sex;
}
```
&emsp;&emsp;在根目录下创建一个mapper文件夹，新建名为`TestMapper.xml`的文件作为映射器：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="TestMapper">     <!-- 命名空间，用于标识这个mapper -->
    <select id="selectStudent" resultType="entity.Student">
        <!-- id是这个操作的名字，等会使用时会用到，retultType表示映射到哪个类中 -->
        select * from student   <!-- SQL语句 -->
    </select>
</mapper>
```
&emsp;&emsp;这样就可以在main()方法中执行查询操作了：
```java
public static void main(String[] args) {
    try (SqlSession sqlSession = MybatisUtil.getSession(true)){
        List<Student> student = sqlSession.selectList("selectStudent"); // 结果装入到List中
        student.forEach(System.out::println);
    }
}
```
&emsp;&emsp;这样是通过一个映射器来将结果转化为一个实体类，但还是不够方便，每次都要在xml文件中找对应操作的名称，我们可以通过`namespace`来绑定到一个接口上，利用接口的特性直接指明方法的行为，而实际实现由MyBatis完成：
```java
public interface TestMapper {
    List<Student> selectStudent();
}
```
&emsp;&emsp;将Mapper文件的命名空间改为接口：
```xml
<mapper namespace="mapper.TestMapper">
    <select id="selectStudent" resultType="entity.Student">
        select * from student
    </select>
</mapper>
```
&emsp;&emsp;将Mapper文件放入到同一个包中，作为内部资源，在配置文件中，mapper的定义为resourse就表明其是一个内部资源。\
&emsp;&emsp;这样就能直接通过接口中定义的行为来获取结果了：
```java
public static void main(String[] args) {
    try (SqlSession sqlSession = MybatisUtil.getSession(true)){
        TestMapper testMapper = sqlSession.getMapper(TestMapper.class);
        //这里涉及到反射
        List<Student> student = testMapper.selectStudent();
        student.forEach(System.out::println);
    }
}
```
&emsp;&emsp;关于MyBatis更多的配置项可以查看官方文档，重心是MyBatis的使用。

## 基本SQL语句的执行
### CRUD

&emsp;&emsp;上面已经提及了基本的查询操作，如果需要传入查询条件，比如说想通过学号sid来查找信息：
```java
Student getStudentBySid(int sid); // 接口中
```
```xml
<select id="getStudentBySid" parameterType="int" resultType="Student">
    select * from student where sid = #{sid} <!-- xml文件中 -->
</select>
```
&emsp;&emsp;这里的`#{sid}`就是要传入的参数sid，Mybatis的本质也是通过`PreparedStatement`进行预编译，还有一种写法是`${sid}`，但这样就和在JDBC当中直接传值一样，具有被SQL注入攻击的风险，所以使用`#{sid}`的安全性会更高。\
&emsp;&emsp;在上面有提及过，当实体类属性与数据库内部的名称不一致时，会导致映射失败，这里有一种解决的办法，就是自定义`resultMap`来设定映射规则：
```xml
<resultMap id="Test" type="Student">
    <result column="sid" property="id"/>
    <!-- column是数据库中的字段名，property是实体类的属性 -->
</resultMap>
```
&emsp;&emsp;这样在实体类`Student`中，学号改为`id`就能正常映射了。\
\
&emsp;&emsp;其他的CRUD操作和查询差不多，比如说试一下Insert操作：
```java
int addStudent(Student student); // 接口中
```
```xml
<insert id="addStudent" parameterType="Student">
    insert into student(name, sex) values(#{name}, #{sex}) <!-- xml文件中 -->
</insert>
```
&emsp;&emsp;如果不想用实体类，属性也可以映射到Map上：
```java
List<Map> selectStudent();
```
```xml
<select id="selectStudent" resultType="Map">
    select * from student
</select>
```

### 多表联查
&emsp;&emsp;一个老师可以教导多个学生，现在创一个Teacher类：
```java
@Data
public class Teacher {
    int tid;
    String name;
    List<Student> studentList; // 学生信息用List存放
}
```
&emsp;&emsp;显然这时是一个一对多的查询，该对此使用`resultMap`来自定义映射规则：
```xml
<select id="getTeacherByTid" resultMap="asTeacher">
        select *, teacher.name as tname from student 
        inner join teach on student.sid = teach.sid 
        inner join teacher on teach.tid = teacher.tid where teach.tid = #{tid}
        <!-- SQL语句 -->
</select>

<resultMap id="asTeacher" type="Teacher"> <!-- 通过resultMap自定义映射规则 -->
    <id column="tid" property="tid"/>
    <result column="tname" property="name"/>
    <collection property="studentList" ofType="Student">
    <!-- 通过使用collection来表示将所得所有结果合并为一个集合 -->
    <!-- tid相同的全部学生最后合并为List -->
        <id property="sid" column="sid"/>
        <result column="name" property="name"/>
        <result column="sex" property="sex"/>
    </collection>
</resultMap>
```
\
&emsp;&emsp;这种是一对多的情况，每个学生都有他对应的一个老师，现在在`Student`类中新增一个`Teacher`对象：
```java
@Data
@Accessors(chain = true)
public class Student {
    private int sid;
    private String name;
    private String sex;
    private Teacher teacher; // 每个学生都对应一个老师，但有很多学生
}
```
&emsp;&emsp;这是多对一的情况，同样可以通过`resultMap`实现：
```xml
<select id="selectStudent" resultMap="test2">
    select *, teacher.name as tname from student 
    left join teach on student.sid = teach.sid
    left join teacher on teach.tid = teacher.tid
</select>

<resultMap id="test2" type="Student">
    <id column="sid" property="sid"/>
    <result column="name" property="name"/>
    <result column="sex" property="sex"/>
    <association property="teacher" javaType="Teacher">
    <!-- 通过使用association来实现多对一，原理其实是一样的 -->
        <id column="tid" property="tid"/>
        <result column="tname" property="name"/>
    </association>
</resultMap>
```

### 事务操作
&emsp;&emsp;和JDBC一样，事务操作默认是自动提交的，在最开始提到的`MybatisUtil`类中有`getSession()`方法，传入的boolean值就代表着事务的自动提交：
```java
    public static SqlSession getSession(boolean autoCommit){ // 该方法用于获取新会话
        return sqlSessionFactory.openSession(autoCommit); // autoCommit为自动提交，关闭的话就会变为事务操作
    }
```
&emsp;&emsp;因此只要传入`false`，就关闭了事务的自动提交，这样在事务提交指令完成之前，对数据库的更改内容是不会进入到数据库当中的，包括回滚等事务操作和JDBC中是一样的。

## 动态SQL
&emsp;&emsp;以下内容来自官方文档：[动态SQL](https://mybatis.org/mybatis-3/zh/dynamic-sql.html)\
&emsp;&emsp;动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。

### if
&emsp;&emsp;使用动态 SQL 最常见情景是根据条件包含 where 子句的一部分。比如：
```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```
&emsp;&emsp;这条SQL语句提供了可选的查找功能，假如不传入`title`，那就会直接返回，如果传入`title`，会对`title`进行模糊查找，下面的`test`也是同理的。

### choose、when、otherwise
&emsp;&emsp;有时候，我们不想使用所有的条件，而只是想从多个条件中选择一个使用。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。
```xml
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose> <!-- switch -->
    <when test="title != null"> <!-- case -->
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise> <!-- default -->
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

### trim、where、set
&emsp;&emsp;回到一开始的示例，如果将`state = 'ACTIVE'`设置为动态条件：
```xml
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG
  WHERE
  <if test="state != null">
    state = #{state}
  </if>
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```
&emsp;&emsp;可以想到，如果`if`条件都不满足，最终SQL语句会变成这样：
```sql
SELECT * FROM BLOG
WHERE
```
&emsp;&emsp;毫无疑问会报错，如果第一个`if`条件不满足，但是第二个满足，SQL语句会变成这样：
```sql
SELECT * FROM BLOG
WHERE
AND title like ‘someTitle’
```
&emsp;&emsp;同样也是错误的。\
&emsp;&emsp;对此MyBatis有一个简单且高效的办法：
```xml
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```
&emsp;&emsp;`where`元素只会在子元素返回任何内容的情况下才插入`WHERE`子句。而且，若子句的开头为`AND`或`OR`，`where`元素也会将它们去除。\
\
&emsp;&emsp;如果`where`元素与你期望的不太一样，你也可以通过自定义`trim`元素来定制 `where`元素的功能：
```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
<!-- prefixOverrides属性会忽略通过管道符分隔的文本序列（注意此例中的空格是必要的）。-->
<!-- 这里会移除所有prefixOverrides属性中指定的内容，并且插入prefix属性中指定的内容。 -->
  ...
</trim>
```
\
&emsp;&emsp;用于动态更新语句的类似解决方案叫做`set`。`set`元素可以用于动态包含需要更新的列，忽略其它不更新的列。比如：
```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
      <!-- set会删除掉额外的逗号 -->
    </set>
  where id=#{id}
</update>
```

### foreach
&emsp;&emsp;动态 SQL 的另一个常见使用场景是对集合进行遍历（尤其是在构建`IN`条件语句的时候）。比如：
```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT * FROM POST P
  <where>
    <foreach item="item" index="index" collection="list"
        open="ID in (" separator="," close=")" nullable="true">
          #{item}
    </foreach>
  </where>
</select>
```
&emsp;&emsp;foreach 元素的功能非常强大，它允许你指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头与结尾的字符串以及集合项迭代之间的分隔符。这个元素也不会错误地添加多余的分隔符。\
&emsp;&emsp;你可以将任何可迭代对象（如`List`、`Set`等）、`Map`对象或者数组对象作为集合参数传递给 foreach。
* 当使用可迭代对象或者数组时，`index`是当前迭代的序号，`item`的值是本次迭代获取到的元素。
* 当使用`Map`对象（或者`Map.Entry`对象的集合）时，`index`是键，`item`是值。

## 使用注解开发
&emsp;&emsp;在前面的例子中，我们只需要编写对应的映射器，并将其绑定到一个接口上，即可直接通过该接口执行我们的SQL语句，极大的简化了我们之前JDBC那样的代码编写模式。\
&emsp;&emsp;还有一种更加简单方便的方式，无需xml映射器配置，而是直接使用注解在接口上进行配置。\
&emsp;&emsp;这是现在一种常用的形式，但由于Java 注解的表达能力和灵活性十分有限，对于xml的高灵活性来说稍有不足，不过在大部分场景下，直接使用注解开发已经绰绰有余了。\
\
&emsp;&emsp;在前面，一般是在xml文件中定义映射规则和SQL语句，再将其绑定到接口上：
```xml
<insert id="addStudent">
    insert into student(name, sex) values(#{name}, #{sex})
</insert>
```
```java
int addStudent(Student student);
```
&emsp;&emsp;如果改为注解实现，就只用在每个操作之前写上对应的注解：
```java
@Insert("insert into student(name, sex) values(#{name}, #{sex})")
int addStudent(Student student);
```
&emsp;&emsp;当然这先需要修改一下配置文件中的Mapper标签：
```xml
<mappers>
    <mapper class="com.test.mapper.MyMapper"/>
    <!-- 直接指定class -->
    <!-- 也可以直接注册整个包下的 <package name="com.test.mapper"/> -->
</mappers>
```
\
&emsp;&emsp;此时可以直接通过`@Results`注解进行自定义映射规则：
```java
@Results({
        @Result(id = true, column = "sid", property = "id"),
        @Resule(...)
})
//@Results注解的value是一个@Result数组，每个@Result都是一个单独的字段配置
@Select("select * from student")
List<Student> getAllStudent();
```
&emsp;&emsp;关于多表联查，可以用上`@Many`注解和`@One`注解，它们分别代表着一对多和多对一的关系，就像之前的`collection`和`assocation`：
```java
@Results({
        @Result(id = true, column = "tid", property = "tid"),
        @Result(column = "name", property = "name"),
        @Result(column = "tid", property = "studentList", many =
            @Many(select = "getStudentByTid")
        )
})
@Select("select * from teacher where tid = #{tid}")
Teacher getTeacherBySid(int tid);

@Select("select * from student inner join teach on student.sid = teach.sid where tid = #{tid}")
List<Student> getStudentByTid(int tid);
```
```java
@Results({
        @Result(id = true, column = "sid", property = "sid"),
        @Result(column = "sex", property = "name"),
        @Result(column = "name", property = "sex"),
        @Result(column = "sid", property = "teacher", one =
            @One(select = "getTeacherBySid")
        )
})
@Select("select * from student")
List<Student> getAllStudent();
```
&emsp;&emsp;注解和xml文件是可以混合使用的，比如说用注解编写SQL语句，映射规则用XML来实现，这时就可以使用`@ResultMap`注解直接指定id：
```java
@ResultMap("test")
@Select("select * from student")
List<Student> getAllStudent();
```

## MyBatis的缓存机制
&emsp;&emsp;MyBatis内置了一个强大的事务性查询缓存机制，我们查询时，如果缓存中存在数据，那就可以直接从缓存中获取，而不是去向数据库请求。\
&emsp;&emsp;MyBatis的缓存分为一级缓存和二级缓存，一级缓存也叫本地缓存，它默认会启用，并且不能关闭。一级缓存存在于SqlSession的生命周期中，即它是SqlSession级别的缓存。\
&emsp;&emsp;在同一个 SqlSession 中查询时，MyBatis 会把执行的方法和参数通过算法生成缓存的键值，将键值和查询结果存入一个Map对象中。如果同一个SqlSession 中执行的方法和参数完全一致，那么通过算法会生成相同的键值，当Map 缓存对象中己经存在该键值时，则会返回缓存中的对象。
&emsp;&emsp;比如说执行以下代码：
```java
public static void main(String[] args) throws InterruptedException {
    try (SqlSession sqlSession = MybatisUtil.getSession(true)){
        TestMapper testMapper = sqlSession.getMapper(TestMapper.class);
        Student student1 = testMapper.getStudentBySid(1);
        Student student2 = testMapper.getStudentBySid(1);
        System.out.println(student1 == student2);
    }
}
```
&emsp;&emsp;可以发现两次得到的是同一个Student对象，也就是说第二次查询并没有去创建对象，而是直接得到之前创建的对象，这就是缓存的应用。\
&emsp;&emsp;当然，如果在两次查询之间修改数据，那缓存就没有生效了，就会得到新创建的对象。除此之外，一级缓存只针对单个会话，如果当前会话关闭，也会清理缓存，多个会话的缓存不互通。比如说执行以下代码：
```java
public static void main(String[] args) {
    try (SqlSession sqlSession = MybatisUtil.getSession(true)){
        TestMapper testMapper = sqlSession.getMapper(TestMapper.class);

        Student student2;
        try(SqlSession sqlSession2 = MybatisUtil.getSession(true)){
            TestMapper testMapper2 = sqlSession2.getMapper(TestMapper.class);
            student2 = testMapper2.getStudentBySid(1);
        }

        Student student1 = testMapper.getStudentBySid(1);
        System.out.println(student1 == student2);
    }
}
```
\
&emsp;&emsp;一级缓存给我们提供了很高速的访问效率，但是作用范围有限，如果希望缓存能够扩展到所有会话都能使用，可以通过二级缓存来实现，二级缓存默认是关闭状态，要开启二级缓存，我们需要在映射器XML文件中添加：
```xml
<cache/>
```
&emsp;&emsp;二级缓存存在于SqlSessionFactory 的生命周期中，即它是SqlSessionFactory级别的缓存。可以根据官方文档对其进行一些配置：
```xml
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
<!-- 
    eviction:先出先出缓存
    flushInterval:每隔60s刷新
    size:存储对象或列表的512个引用
    readOnly:返回对象是只读的
-->
```
&emsp;&emsp;二级缓存具有如下效果：
* 映射语句文件中的所有SELECT 语句将会被缓存。
* 映射语句文件中的所有时INSERT 、UPDATE 、DELETE 语句会刷新缓存，但可以通过`flushCache="false"`来关闭。
* 缓存默认使用`LRU算法`来收回。\
对于查找数据来说，读取顺序：二级缓存$\geq$一级缓存$\geq$数据库