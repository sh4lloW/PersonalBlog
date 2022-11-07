---
title: "JAVA - Maven"
date: 2022-08-20T11:58:19+08:00
categories:
    - Java
tags:
    - Web
draft: false
---

## Maven是什么
&emsp;&emsp;Maven 是最流行的 Java 项目构建系统，Maven项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。\
&emsp;&emsp;Maven可以帮助我们：
* 项目的自动构建，包括代码的编译、测试、打包、安装、部署等操作。
* 依赖管理，项目使用到哪些依赖，可以快速完成导入。
如果一个一个导入jar包作为依赖，无疑是十分繁琐的，更何况一个依赖可能还有前置依赖，比如JUnit。使用Maven就可以集中管理依赖，更加方便快捷。

## Maven项目结构
&emsp;&emsp;Maven的项目结构十分清晰：

![maven.png](http://tva1.sinaimg.cn/large/008kE3f3gy1h7wmbu3s18j30q20gymz5.jpg)

​		可以看到，Maven自动构建了一套项目结构，将测试与主项目文件分开，并对配置文件等进行了分类，Maven给与了清晰的项目结构，开发者可以用Maven进行快捷的开发。

## POM文件的配置

​		`pom.xml`相当于Maven的配置文件，一个基本的pom文件长这样：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>BookManagementSystem</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

</project>
```

​		pom文件以`project`为根节点，`modelVersion`定义了当前模型的版本，一般为4.0。

​		<groupId>,<artifactId>,<version>三个子标签一起用于区别每一个项目，通过这三个子标签来定位项目，可以看作是一个项目的坐标，其中：

* <groupId>用于指定组名称，命名规则一般与包名一致，一个组下可以有很多个项目

* <artifactId>一般用于指定项目在当前组中的唯一名称，可以在组中与其他项目做区分

* <version>代表项目的版本，可以手动指定项目的版本号，比如上面的代码说明了该项目是1.0版本，`SNAPSHOT`表示这是一个开发中的项目

  <properties>中的是一些变量与选项的配置，这里标注了JDK的版本为17。

## Maven依赖的导入与管理

​		现在可以使用Maven来导入依赖，首先创建一个`dependencies`结点：

```xml
<dependencies>
    //里面填写所有依赖
</dependencies>
```

​		在<dependencies>中填入所有依赖，可以在https://mvnrepository.com/中查找某个依赖的坐标，比如这里导入`Lombok`依赖：

```xml
<dependency>
	<groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.24</version>
    <scope>provided</scope> <!-- 作用域 -->
</dependency>
```

​		刷新pom文件的配置后，就可以直接在项目中使用依赖了。



​		依赖一开始存储于中央仓库中，Maven通过依赖的坐标找到位置并下载到本地：

![图片](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimage.bubuko.com%2Finfo%2F201901%2F20190106202802893827.png&refer=http%3A%2F%2Fimage.bubuko.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1639624645&t=75fdf146baa915fbba88918895f92b81)

​		除了中央仓库，有的依赖会存储于一些远程仓库中（相当于私服），在中央仓库或远程仓库下载以后，会在本地创建一个`.m2`文件夹，这就是Maven本地仓库的文件夹，默认建立在用户的根目录下（`C:\Users\Admin\.m2`）。

​		在下次导入依赖时，会先从本地的仓库中查找，如果已经存在，就不用再从中央仓库中联网下载了。

## 依赖的属性

​		除了三个基本的属性以外，依赖还可以添加以下的属性：

* **type**：依赖的类型，对于项目坐标定义的packaging。大部分情况下，该属性不必声明，默认为jar
* **scope**：依赖的作用域，也就是范围
* **optional**：标记依赖是否可选
* **exclusions**：用来排除传递性依赖，一个项目可能依赖于另一个项目，比如Junit，如果要使用，就必须下载这个项目的依赖，这就是传递性依赖。

### 依赖的作用域

​		**scope**属性决定了依赖的作用域范围：

* **compile**：默认范围，在编译、运行、测试时都有效。如果定义依赖关系时没有指定作用域，则默认为compile。
* **provided**：在编译、测试时有效，在运行时无效。比如Lombok，在编译阶段需要它，编译完成后目标代码已生成，在运行阶段就不需要Lombok的存在了。
* **runtime**：在运行、测试时有效，在编译时无效。
* **test**：只在测试时有效。比如JUnit只在测试阶段生效。
* **system**：与provided一样，不过不是从远程仓库获取，而是直接导入本地jar包，一般用于依赖没有上传的远程仓库，只有一个jar。

### 可选依赖

​		当项目中的某些依赖不希望被使用此项目作为依赖的项目使用时，我们可以给依赖添加`optional`标签表示此依赖是可选的，默认在导入依赖时，不会导入可选的依赖：

```xml
<optional>true</optional>
```

​		比如MyBatis的pom文件中就存在着大量的可选依赖：

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.30</version>
  <optional>true</optional>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.7.30</version>
  <optional>true</optional>
</dependency>
<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
  <version>1.2.17</version>
  <optional>true</optional>
</dependency>
 ...
```

​		MyBatis支持多种类型的日志，需要用到多种不同的日志框架，因此需要导入这些依赖做兼容，但项目中不一定会使用这些框架作为日志打印器，所以他们是可选的。

### 排除依赖

​		如果存在不是可选依赖，但不希望被导入的项目使用此依赖，就可以使用排除依赖来防止添加不必要的依赖：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

​		排除依赖可以有效节约内存空间，也可以避免许可证等问题。

## Maven的继承

​		一个Maven项目可以继承自另一个Maven项目，比如多个子项目都需要父项目的依赖，就可以使用继承关系来快速配置。

​		通过对项目创建新模块来创建子项目，子项目的pom文件会自带`parent`结点，用于表示该Maven项目是父Maven项目的子项目：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>FatherMaven</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>child</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

</project>
```

​		子项目会直接继承父项目的所有依赖，可选依赖除外。

​		如果希望子项目只需要部分父项目的依赖即可，可以对父项目的依赖进行统一管理，在`dependencies`外面套上`dependencyManagement`结点：

```xml
<dependencyManagement>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
            <scope>provided</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.9.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    ...
    
</dependencyManagement>
```

​		这样做以后，子项目原本可以正常使用的依赖会全部失效，只需要在取出需要使用的依赖就可以了：

```xml
<dependencies>
        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
	<dependency>
    	<groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.24</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

​		**如果父项目既有`dependencies`也有`dependencyManagement`结点，那么子项目仍会自动继承`dependencies`中的依赖。**

## Maven的生命周期

​		Maven的生命周期就是对所有的构建过程进行抽象和统一。包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等几乎所有的构建步骤。

​		在IDEA的右上角Maven板块中，每个Maven都有一个生命周期，实际就是Maven的一些插件，每个插件都有相应的功能：

* **clean**：执行后会清空`target`文件夹，可以解决一些缓存没更新的问题
* **validate**：验证项目可用性
* **compile**：将项目编译为class文件
* **install**：将当前项目安装到本地仓库，以供其他项目作为依赖使用
* **test**：测试所有位于test目录下的测试案例
* **package**：对项目进行打包，生成jar文件
* **verify**：按顺序执行每个默认生命周期阶段（`validate`, `compile`, `package`等）
* **deploy**：发布项目到本地仓库和远程仓库
* **site**：生成项目站点文档