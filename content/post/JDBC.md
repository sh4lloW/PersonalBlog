---
title: "JAVA - JDBC"
date: 2022-07-28T22:02:52+08:00
categories:
    - Java
tags:
    - Web
    - JDBC
draft: false
---

## JDBC是什么
&emsp;&emsp;JDBC的全称是Java Data Base Connectivity，是Java语言中用来规范客户端程序如何访问数据库的应用程序接口，简单来说就是Java与数据库间的桥梁，可以通过Java代码来操作数据库的CRUD等操作。

## 连接数据库
```java
public static void main(String[] args) {
        //前两步需要释放资源，故放在try()中
        try(Connection connection = DriverManager.getConnection("URL","user","password");   //通过DriverManager来实现与数据库的连接
            Statement statement = connection.createStatement())     //Statement对象用于执行SQL指令
        {
            ResultSet set = statement.executeQuery("SELECT * FROM Student");    //执行查询指令，得到结果集
            while(set.next())
            {
                System.out.println(set.getString(1));   //输出第一列
            }
        }catch(SQLException e)
        {
            e.printStackTrace();
        }
    }
```
&emsp;&emsp;其中，DriverManager类是用来管理数据库驱动的，它可以调用`getConnection()`方法来进行数据库的链接：
```java
public static Connection getConnection(String url,
        String user, String password) throws SQLException {
        java.util.Properties info = new java.util.Properties();

        if (user != null) {
            info.put("user", user);
        }
        if (password != null) {
            info.put("password", password);
        }

        return (getConnection(url, info, Reflection.getCallerClass()));
    }
```

## 利用Statement执行SQL语句
&emsp;&emsp;在上面的例子中使用了`executeQuery()`方法来执行select语句，查询得到的数据放在结果集ResultSet中，除此之外，还可以用`executeUpdate()`方法来执行一个DML或DDL语句，它会返回一个int类型的值，表示执行后受影响的行数，可以通过这个来判断是否执行成功。\
&emsp;&emsp;Statement还可以是通过`execute()`方法来执行任意SQL语句，它会返回一个boolean值来表示执行结果是ResultSet还是int，可以用`getResultSet()`或`getUpdateCount()`方法来获取结果集或受影响的行数。
### 执行DML操作
```java
int temp = statement.executeUpdate("INSERT INTO Student VALUES (4,'李四');");
System.out.println("受影响的行数为："+temp);
ResultSet set = statement.executeQuery("SELECT * FROM Student");    //查询整个学生表
while(set.next())   //打印查询结果
{
    int id = set.getInt(1);     //第一列
    String name = set.getString(2);     //第二列
    System.out.println("学号："+id+" "+"姓名："+name);
}
```
&emsp;&emsp;输出结果为：
```
受影响的行数为：1
学号：1 姓名：A
学号：2 姓名：B
学号：3 姓名：张三
学号：4 姓名：李四
```
### 批量操作
```java
public static void main(String[] args) {
        try(Connection connection = DriverManager.getConnection("URL","user","password");
            Statement statement = connection.createStatement())
        {
            statement.addBatch("INSERT INTO Student VALUES(5,'C');");
            statement.addBatch("INSERT INTO Student VALUES(6,'D');");
            statement.addBatch("INSERT INTO Student VALUES(7,'E');");
            statement.executeBatch();   //统一执行
        }catch(SQLException e)
        {
            e.printStackTrace();
        }
    }
```

## SQL注入攻击
### 实现登陆
&emsp;&emsp;如果要模拟登陆一个用户，可以这样写：
```java
public static void main(String[] args) {
    try(Connection connection = DriverManager.getConnection("URL","user","password");
        Statement statement = connection.createStatement();
        Scanner scanner = new Scanner(System.in))
    {
        ResultSet set = statement.executeQuery("SELECT * FROM user WHERE username='"+scanner.nextLine()+"'AND password='"+scanner.nextLine()+"';");
        //输入的账号密码是否与数据库内对应
        while(set.next())
        {
            System.out.println("登陆成功！");
        }
    }catch(SQLException e)
    {
        e.printStackTrace();
    }
}
```
&emsp;&emsp;这样似乎大概实现了用户登陆的功能，但如果尝试下面这种输入：
```
user1
wrongpwd' or 1=1; -- 
登陆成功！
```
&emsp;&emsp;发现即使密码不匹配同样可以登陆成功，这是因为SQL语句变成了：
```SQL
SELECT * FROM user WHERE username='user1'AND password='wrongpwd' or 1=1; --';
```
&emsp;&emsp;可以发现后面的内容已经被注释掉，而1=1破坏了WHERE的功能性，使得所有的账户都可以随意登陆。\
&emsp;&emsp;对此我们的解决方法可能是对用户的输入加一下限制条件，防止用户输入一些SQL语句的关键字，但关键字非常多，这样的办法是低效且解决不彻底的。
### 使用PreparedStatement
&emsp;&emsp;PreparedStatement类可以解决上述问题：
```java
public static void main(String[] args) {
    try(Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/test","root","zzydatabase3141");
        PreparedStatement statement = connection.prepareStatement("SELECT * FROM user WHERE username= ? and pwd=?;");
        Scanner scanner = new Scanner(System.in))
    {
        statement.setString(1, scanner.nextLine());
        statement.setString(2, scanner.nextLine());
        ResultSet set = statement.executeQuery();
        while(set.next())
        {
            System.out.println("登陆成功！");
        }
    }catch(SQLException e)
    {
        e.printStackTrace();
    }
}
```