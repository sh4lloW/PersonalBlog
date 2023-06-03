---
title: "MyBatis中的模糊查询与Oracle中的CONCAT"
date: 2023-06-03T17:26:16+08:00
categories:
    - SQL
tags:
    - MyBatis
draft: false
---

## MyBatis中使用模糊查询 

​		在做Oracle课设时使用到了模糊查询，但是在下面的模糊的查询中，`#{}`会被识别为字符串：

```sql
SELECT * FROM book WHERE name LIKE '%#{name}%'
```

​		正确的写法一：

```sql
SELECT * FROM book WHERE name LIKE '%${name}%'
```

​		但是这样的话就是字符串拼接，无法防止SQL注入，所以常见的写法是使用SQL的CONCAT函数来拼接：

```sql
SELECT * FROM book WHERE name LIKE CONCAT('%', #{name}, '%')
```

## Oracle中的CONCAT

​		在我自己写的项目中，这样写仍然会报错`ORA-00909: 参数个数无效 `，是因为Oracle中的CONCAT与MySQL中的不同，最多只允许两个参数，所以只能这里通过函数嵌套的方式实现：

```sql
SELECT * FROM book WHERE name LIKE CONCAT(CONCAT('%', #{name}), '%')
```



