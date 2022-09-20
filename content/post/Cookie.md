---
title: "JAVA - Cookie"
date: 2022-08-29T23:26:04+08:00
categories:
    - Java
tags:
    - Web
draft: false
---

## Cookie简介

​		Cookie就是一些数据信息，一般为存储在电脑上的小型文本文件。

​		在现在的很多网站的登录界面中，都会有记住当前登录用户账号密码的选项，如果勾选，当再次登录这个网站时就不用再登陆了。这就是Cookie的应用之一，服务器保存了cookie，返回给浏览器这个信息，于是就可以不用再次登录了。

## Cookie属性

​		一个Cookie有以下属性：

* **name**: Cookie的名称，一旦创建便不可更改
* **value**: Cookie的值，如果值为Unicode字符，需要为字符编码，如果值为二进制数据，则需要使用BASE64编码
* **maxAge**: Cookie的失效时间，单位为秒，默认为-1，maxAge如果为正数，说明将在maxAge秒后失效，如果为负数，则该Cookie为临时Cookie，关闭浏览器就会失效，如果为0，则删除该Cookie
* **domain**: 可以访问该Cookie的域名。比如设置为“.baidu.com”，则所有以“baidu.com”结尾的域名都可以访问该Cookie
* **path**: Cookie的使用路径。如果设置为“/WebTest/”，则只有contextPath为“/WebTest”的程序可以访问该Cookie
* **secure**: 表示该Cookie的是否仅被使用安全协议传输，默认为false
* **comment**: Cookie的用处说明，在浏览器显示该Cookie信息时显示说明，类似注释
* **version**: Cookie使用的版本号。0表示遵循Netscape的Cookie规范，1表示遵循W3C的RFC 2109规范

​		通常情况下，最重要的都是**name**，**value**，**maxAge**，**domain**属性。

## Cookie应用实例

​		在这里尝试对勾选记住账号密码的应用实现，当然，是很简单的版本。

​		在这之前，需要先有一个登录的前端页面和一个Servlet。

​		在前端页面添加勾选的按钮：

```html
<div>
    <label>
        <input type="checkbox" placeholder="记住我" name="rememberMe">
        记住我
    </label>
</div>
```

​		在Servlet中编写对应功能：

```java
@WebServlet(value = "/login", loadOnStartup = 1)
public class LoginServlet extends HttpServlet {
    SqlSessionFactory sqlSessionFactory;
    @SneakyThrows
    @Override
    public void init() throws ServletException {
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader("mybatis-config.xml"));
        
    }

    // doGet方法中完成用Cookie跳转的操作
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie[] cookies = req.getCookies();	// 所有Cookie
        if (cookies != null) {
            String username = null;
            String password = null;
            for (Cookie cookie : cookies) {		// 遍历所有Cookie，找到用户名和密码，赋值给字符串
                if (cookie.getName().equals("username")) {
                    username = cookie.getValue();
                }
                if (cookie.getName().equals("password")) {
                    password = cookie.getValue();
                }
            }
            if (username != null && password != null) {		// 这里是为了保证表单正确
                try (SqlSession sqlSession = sqlSessionFactory.openSession(true)) {		// MyBatis操作
                    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
                    User user = userMapper.getUser(username, password);			// 验证是否有该用户且密码是否正确
                    if (user != null) {
                        resp.sendRedirect("https://www.baidu.com");				// 正确就直接重定向到目标网站
                        return;
                    }
                }
            }
        }
        req.getRequestDispatcher("/").forward(req, resp);						// 没有Cookie还是默认请求转发到根目录
    }

    // doPost中为登录操作
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        Map<String,String[]> map = req.getParameterMap();
        if (map.containsKey("username") && map.containsKey("password")) {
            String username = req.getParameter("username");
            String password = req.getParameter("password");
            try (SqlSession sqlSession = sqlSessionFactory.openSession(true)) {
                UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
                User user = userMapper.getUser(username, password);
                if (user != null) {
                    if (map.containsKey("rememberMe")) {	// 假如勾选了选项
                        Cookie cookie_username = new Cookie("username", username);	// 创建记录用户名和密码的Cookie
                        Cookie cookie_password = new Cookie("password", password);
                        resp.addCookie(cookie_username);	// 将Cookie返回给浏览器
                        resp.addCookie(cookie_password);
                    }
                    resp.sendRedirect("https://www.baidu.com");		// 假设登录成功后重定向到百度首页
                }else {
                    resp.getWriter().write("您登录的用户不存在或密码不正确！");
                }
            }
        }else {
            System.out.println("您提交的表单不正确！");
        }
    }
}
```

