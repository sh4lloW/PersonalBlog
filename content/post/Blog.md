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

​		这几个月的学习路线是Java SE -> MySQL -> Lombok -> JDBC -> MyBatis -> JUnit -> Maven -> Java Web -> Spring -> Spring MVC -> Spring Security -> Spring Boot -> Git -> Redis -> JPA -> Swagger，同时学校这个学期还开了Linux和前端课，前端课除了三件套以外主要学了Vue框架，我为了Vue3特地补了一下TypeScript，现在做一个前后端分离的博客项目，但主要写后端代码，前端代码只做小幅修改。

​		这篇博客主要逐步记录开发的过程与遇到的BUG记录以及如何解决。

​		后端主要技术栈：

* SpringBoot
* SpringSecurity
* MyBatisPlus
* Redis
* EasyExcel
* Swagger2

​		其中MyBatisPlus没有系统学过，根据MyBatis的经验边看官方文档边写。EasyExcel同样跟着官方文档写。

​		前端主要技术栈：

* Vue
* ElementUI
* Echarts

​	

​		博客项目的后端包括博客后端和管理系统后端。

## 项目初始化 1.15

### 创建多模块项目

​		项目分为父模块Blog，博客子模块blog，管理系统子模块admin，公共模块framework。

​		项目初始目录结构如下：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301152202818.png)

### 接口响应 1.15

​		建立响应枚举类，包括请求的返回类型。

​		建立ReponseResult，集体统一响应，符合接口开发规范。

## 博客后端

### 配置文件

​		在blog子模块下编写application.yml配置文件。

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

​		建立一个文章表article，共有以下字段：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301152223452.png)

#### 需求

​		查询访问量最高的十篇文章。

​		以访问量降序展示文章标题与访问量，并能点击标题跳转文章详情。

#### 实现

​		为解决跨域问题创建WebConfig继承WebMvcConfigurer类，允许跨域请求。

​		展示热门的文章列表只需要文章表中的三个字段：id, title, view_count，其它的字段是不需要的，因此这里建立VO。

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

​		将文章表中的所有内容都取出来，将指定字段复制到VO中。

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

​		其中BeanCopyUtils为自建的工具类，里面定义了两个方法，一个是根据传入的字节码类型创建VO对象，并通过反射获取对象实现对象的复制；一个是通过流将复制后的对象封装成顺序表。

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

​		VO和工具类都是第一次写，还有字面值特别定义一个常量来写，这种规范的写法暂时有点不适应，虽然有利于后续维护但一个人写起来有点累。

​		写工具类的时候泛型写的有点吃力，需要回去补一下基础。

​		第一次用postman测试后端接口。

​		目前遇到最大的一个问题是项目的配置问题，每次重启IDE后，blog子模块的java包都会被标记为普通包，需要手动将其置为源代码根目录，然后需要将父模块和子模块重新install一遍，不然会出现依赖不加载问题。推测问题出现在SpringBoot的配置文件当中，暂时找不到debug的方法。（1.16）将项目配置保存为项目文件后这个问题暂时消失了。（1.18）

### 文章分类 1.17

#### 数据表

​		建立文章分类表category，共有以下字段：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301171218461.png)

​		pid为父id，为某分类的父分类标识，初始值为-1。

#### 需求

​		页面上展示分类列表，点击分类后展示分类下的所有文章。

​		分类本身必须是正常状态，不能被标识为禁用，并且只展示已发布文章。

#### 实现

​		article表中有分类id字段category_id，category表中有分类名name，根据阿里的代码规范尽量不使用多表联查，因此这里使用单表查询查两次，先查找文章对应的分类号，再去分类表中找分类对应的分类名。

​		然而发现MyBatisPlus不用写SQL语句反而使逻辑写的有点复杂，这本身是一句SQL就能解决的事情。我感觉可能我写完MyBatisPLus的项目以后得回去补一个MyBatis的项目。

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

#### 遇到的问题

​		中途和重启IDE后都有出现找不到主类BlogApplication.java的问题，解决的方式是在maven管理中先父模块install，再将blog子模块install。

