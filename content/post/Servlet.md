---
title: "JAVA - Servlet"
date: 2022-08-28T11:33:18+08:00
categories:
    - Java
tags:
    - Web
draft: false
---

## Servlet简介

​		Servlet是Java Servlet Applet的简称，是运行在服务器上的一个小程序，用来处理服务器接收到的请求。

​		对于一般的网页程序都是通过浏览器的访问实现的，浏览器发出访问请求，服务器接收并处理请求，这就是常见的B/S模型（浏览器/服务器模型）。Servlet就是对请求做出处理的组件。

​		狭义上来说，Servlet是Java的一个接口，广义上来说，是指任何实现了这个接口的子类。一般常说的Servlet都是指后者，比如后文将提到的常用的HttpServlet。

## Servlet的创建

### Servlet配置

​		首先在maven配置文件`pom.xml`中导入Servlet依赖：

```xml
<dependency>
	<groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>5.0.0</version>
    <scope>provided</scope>
</dependency>
```

#### 配置web.xml文件

​		在用Maven创建的JavaWeb项目中，在`/src/main/webapp/WEB-INF`中，有一个叫做`web.xml`的文件，它是项目中的一个配置文件，主要用于配置首页、Servlet、Filter、Listener等等。

​		在`web.xml`中注册，就可以在类中使用Serlvet了：

```xml
<servlet>
    <servlet-name>test</servlet-name>	<!-- Servlet的名字 -->
    <servlet-class>com.example.webtest.TestServlet</servlet-class>	<!-- Servlet的路径 -->
</servlet>
<servlet-mapping>	<!-- 浏览器访问Servlet的映射地址 -->
    <servlet-name>test</servlet-name>
    <url-pattern>/test</url-pattern>
</servlet-mapping>
```

​		如果不进行配置，其实Tomcat也自带了一些默认的Servlet，编写在conf目录下的web.xml中：

```xml
<!-- The mapping for the default servlet -->
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- The mappings for the JSP servlet -->
    <servlet-mapping>
        <servlet-name>jsp</servlet-name>
        <url-pattern>*.jsp</url-pattern>
        <url-pattern>*.jspx</url-pattern>
    </servlet-mapping>
```

#### 使用注解配置Servlet

​		相比于配置`web.xml`文件，使用注解配置Servlet更加方便快捷：

```java
@WebServlet("/test")
```

​		注解配置无疑更加简洁，它也是现在更加常用的配置方式。

​		（配置`web.xml`文件与注解配置选其一，避免混淆）

### Servlet创建

​		Servlet的创建非常简单，只需要实现Servlet接口即可：

```java
@WebServlet("/test")
public class TestServlet implements Servlet {
  
}
```

​		但相比于直接实现Servlet接口，更常用的方式是继承`HttpServlet`类

​		Servlet有一个直接的抽象类`GenericServlet`，这个类仅仅完善了一个Servlet的基本操作，比如配置文件的读取等等，没有实现service方法，service方法就是用来处理客户端的请求（下面生命周期会有）。

​		对此，`GenericServlet`有一个子类`HttpServlet`，它同样是一个抽象类，它根据http规则，完善了service方法：

```java
// HttpServlet源码
public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
	if (req instanceof HttpServletRequest && res instanceof HttpServletResponse) {
    	HttpServletRequest request = (HttpServletRequest)req;
        HttpServletResponse response = (HttpServletResponse)res;
        this.service(request, response);
    } else {
        throw new ServletException("non-HTTP request or response");
    }
}
```

​		service方法中有两个参数，`ServletRequest`和`ServletResponse`，用户发起的http请求被Tomcat服务器封装为了一个`ServletRequest`对象，我们得到的是Tomcat服务器帮我们创建的一个实现类，http请求报文中所有的内容都可以从该对象中获取。`ServletResponse`就是我们需要返回给浏览器的http响应报文实体类的封装。

​		通过源码可以发现，`HttpServlet`的实现已经比较完善，它帮我们提前实现了一些操作，所以可以直接继承该类来编写Servlet：

```java
@WebServlet("/test")
public class TestServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");		// 响应类型及编码格式
        resp.getWriter().write("在使用Servlet");
    }
}
```

​		一般我们需要重写`HttpServlet`的doGet()或doPost()方法，在里面处理客户端的请求。

## Servlet的生命周期

​		Servlet的生命周期代表一个Servlet是怎么运行的，可以通过实现Servlet接口的方式来观察Servlet的生命周期：

```java
public class TestServlet implements Servlet {

    public TestServlet(){	// 构造方法
        System.out.println("构造方法");
    }

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {		// Servlet初始化
        System.out.println("init");
    }

    @Override
    public ServletConfig getServletConfig() {
        System.out.println("getServletConfig");
        return null;
    }

    @Override	// Servlet处理请求数据的方法
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("service");
    }

    @Override
    public String getServletInfo() {
        System.out.println("getServletInfo");
        return null;
    }

    @Override
    public void destroy() {		// Servlet销毁
        System.out.println("destroy");
    }
}
```

​		启动服务器后访问定义的页面，然后关闭服务器，得到的输出为：

```
构造方法
init
service

destroy
```

​		尝试多次访问页面，可以发现init方法和构造方法都只会执行一次，而每次访问都会执行一次service方法，在关闭服务器后，会执行destroy方法，因此可以得到一个Servlet的生命周期为：

* 执行构造方法完成Servlet的实例化
* 调用init()方法完成初始化
* 调用service()方法处理客户端请求
* 销毁前调用destroy()方法
* （Servlet由JVM的垃圾回收器进行垃圾回收）



​		Servlet默认在第一次访问时才加载，但可以通过`load-on-startup`来控制Servlet在服务器启动时就加载（实例化+初始化）。

