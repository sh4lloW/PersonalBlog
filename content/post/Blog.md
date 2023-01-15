---
title: "JAVA - 博客项目实战"
date: 2023-01-15T13:05:23+08:00
categories:
    - Java
tags:
    - 项目
draft: false
---

# 项目介绍

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

# 项目初始化 1.15

​	项目分为父模块Blog，博客子模块blog，管理系统子模块admin，公共模块framework。

​	项目初始目录结构如下：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301152202818.png)

# 博客后端

## 热门文章列表 1.15

### 数据表

​	建立一个文章表article，共有以下字段：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301152223452.png)

### 需求

​	查询访问量最高的十篇文章。

​	以访问量降序展示文章标题与访问量，并能点击标题跳转文章详情。