### 文章列表展示与分页 1.18

#### 需求

​		博客首页和分类页都显示文章列表，这个文章列表需要分页显示。

​		文章列表只显示已发布的文章，不显示草稿，并且文章表中is_top字段为1也就是置顶的文章必须显示在上面。

#### 实现

​		文章列表只需要查询文章表就可以了，注意和前端传入的字段进行对接。

​		主要有四个问题需要解决：

* 显示已发布的文章
* 文章置顶
* 分页

​		解决的方法分别是：

* 直接查询文章状态
* 查询is_top字段，将其降序排序，这样置顶的就会优先展示（is_top为1才是置顶，0为非置顶）
* 和热门文章列表接口的一样，直接分页，分页的大小由前端请求的参数指定

```java
@Override
public ResponseResult<?> articleList(Integer pageNum, Integer pageSize, Long categoryId) {
    // 展示文章列表并分页展示，需要使用分页查询

    LambdaQueryWrapper<Article> queryWrapper = new LambdaQueryWrapper<>();
    // 需要对categoryId进行判断，因为前端可能是分类后查询也有可能是首页所有的文章列表查询
    if (Objects.nonNull(categoryId) && categoryId > 0) {    // 分类不为null也大于0
        queryWrapper.eq(Article::getCategoryId, categoryId);
    }
    // 状态是已发布的文章，并且判断文章是否置顶，这里使用的方式是对文章表中的is_top字段降序排序
    queryWrapper.eq(Article::getStatus, SystemConstants.ARTICLE_STATUS_NORMAL);
    queryWrapper.orderByDesc(Article::getIsTop);
    // 分页查询
    Page<Article> page = new Page<>(pageNum, pageSize);
    page(page, queryWrapper);

    // 返回的是分类名而不是分类Id，用categoryId去查询categoryName
    List<Article> articleList = page.getRecords();
    for (Article article : articleList) {
        Category category = categoryService.getById(article.getCategoryId());
        article.setCategoryName(category.getName());
    }

    // bean copy
    List<ArticleListVo> articleListVoList = BeanCopyUtils.copyBeanList(page.getRecords(), ArticleListVo.class);

    // 根据前端请求字段，需要进行第二次封装，详见ArticleListPageVo
    ArticleListPageVo articleListPageVo = new ArticleListPageVo(articleListVoList, page.getTotal());

    // 封装返回
    return ResponseResult.okResult(articleListPageVo);
}
```

​		这里还涉及到两个问题，一个是需要返回分类名字，而这个名字在分类表而不是文章表，所以需要再一次进行查询。另外一个是时间的传输格式问题，实体类时间使用的是Date类型，需要在变量上使用注解`@JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")`。

#### 遇到的问题

​		今天暂时解决了根目录源代码的设置问题，但出现了新的问题就是我重启IDE后项目结构直接变得面目全非，父模块和blog子模块直接不见了，直接手动重新导入模块，不知道是不是IDEA的抽风问题，我是不是该降个版本。

​		还有，再次重申每次运行项目前记得重新install父模块，没有install的话暂时出现过以下问题：

* 启动文件BlogApplication找不到
* 新写的接口不生效，返回404

### 文章详情 1.18

#### 需求

​		首页及分类的文章可以点击进入文章详情，在文章详情页面显示文章。

#### 实现

​		文章根据id定位，根据文章的id去查询文章的详情。

```java
@SneakyThrows
@Override
public ResponseResult<?> articleDetail(Long id) {
    // 根据id查询文章

    Article article = getById(id);
    // bean copy
    ArticleDetailVo articleDetailVo = BeanCopyUtils.copyBean(article, ArticleDetailVo.class);
    // 根据分类id查询分类名
    Long categoryId = article.getCategoryId();
    Category category = categoryService.getById(categoryId);
    if (category != null) {
        articleDetailVo.setCategoryName(category.getName());
    }

    // 封装返回
    return ResponseResult.okResult(articleDetailVo);
}
```

#### 遇到的问题

​		无。



