---
title: "JAVA - 博客项目实战"
date: 2023-01-15T13:05:23+08:00
categories:
    - Java
tags:
    - 项目
draft: false
---

## 项目介绍

​	这几个月的学习路线是Java SE -> MySQL -> Lombok -> JDBC -> MyBatis -> JUnit -> Maven -> Java Web -> Spring -> Spring MVC -> Spring Security -> Spring Boot -> Git -> Redis -> JPA -> Swagger，同时学校这个学期还开了Linux和前端课，前端课除了三件套以外主要学了Vue框架，我为了Vue3特地补了一下TypeScript，现在做一个前后端分离的博客项目，但主要写后端代码，前端代码只做小幅修改。

​	这篇博客主要逐步记录开发的过程与遇到的BUG记录以及如何解决。

​	后端主要技术栈：

* SpringBoot
* SpringSecurity
* MyBatisPlus
* Redis
* EasyExcel
* Swagger2

​	其中MyBatisPlus没有系统学过，根据MyBatis的经验边看官方文档边写。EasyExcel同样跟着官方文档写。

​	前端主要技术栈：

* Vue
* ElementUI
* Echarts

​	

​	博客项目的后端包括博客后端和管理系统后端。

## 项目初始化 1.15

### 创建多模块项目

​	项目分为父模块Blog，博客子模块blog，管理系统子模块admin，公共模块framework。

​	项目初始目录结构如下：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301152202818.png)

### 接口响应 1.15

​	建立响应枚举类，包括请求的返回类型。

​	建立ReponseResult，集体统一响应，符合接口开发规范。

## 博客后端

### 配置文件

​	在blog子模块下编写application.yml配置文件。

```yaml
server:
  port: 7777

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/blog?characterEncoding=utf-8
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
  servlet:
    multipart:
      max-file-size: 2MB
      max-request-size: 5MB
  main:
    allow-circular-references: true

mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      logic-delete-field: delFlag
      logic-delete-value: 1
      logic-not-delete-value: 0
      id-type: auto
```

### 热门文章列表 1.16

#### 数据表

​	建立一个文章表article，共有以下字段：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301152223452.png)

#### 需求

​	查询访问量最高的十篇文章。

​	以访问量降序展示文章标题与访问量，并能点击标题跳转文章详情。

#### 实现

​	为解决跨域问题创建WebConfig继承WebMvcConfigurer类，允许跨域请求。

​	展示热门的文章列表只需要文章表中的三个字段：id, title, view_count，其它的字段是不需要的，因此这里建立VO。

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class HotArticleVo {
    private Long id;
    private String title;
    private Long viewCount;
}
```

​	将文章表中的所有内容都取出来，将指定字段复制到VO中。

```java
@Override
public ResponseResult<?> hotArticleList() {
    // 查询访问量前十的文章，封装成ResponseResult

    LambdaQueryWrapper<Article> queryWrapper = new LambdaQueryWrapper<>();
    // 展示正式文章，不能是草稿
    queryWrapper.eq(Article::getStatus, SystemConstants.ARTICLE_STATUS_NORMAL);
    // 访问量降序排序
    queryWrapper.orderByDesc(Article::getViewCount);
    // 分页，一页显示十条
    Page<Article> page = new Page<>(SystemConstants.HOT_ARTICLE_PAGES, SystemConstants.HOT_ARTICLE_PAGE_SIZE);
    page(page, queryWrapper);

    // 将文章实体映射
    List<Article> articleList = page.getRecords();
    // bean copy
    List<HotArticleVo> hotArticleVoList = BeanCopyUtils.copyBeanList(articleList, HotArticleVo.class);
    // 封装返回
    return ResponseResult.okResult(hotArticleVoList);
}
```

​	其中BeanCopyUtils为自建的工具类，里面定义了两个方法，一个是根据传入的字节码类型创建VO对象，并通过反射获取对象实现对象的复制；一个是通过流将复制后的对象封装成顺序表。

```java
public class BeanCopyUtils {

    private BeanCopyUtils() {

    }

    public static <T> T copyBean(Object source, Class<T> clazz) throws InstantiationException, IllegalAccessException {
        // 创建目标对象
        T obj = null;
        try {
            // 通过反射获取
            obj = clazz.newInstance();
            // 实现copy
            BeanUtils.copyProperties(source, obj);
        } catch (Exception e) {
            e.printStackTrace();
            throw e;
        }
        return obj;
    }

    public static <T> List<T> copyBeanList(List<?> list, Class<T> clazz) {
        return list.stream()
                .map(o -> {
                    try {
                        return copyBean(o, clazz);
                    } catch (InstantiationException | IllegalAccessException e) {
                        throw new RuntimeException(e);
                    }
                })
                .collect(Collectors.toList());
    }
}
```

#### 遇到的问题

​	VO和工具类都是第一次写，还有字面值特别定义一个常量来写，这种规范的写法暂时有点不适应，虽然有利于后续维护但一个人写起来有点累。

​	写工具类的时候泛型写的有点吃力，需要回去补一下基础。

​	第一次用postman测试后端接口。

​	目前遇到最大的一个问题是项目的配置问题，每次重启IDE后，blog子模块的java包都会被标记为普通包，需要手动将其置为源代码根目录，然后需要将父模块和子模块重新install一遍，不然会出现依赖不加载问题。推测问题出现在SpringBoot的配置文件当中，暂时找不到debug的方法。（1.16）

### 文章分类 1.17

#### 数据表

​	建立文章分类表category，共有以下字段：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301171218461.png)

​	pid为父id，为某分类的父分类标识，初始值为-1。

#### 需求

​	页面上展示分类列表，点击分类后展示分类下的所有文章。

​	分类本身必须是正常状态，不能被标识为禁用，并且只展示已发布文章。

#### 实现

​	article表中有分类id字段category_id，category表中有分类名name，根据阿里的代码规范尽量不使用多表联查，因此这里使用单表查询查两次，先查找文章对应的分类号，再去分类表中找分类对应的分类名。

​	然而发现MyBatisPlus不用写SQL语句反而使逻辑写的有点复杂，这本身是一句SQL就能解决的事情。我感觉可能我写完MyBatisPLus的项目以后得回去补一个MyBatis的项目。

```java
@Override
public ResponseResult<?> getCategoryList() {
    // 查询文章表

    LambdaQueryWrapper<Article> queryWrapper = new LambdaQueryWrapper<>();
    // 文章状态为已发布
    queryWrapper.eq(Article::getStatus, SystemConstants.ARTICLE_STATUS_NORMAL);
    List<Article> articleList = articleService.list(queryWrapper);
    // 利用集合Set去重分类id
    Set<Long> categoryIdList = articleList.stream()
        .map(Article::getCategoryId)
        .collect(Collectors.toSet());

    // 查询分类表

    List<Category> categoryList = listByIds(categoryIdList);
    // 筛选掉状态为禁用的分类，只留状态正常的分类
    categoryList = categoryList.stream()
        .filter(category -> SystemConstants.CATEGORY_STATUS_NORMAL.equals(category.getStatus()))
        .collect(Collectors.toList());

    // bean copy
    List<CategoryVo> categoryVoList = BeanCopyUtils.copyBeanList(categoryList, CategoryVo.class);
    // 封装返回
    return ResponseResult.okResult(categoryVoList);
}
```