​		`load-on-startup`可以通过`web.xml`文件配置：

```xml
<servlet>
    <servlet-name>test</servlet-name>
    <servlet-class>com.example.webtest.TestServlet</servlet-class>
    <load-on-startup>1</load-on-startup>	<!-- 控制自动加载的优先级 -->
</servlet>
```

​		同样，更通常使用注解来配置：

```java
@WebServlet(value = "/test", loadOnStartup = 1)
```

​		加载标签/属性中的数字表示加载的优先级，默认为-1。数字若为负数表示启动时不加载，第一次访问时才加载。正数表示服务器启动时就加载，从0开始，数字越小，优先级越高。

## Servlet应用

### 利用POST完成登录操作

​		在利用POST完成登录操作前，需要有几个预备：

* 在数据库中存入用户名与密码的数据

* 导入MyBatis和JDBC依赖，遍写MyBatis配置文件
* 编写MyBatis初始化代码，创建实体类和Mapper等等
* 编写关于登录的前端页面代码

​		前端html代码：

```html
<body>
    <form method="post" action="login">		<!-- 修改form标签的属性，这样提交表单会向后台发送POST请求 -->
        <div>
            <h1>系统登录</h1>
        </div>
        <hr>
        <div>
            <label>
                <input type="text" placeholder="username" name="username">	<!-- 用户名输入框 -->
            </label>
        </div>
        <div>
            <label>
                <input type="password" placeholder="password" name="password">	<!-- 密码输入框 -->
            </label>
        </div>
        <br>
        <br>
        <div>
            <input id="button" type="submit" value="登录">	<!-- 登录按钮 -->
        </div>
    </form>
</body>
```

​		在前端代码中修改form标签的属性，之后就可以开始编写Servlet的逻辑了：

```java
@WebServlet(value = "/login", loadOnStartup = 1)
public class LoginServlet extends HttpServlet {
    SqlSessionFactory sqlSessionFactory;
    @SneakyThrows
    @Override
    public void init() throws ServletException {
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader("mybatis-config.xml"));
    }	// 初始化代码为MyBatis的初始化

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");		// 响应类型和编码
        Map<String,String[]> map = req.getParameterMap();	// 存入填入的用户名和密码
        if (map.containsKey("username") && map.containsKey("password")) {	// 要用户名和密码都填入信息
            String username = req.getParameter("username");
            String password = req.getParameter("password");
            try (SqlSession sqlSession = sqlSessionFactory.openSession(true)) {		// MyBatis操作
                UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
                User user = userMapper.getUser(username, password);		// user是用户的实体类
                if (user != null) {
                    resp.getWriter().write("登录成功！");
                }else {
                    resp.getWriter().write("您登录的用户不存在或密码不正确！");
                }
            }
        }else {		// 如果用户名和密码有没填的
            System.out.println("您提交的表单不正确！");
        }
    }
}
```

### 重定向和请求转发

​		假如现在有个需求，希望用户登录后转到某页面，就可以用重定向完成，当浏览器收到一个重定向的请求时，会按照重定向相应给出的地址，向该地址发出请求。

​		重定向的实现只需要调用一个方法就可以：

```java
resp.sendRedirect("https://www.baidu.com");
```

​		请求转发和重定向一样都是实现页面的跳转，不同的是，请求转发是一种内部的跳转方式，一般只用于服务器内部的跳转，比如调用另一个Servlet进行处理并返回结果等等，请求转发同样通过调用方法实现：

```java
req.getRequestDispatcher("/...").forward(req,resp);
```

​		请求转发并没有生成新的url，所以客户端只会发送一次请求，并且从代码可以看出，请求转发同时传送了req和resp参数，所以使用请求转发的request域和response域是共享的。

​		对此，可以总结出两者的区别：

* 重定向是客户端转发，请求转发是服务器端转发
* 重定向是一次请求，请求转发是两次请求（客户端请求次数）
* 重定向的地址栏会发生改变，请求转发不会
* 重定向不共享request域和response域，请求转发共享

​		原则上来说，一般要访问外站资源时使用重定向，要保持request域时的数据时使用请求转发。

### 下载和上传文件

​		下载和上传操作涉及到IO操作，为了更快编写IO代码，这里引入一个工具库：

```xml
<dependency>
	<groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

​		接下来就可以编写Servlet了：

```java
@MultipartConfig		// 该注解用于表示此Servlet用于处理文件上传请求
@WebServlet("/file")
public class FileServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("icon/png");	// 下载的文件
        OutputStream outputStream = resp.getOutputStream();		// 输出流
        InputStream inputStream = Resources.getResourceAsStream("icon.png");	// 输入流
        IOUtils.copy(inputStream, outputStream);	// 工具库中的函数，可以快速完成转换
    }	// 下载

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        try (FileOutputStream outputStream = new FileOutputStream("...")) {		// 上传文件到哪，填路径
            Part part = req.getPart("test-file");		// 获取上传文件
            IOUtils.copy(part.getInputStream(), outputStream);
            resp.setContentType("text/html;charset=UTF-8");
            resp.getWriter().write("上传成功！");
        }
    }	// 上传
}
```

​		Servlet编写完成后，编写前端代码：

```html
<!-- 下载 -->
<a href="file" download="icon.jpg">点击下载</a>		<!-- 下载的超链接 -->
<hr>
<!-- 上传 -->
<form method="post" action="file" enctype="multipart/form-data">	<!-- enctype表示此表单用于文件传输 -->
    <div>
    	<input type="file" name="test-file">		<!-- 选择文件 -->
    </div>
    <div>
    	<button>上传文件</button>					<!-- 上传按钮 -->
	</div>
</form>
```





