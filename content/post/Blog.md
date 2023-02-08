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

​		博客项目的后端包括博客和管理系统两部分。

## 项目初始化 1.15

### 创建多模块项目

​		项目分为父模块Blog，博客子模块blog，管理系统子模块admin，公共模块framework。

​		项目初始目录结构如下：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301152202818.png)

​		更正为：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301201427517.png)

​		原因见[友链 1.19中遇到的问题](#jump1)。

### 接口响应 1.15

​		建立响应枚举类，包括请求的返回类型。

​		建立ReponseResult，集体统一响应，符合接口开发规范。

## 博客

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

​		目前遇到最大的一个问题是项目的配置问题，每次重启IDE后，blog子模块的java包都会被标记为普通包，需要手动将其置为源代码根目录，然后需要将父模块和子模块重新install一遍，不然会出现依赖不加载问题。推测问题出现在SpringBoot的配置文件当中，暂时找不到debug的方法。（1.16）

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

### 友链 1.19

#### 数据表

​		建立友链表link，共有以下字段：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301191703287.png)

#### 需求

​		在前端友链的页面显示出所有审核通过的友链。

#### 实现

​		查询所有审核状态为0的友链数据。

```java
@Override
public ResponseResult<?> getAllLink() {
    // 查询所有审核状态为0的友链

    LambdaQueryWrapper<Link> queryWrapper = new LambdaQueryWrapper<>();
    queryWrapper.eq(Link::getStatus, SystemConstants.LINK_STATUS_PASSED);
    List<Link> linkList = list(queryWrapper);

    // bean copy
    List<LinkVo> linkVoList = BeanCopyUtils.copyBeanList(linkList, LinkVo.class);
    // 封装返回
    return ResponseResult.okResult(linkVoList);
}
```

#### 遇到的问题

​		仍是重启IDE后模块丢失问题，使用索引诊断后确认是文件索引问题：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301191814196.png)

​		尝试过以下方法，均得不到解决：

* [异常：解决idea一直更新索引的问题](https://blog.csdn.net/weixin_45865428/article/details/111885961)	根目录下没有config和system，更改plugins后无效
* [IDEA每次启动都会Indexing](https://blog.csdn.net/qq_45162113/article/details/121128721?spm=1001.2101.3001.6650.16&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-16-121128721-blog-111885961.pc_relevant_3mothn_strategy_recovery&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-16-121128721-blog-111885961.pc_relevant_3mothn_strategy_recovery&utm_relevant_index=23)    仍然无效

​		(1.19)

​		<span id="jump1">问题已解决(1.20)，有点灯下黑了，是maven模块的命名问题，可能是父子模块识别时不区分大小写，我将子模块blog改名为blog-itself以后就没有出现过上述问题。</span>

### 登录功能 1.20

#### 数据表

​		建立用户表user，共有以下字段：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202301201835814.png)

#### 需求

​		实现登录功能，后续有的功能如评论等需要登录后的状态才可以使用。

#### 实现

​		使用SpingSecurity框架实现登录的认证授权。

​		SpringSecurity配置：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        //关闭csrf
        .csrf().disable()
        //不通过Session获取SecurityContext
        .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        .and()
        .authorizeRequests()
        // 对于登录接口 允许匿名访问
        .antMatchers("/login").anonymous()
        // 除上面外的所有请求全部不需要认证即可访问
        .anyRequest().permitAll();

    http.logout().disable();
    //允许跨域
    http.cors();
}
```

​		调用ProviderManager的方法进行认证，认证成功则生成JWT并将用户信息存入Redis中（暂时使用本地的Redis）。

```java
@Override
public ResponseResult<?> login(User user) {
    UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(user.getUserName(), user.getPassword());
    Authentication authenticate = authenticationManager.authenticate(authenticationToken);
    // 判断是否认证通过
    if (Objects.isNull(authenticate)) {
        throw new RuntimeException("用户名或密码错误");
    }
    // 获取userId生成token
    LoginUser loginUser = (LoginUser) authenticate.getPrincipal();
    String userId = loginUser.getUser().getId().toString();
    String jwt = JwtUtil.createJWT(userId);
    // 把用户信息存入Redis
    redisCache.setCacheObject("bloglogin:" + userId, loginUser);
    // 封装token和UserInfo
    UserInfoVo userInfoVo = BeanCopyUtils.copyBean(loginUser.getUser(), UserInfoVo.class);
    BlogUserLoginVo blogUserLoginVo = new BlogUserLoginVo(jwt, userInfoVo);
    return ResponseResult.okResult(blogUserLoginVo);
}
```

```java
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    // 根据用户名查询用户信息
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
    queryWrapper.eq(User::getUserName, username);
    User user = userMapper.selectOne(queryWrapper);
    // 是否查到用户
    if (Objects.isNull(user)) {
        throw new RuntimeException("用户名或密码错误");
    }
    // 返回用户信息
    // TODO 查询权限信息封装
    return new LoginUser(user);
}
```

​		完成了基本的登录功能后需要进行鉴权校验，获取token并提取其中的userId，根据userId从Redis中获取用户的信息存入SecurityContextHolder。

```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Resource
    private RedisCache redisCache;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // 获取token
        String token = request.getHeader("token");
        if (!StringUtils.hasText(token)) {
            // token为空说明该接口不需要登录就能访问
            filterChain.doFilter(request, response);
            return;
        }
        // 提取userId
        Claims claims = null;
        try {
            claims = JwtUtil.parseJWT(token);
        } catch (Exception e) {
            e.printStackTrace();
            // token超时或token非法
            // 响应重新登录
            ResponseResult<?> result = ResponseResult.errorResult(HttpCodeEnum.NEED_LOGIN);
            WebUtils.renderString(response, JSON.toJSONString(result));
            return;
        }
        String userId = claims.getSubject();
        // 根据userId从Redis中检索用户信息
        LoginUser loginUser = redisCache.getCacheObject("bloglogin:" + userId);
        // 如果检索不到
        if (Objects.isNull(loginUser)) {
            ResponseResult<?> result = ResponseResult.errorResult(HttpCodeEnum.NEED_LOGIN);
            WebUtils.renderString(response, JSON.toJSONString(result));
            return;
        }
        // 用户信息存入SecurityContextHolder
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(loginUser, null, null);
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);

        filterChain.doFilter(request, response);
    }
}
```

#### 遇到的问题

​		数据库表与实体类没有对应导致403。

​		未进行认证授权失败的返回结果。

#### 认证授权失败

​		认证授权失败做两个实现类返回异常代码与信息。

```java
@Component
public class AuthenticationEntryPointImpl implements AuthenticationEntryPoint {
    // 认证异常处理
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        authException.printStackTrace();
        ResponseResult<?> result = null;
        if (authException instanceof BadCredentialsException) {
            // 如果是用户名或密码错误
             result = ResponseResult.errorResult(HttpCodeEnum.LOGIN_ERROR.getCode(), authException.getMessage());
        }else if (authException instanceof InsufficientAuthenticationException) {
            // 如果是未登录
            result = ResponseResult.errorResult(HttpCodeEnum.NEED_LOGIN);
        }else {
            result = ResponseResult.errorResult(HttpCodeEnum.SYSTEM_ERROR.getCode(), "认证或授权失败");
        }
        WebUtils.renderString(response, JSON.toJSONString(result));
    }
}
```

```java
@Component
public class AccessDeniedHandlerImpl implements AccessDeniedHandler {
    // 授权异常处理
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        accessDeniedException.printStackTrace();
        ResponseResult<?> result = ResponseResult.errorResult(HttpCodeEnum.NO_OPERATOR_AUTH);
        WebUtils.renderString(response, JSON.toJSONString(result));
    }
}
```

### 统一异常处理 1.21

​		做一个SystemException类，继承自RuntimeException。

```java
@Getter
public class SystemException extends RuntimeException{
    private final int code;
    private final String msg;

    public SystemException(HttpCodeEnum httpCodeEnum) {
        super(httpCodeEnum.getMsg());
        this.code = httpCodeEnum.getCode();
        this.msg = httpCodeEnum.getMsg();
    }
}
```

​		在controller中判断是否输入用户名并返回信息，这部分可以交给前端来做，但是以安全角度考虑的话前后端都要写。

```
RestController
public class BlogLoginController {

    @Resource
    private BlogLoginService blogLoginService;

    @PostMapping("/login")
    public ResponseResult<?> login(@RequestBody User user){
        if (!StringUtils.hasText(user.getUserName())) {
            // 提示需要用户名，其实这里可以交给前端做
            throw new SystemException(HttpCodeEnum.REQUIRE_USERNAME);
        }
        return blogLoginService.login(user);
    }
}
```

​		接着做一个全局的异常处理，将异常信息记录日志并包装为JSON返回给前端。

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    @ExceptionHandler(SystemException.class)
    public ResponseResult<?> systemExceptionHandler(SystemException e) {
        log.error("出现了异常:", e);
        // 封装返回
        return ResponseResult.errorResult(e.getCode(), e.getMsg());
    }

    @ExceptionHandler(Exception.class)
    public ResponseResult<?> exceptionHandler(Exception e){
        //打印异常信息
        log.error("出现了异常:",e);
        //从异常对象中获取提示信息封装返回
        return ResponseResult.errorResult(HttpCodeEnum.SYSTEM_ERROR.getCode(),e.getMessage());
    }
}
```

### 退出登录 1.29

​		过完年以后回来看代码都感觉有点陌生了...

#### 需求

​		登录了账号以后可以点击退出登录重新回到游客状态。

#### 实现

​		重点就是删除Redis中的用户信息。

​		在登录时已经将用户信息存入了SecurityContextHolder，可以根据SecurityContextHolder中的userId来删除Redis中的用户信息。

```java
@Override
public ResponseResult<?> logout() {
    // 获取userId以删除Redis中的用户信息
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    LoginUser loginUser = (LoginUser) authentication.getPrincipal();
    Long userId = loginUser.getUser().getId();
    // 删除
    redisCache.deleteObject("bloglogin:" + userId);
    return ResponseResult.okResult();
}
```

​		并且退出登录的接口必须要授权后才能使用，所以需要更改以下Security的设置。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        //关闭csrf
        .csrf().disable()
        //不通过Session获取SecurityContext
        .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        .and()
        .authorizeRequests()
        // 对于登录接口 允许匿名访问
        .antMatchers("/login").anonymous()
        // 退出接口需要授权
        .antMatchers("/logout").authenticated()
        // 测试用
        .antMatchers("/link/getAllLink").authenticated()
        // 除上面外的所有请求全部不需要认证即可访问
        .anyRequest().permitAll();

    // 异常处理
    http.exceptionHandling()
        .authenticationEntryPoint(authenticationEntryPoint)
        .accessDeniedHandler(accessDeniedHandler);

    http.logout().disable();

    http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
    //允许跨域
    http.cors();
}
```

#### 遇到的问题

​		登录认证授权以及退出登录我都是用postman进行接口测试的，在测试时接口没问题，但是前端联调不上，报的错是需要填写用户名，也就是说前端输入框输入的用户名无法传到后端。

​		查看前端代码后发现是响应格式有问题，username改为userName就可以了。

```javascript
export function userLogin(username,password) {
    return request({
        url: '/login',
        method: 'post',
        headers: {
            isToken: false
          },
        data: {'userName':username,'password':password}
    })
}
```

### 评论列表 1.30

#### 数据表

​		建立评论表comment，共有以下字段：

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202302032252020.png)

#### 需求

​		在文章详情页面显示该文章的评论，评论分为根评论与子评论，子评论上要附带回复人与被回复人的ID。评论显示ID、发布时间与内容。

#### 实现

​		对评论表进行分页查询，前端会传过来文章的ID、分页的页数和页大小。

​		这里分成两部分完成，第一部分是先查询根评论，查询的条件是article_id和root_id，root_id要为-1。第二部分是查询根评论下的子评论，并按生成时间升序排序。

```java
@Service
public class CommentServiceImpl extends ServiceImpl<CommentMapper, Comment> implements CommentService {

    @Resource
    private UserService userService;

    @Override
    public ResponseResult<?> commentList(Long articleId, Integer pageNum, Integer pageSize) {
        // 查询对应文章根评论
        LambdaQueryWrapper<Comment> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(Comment::getArticleId, articleId);
        queryWrapper.eq(Comment::getRootId, SystemConstants.COMMENT_ROOT);
        // 分页
        Page<Comment> page = new Page<>(pageNum, pageSize);
        page(page, queryWrapper);

        // bean copy
        List<CommentVo> commentVoList = fixCommentVoList(page.getRecords());

        // 查询根评论对应的子评论
        for (CommentVo commentVo : commentVoList) {
            List<CommentVo> children = getChildren(commentVo.getId());
            commentVo.setChildren(children);
        }

        // 封装返回
        return ResponseResult.okResult(new ListPageVo(commentVoList, page.getTotal()));
    }

    private List<CommentVo> fixCommentVoList(List<Comment> list) {
        // 数据库中没有Vo的username和toCommentUserName，需要手动赋值
        List<CommentVo> commentVoList = BeanCopyUtils.copyBeanList(list, CommentVo.class);
        for (CommentVo commentVo : commentVoList) {
            // 通过createBy查询username
            commentVo.setUsername(userService.getById(commentVo.getCreateBy()).getNickName());
            // 通过toCommentUserId查询toCommentUserName
            // 需要判断这不是个根评论
            if (commentVo.getToCommentId() != SystemConstants.COMMENT_ROOT) {
                commentVo.setToCommentUserName(userService.getById(commentVo.getToCommentUserId()).getNickName());
            }
        }
        return commentVoList;
    }

    private List<CommentVo> getChildren(Long id) {
        LambdaQueryWrapper<Comment> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(Comment::getRootId, id);
        queryWrapper.orderByAsc(Comment::getCreateTime);
        // 和上面一样需要在这里内部手动赋值
        List<Comment> commentList = list(queryWrapper);
        return fixCommentVoList(commentList);
    }
}
```

#### 遇到的问题

​		无。

### 发送评论功能 1.31

#### 需求

​		用户在登录的状态下可以对文章或友链发送评论、在评论下回复。

#### 实现

​		发送评论功能的实现本身很简单，只需要往数据库里插数据就行，MyBatisPlus有save函数可以完成插入。

```java
@Override
public ResponseResult<?> addComment(Comment comment) {
    // 评论内容不能为空
    if (!StringUtils.hasText(comment.getContent())) {
        throw new SystemException(HttpCodeEnum.CONTENT_NOT_NULL);
    }
    save(comment);
    return ResponseResult.okResult();
}
```

​		这里的重点是数据库的评论表中有创建时间和更新时间等字段，这里建一个工具类自动填充。

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        Long userId = null;
        try {
            userId = SecurityUtils.getUserId();
        } catch (Exception e) {
            e.printStackTrace();
            userId = -1L;//表示是自己创建
        }
        this.setFieldValByName("createTime", new Date(), metaObject);
        this.setFieldValByName("createBy",userId , metaObject);
        this.setFieldValByName("updateTime", new Date(), metaObject);
        this.setFieldValByName("updateBy", userId, metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("updateTime", new Date(), metaObject);
        this.setFieldValByName(" ", SecurityUtils.getUserId(), metaObject);
    }
}
```

​		在评论实体类中设置自动填充。

```java
@TableField(fill = FieldFill.INSERT)
private Long createBy;
@TableField(fill = FieldFill.INSERT)
private Date createTime;
@TableField(fill = FieldFill.INSERT_UPDATE)
private Long updateBy;
@TableField(fill = FieldFill.INSERT_UPDATE)
private Date updateTime;
```

#### 遇到的问题

​		无。

### 友链评论 1.31

#### 需求

​		友链页面需要评论的接口，包括评论的查看和发送。

#### 实现

​		由于文章评论和友链评论在数据库的同一张表中，通过字段type区分，所以这里选择和前面的文章评论列表用同一个接口，传参多一个类型参数用于判断是文章评论还是友链评论。

```java
@GetMapping("/commentList")
public ResponseResult<?> commentList(Long articleId, Integer pageNum, Integer pageSize){
    return commentService.commentList(SystemConstants.ARTICLE_COMMENT, articleId, pageNum, pageSize);
}

@GetMapping("/linkCommentList")
public ResponseResult<?> linkCommentList(Integer pageNum, Integer pageSize) {
    return commentService.commentList(SystemConstants.LINK_COMMENT, null, pageNum, pageSize);
}
```

​		然后修改接口的实现，加一行评论类型的判断，不过友链评论的话不会传articleId，所以在查文章评论的时候才需要加入文章id作为查询条件，其他的都是一样的。

```java
@Override
public ResponseResult<?> commentList(String commentType, Long articleId, Integer pageNum, Integer pageSize) {
    // 查询对应文章或友链的根评论
    LambdaQueryWrapper<Comment> queryWrapper = new LambdaQueryWrapper<>();
    // 只有是文章评论时才查articleId
    queryWrapper.eq(SystemConstants.ARTICLE_COMMENT.equals(commentType), Comment::getArticleId, articleId);
    queryWrapper.eq(Comment::getRootId, SystemConstants.COMMENT_ROOT);
    queryWrapper.eq(Comment::getType, commentType);

    // 分页
    Page<Comment> page = new Page<>(pageNum, pageSize);
    page(page, queryWrapper);

    // bean copy
    List<CommentVo> commentVoList = fixCommentVoList(page.getRecords());

    // 查询根评论对应的子评论
    for (CommentVo commentVo : commentVoList) {
        List<CommentVo> children = getChildren(commentVo.getId());
        commentVo.setChildren(children);
    }

    // 封装返回
    return ResponseResult.okResult(new ListPageVo(commentVoList, page.getTotal()));
}
```

​		这里是通过比对传入的类型，其实判断形参articleId是否为空也可以。

#### 遇到的问题

​		无。

### 用户个人信息 2.1

#### 需求

​		登录后进入个人中心可以查看当前用户的信息。

#### 实现

​		从SecurityContextHolder中获取userId，通过userId来查用户信息。

```java
@SneakyThrows
@Override
public ResponseResult<?> userInfo() {
    // 从SecurityContextHolder中获取userId
    Long userId = SecurityUtils.getUserId();
    // 用userId来查询用户信息
    User user = getById(userId);
    UserInfoVo userInfoVo = BeanCopyUtils.copyBean(user, UserInfoVo.class);
    return ResponseResult.okResult(userInfoVo);
}
```

​		记得修改权限，只有登录后才能查看个人信息。

```java
// 个人信息接口需要授权
.antMatchers("/user/userInfo").authenticated()
```

#### 遇到的问题

​		修改数据库中的头像发现前端并不会更换，看了一下前端代码好像头像是写死的，引用的静态变量。(2.1)

​		这个问题在做完更新个人信息后得到了解决，然而我并没有修改前端代码，前端引用的静态变量仅用于未登录时，真是一个玄学问题...(2.2)

### 头像上传 2.2

#### 需求

​		用户可以上传自己的头像用以更改个人信息。

#### 实现

​		采用七牛云OSS存储用户上传的头像图片。（为什么不用阿里云OSS，因为没钱）

​		上传的相关代码参考了[七牛云官方文档](https://developer.qiniu.com/kodo/1239/java#5)的模板代码，这里将AK、SK、空间名称、图片外链等用SpringBoot的配置文件进行了封装。

​		对文件的后缀类型进行检查，并在上传后将图片的外链返回。

```java
@Service
@Data
@ConfigurationProperties(prefix = "oss")
public class UploadServiceImpl implements UploadService {
    @Override
    public ResponseResult<?> uploadImg(MultipartFile img) {
        // 判断文件类型
        String originalFilename = img.getOriginalFilename();
        // 只准上传png/jpg/jpeg格式图片
        if (!originalFilename.endsWith(".png") && !originalFilename.endsWith(".jpg") && !originalFilename.endsWith(".jpeg")) {
            throw new SystemException(HttpCodeEnum.FILE_TYPE_ERROR);
        }
        // 上传文件到OSS
        String imgUrl = uploadToOSS(img, PathUtils.generateFilePath(originalFilename));
        // 返回图片的外链地址
        return ResponseResult.okResult(imgUrl);
    }

    private String accessKey;
    private String secretKey;
    private String bucket;
    
    private String testCDN;

    @SneakyThrows
    private String uploadToOSS(MultipartFile img, String filePath) {
        //构造一个带指定 Region 对象的配置类
        Configuration cfg = new Configuration(Region.autoRegion());
        cfg.resumableUploadAPIVersion = Configuration.ResumableUploadAPIVersion.V2;// 指定分片上传版本
        //...其他参数参考类注释
        UploadManager uploadManager = new UploadManager(cfg);
        //...生成上传凭证，然后准备上传

        String key = filePath;

        try {
            InputStream inputStream = img.getInputStream();

            Auth auth = Auth.create(accessKey, secretKey);
            String upToken = auth.uploadToken(bucket);

            try {
                Response response = uploadManager.put(inputStream, key, upToken, null, null);
                //解析上传成功的结果
                DefaultPutRet putRet = new Gson().fromJson(response.bodyString(), DefaultPutRet.class);
                System.out.println(putRet.key);
                System.out.println(putRet.hash);
                return testCDN + key;
            } catch (QiniuException ex) {
                Response r = ex.response;
                System.err.println(r.toString());
                try {
                    System.err.println(r.bodyString());
                } catch (QiniuException ex2) {
                    //ignore
                }
            }
        } catch (UnsupportedEncodingException ex) {
            //ignore
        }
        return "error upload";
    }

}
```

​		其中PathUtils是新定义的工具类，它对上传的文件命名进行了重命名，存放在对应日期的路径内，并随机生成uuid。

```java
public class PathUtils {
    public static String generateFilePath(String fileName){
        // 根据日期生成路径
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd/");
        String datePath = sdf.format(new Date());
        // uuid作为文件名
        String uuid = UUID.randomUUID().toString().replaceAll("-", "");
        // 获取文件后缀并替换
        int index = fileName.lastIndexOf(".");
        String fileType = fileName.substring(index);
        return datePath + uuid + fileType;
    }
}
```

#### 遇到的问题

​		原本后端定义了token，要求验证后才可以上传，但是前端没做校验，所以只能把后端的权限校验删除了。

​		第三次看这种官方文档来用API，目前觉得API官方文档写的百度语音识别 > 天行数据接口 > 七牛云OSS。大公司的文档还是写的比较清晰一点啊。

### 更新个人信息 2.2

#### 需求

​		编辑个人资料后点击保存会对个人资料进行更新。

#### 实现

​		更新的操作使用的是put请求，这里直接用MyBatisPlus的方法对对应的userId信息进行更新。

````java
public ResponseResult<?> updateUserInfo(User user) {
    updateById(user);
    return ResponseResult.okResult();
}
````

#### 遇到的问题

​		无。

### 注册用户 2.3

#### 需求

​		可以注册用户，注册后的用户可以进行登录操作。

​		用户名、昵称、邮箱、密码不能为空，且用户名与昵称不能与数据库中的重复，且密码必须加密存储。

#### 实现

​		对传入的参数进行非空判断（这里没用validate，因为要结合DTO会与上面的更新个人信息接口冲突，所以这里写的很笨）并检查数据库中是否重复，再用PasswordEncoder对密码进行加密。

```java
@Override
public ResponseResult<?> register(User user) {
    // 判断数据是否为空
    if (!StringUtils.hasText(user.getUserName())) {
        throw new SystemException(HttpCodeEnum.USERNAME_NOT_NULL);
    }
    if (!StringUtils.hasText(user.getPassword())) {
        throw new SystemException(HttpCodeEnum.PASSWORD_NOT_NULL);
    }
    if (!StringUtils.hasText(user.getEmail())) {
        throw new SystemException(HttpCodeEnum.EMAIL_NOT_NULL);
    }
    if (!StringUtils.hasText(user.getNickName())) {
        throw new SystemException(HttpCodeEnum.NICKNAME_NOT_NULL);
    }
    // 判断用户名、昵称和邮箱是否存在
    if (userNameExist(user.getUserName())) {
        throw new SystemException(HttpCodeEnum.USERNAME_EXIST);
    }
    if (nickNameExist(user.getNickName())) {
        throw new SystemException(HttpCodeEnum.NICKNAME_EXIST);
    }
    if (emailExist(user.getEmail())) {
        throw new SystemException(HttpCodeEnum.EMAIL_EXIST);
    }
    // 密码加密
    String encodedPassword = passwordEncoder.encode(user.getPassword());
    user.setPassword(encodedPassword);
    save(user);
    return ResponseResult.okResult();
}
```

#### 遇到的问题

​		前端的用户名传的不对，去前端改了一下就好了。

### 日志记录 2.3

#### 需求

​		通过日志记录接口调用的信息。

#### 实现

​		使用AOP实现日志记录，实现解耦。

​		日志格式为：

```java
log.info("=======Start=======");
// 打印请求 URL
log.info("URL            : {}",);
// 打印描述信息
log.info("BusinessName   : {}", );
// 打印 Http method
log.info("HTTP Method    : {}", );
// 打印调用 controller 的全路径以及执行方法
log.info("Class Method   : {}.{}", );
// 打印请求的 IP
log.info("IP             : {}",);
// 打印请求入参
log.info("Request Args   : {}",);
// 打印出参
log.info("Response       : {}", );
// 结束后换行
log.info("=======End=======" + System.lineSeparator());
```

​		定义一个SystemLog注解，用以实现通知。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface SystemLog {
    String businessName();
}
```

​		定义切面类，使用`@Pointcut`注解标识切点，`@Around`注解实现通知方法。

```java
@Component
@Aspect
@Slf4j
public class LogAspect {

    // 切点
    @Pointcut("@annotation(org.tseng.annotation.SystemLog)")
    public void establishPointCut() {

    }

    // 通知方法
    @Around("establishPointCut()")
    public Object printLog(ProceedingJoinPoint joinPoint) throws Throwable {
        Object proceed;
        try {
            handleBefore(joinPoint);
            proceed = joinPoint.proceed();
            handleAfter(proceed);
        } finally {
            // 结束换行
            log.info("=======End=======" + System.lineSeparator());
        }

        return proceed;
    }

    private void handleBefore(ProceedingJoinPoint joinPoint) {

        // 获取当前类的实现
        ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = requestAttributes.getRequest();

        // 获取被切的方法对象
        SystemLog systemLog = getSystemLog(joinPoint);

        log.info("=======Start=======");
        // 打印请求的URL
        log.info("URL            : {}", request.getRequestURL());
        // 打印描述信息
        log.info("BusinessName   : {}", systemLog.businessName());
        // 打印 Http method
        log.info("HTTP Method    : {}", request.getMethod());
        // 打印调用 controller 的全路径以及执行方法
        log.info("Class Method   : {}.{}", joinPoint.getSignature().getDeclaringTypeName(), joinPoint.getSignature().getName());
        // 打印请求的 IP
        log.info("IP             : {}", request.getRemoteHost());
        // 打印请求入参
        log.info("Request Args   : {}", JSON.toJSONString(joinPoint.getArgs()));
    }

    private void handleAfter(Object proceed) {
        // 打印出参
        log.info("Response       : {}", JSON.toJSONString(proceed));
    }

    private SystemLog getSystemLog(ProceedingJoinPoint joinPoint) {
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        return methodSignature.getMethod().getAnnotation(SystemLog.class);
    }
}
```

#### 遇到的问题

​		无。

### 浏览量 2.6

#### 需求

​		在用户浏览博文后实现其浏览量的增加。

#### 实现

​		文章的浏览量作为字段存储在article表中，每次应用启动时，将数据库中的字段存储到Redis中，用户访问文章后更新Redis的数据，设置定时将Redis中的数据同步到数据库中，这样做可以减少数据库的访问次数，在并发量大时减轻数据库的压力。

​		文章的浏览量读取的是Redis中的数据。

​		1.应用启动时将数据库字段存储到Redis中

​		使用CommandLineRunner实现项目启动时操作，将整个文章表查询出来，将文章id与其对应的浏览量存进HashMap中，接着同步到Redis。

```java
@Component
public class ViewCountRunner implements CommandLineRunner {

    @Resource
    private ArticleMapper articleMapper;

    @Resource
    private RedisCache redisCache;

    @Override
    public void run(String... args) throws Exception {
        // 查询博文id与其浏览量
        List<Article> articles = articleMapper.selectList(null);
        Map<String, Integer> viewCountMap = new HashMap<>();
        for (Article article : articles) {
            viewCountMap.put(article.getId().toString(), article.getViewCount().intValue());
        }
        // 初始化同步到Redis
        redisCache.setCacheMap(SystemConstants.REDIS_ARTICLE_VIEW_COUNT, viewCountMap);
    }
}
```

​		2.用户访问文章后更新Redis中的数据

​		在工具类里面新增方法用以更新Redis，在ServiceImpl中调用。

```java
/**
*  增加Hash中的数据
*
* @param key Redis键
* @param hKey Hash键
* @param value 值
*/
public void incrementCacheMapValue(final String key, final String hKey, final int value) {
redisTemplate.opsForHash().increment(key, hKey, value);
}
```

```java
@Override
public ResponseResult<?> updateViewCount(Long id) {
    // 更新redis中的浏览量
    redisCache.incrementCacheMapValue(SystemConstants.REDIS_ARTICLE_VIEW_COUNT, id.toString(), 1);
    return ResponseResult.okResult();
}
```

​		3.设置定时将Redis中的数据同步到数据库中

​		用cron表达式设置定时为2分钟一次同步，将Redis中的数据取出并用LambdaUpdateWrapper更新数据库。

```java
@Scheduled(cron = "* 0/2 * * * ?")
public void updateViewCount() {
    // 获取Redis中的浏览量
    Map<String, Integer> viewCountMap = redisCache.getCacheMap(SystemConstants.REDIS_ARTICLE_VIEW_COUNT);
    List<Article> articles = viewCountMap.entrySet()
        .stream()
        .map(entry -> new Article(Long.valueOf(entry.getKey()), entry.getValue().longValue()))
        .collect(Collectors.toList());
    // 同步到数据库
    for (Article article : articles) {
        LambdaUpdateWrapper<Article> queryWrapper = new LambdaUpdateWrapper<>();
        queryWrapper.eq(Article::getId, article.getId());
        queryWrapper.set(Article::getViewCount, article.getViewCount());
        articleService.update(queryWrapper);
    }
}
```

​		4.将读取访问量从数据库改为Redis

​		将文章详情接口的方法加上从Redis读取浏览量就好了。

```java
@Override
public ResponseResult<?> articleDetail(Long id) {
    // 根据id查询文章
    Article article = getById(id);
    // 从Redis中获取浏览量
    Integer viewCount = redisCache.getCacheMapValue(SystemConstants.REDIS_ARTICLE_VIEW_COUNT, id.toString());
    article.setViewCount(viewCount.longValue());
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

​		将Redis的数据同步到数据库时直接写的`articleService.updateBatchById(articles);`，但是报了空指针的错误，发现是前端发起请求时update并不携带token，所以MyBatisPlus在`getAuthentication()`时捕获不到token，这里有两种解决办法，一种是对此进行判空操作，一种是循环检索更新数据库，这里选取的是第二种。

### Swagger2文档 2.7

​		编写文档配置类，并在各接口添加相应注解，生成Swagger文档。

```java
@Configuration
public class SwaggerConfig {
    @Bean
    public Docket customDocket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("org.tseng.controller"))
                .build();
    }

    private ApiInfo apiInfo() {
        Contact contact = new Contact("sh4llow", "https://github.com/sh4lloW", "songofapple@outlook.com");
        return new ApiInfoBuilder()
                .title("博客前台文档")
                .description("博客前台")
                .contact(contact)   // 联系方式
                .version("1.1.0")  // 版本
                .build();
    }
}
```

![img](https://raw.githubusercontent.com/sh4lloW/ImageHostingService/main/BlogImg/202302082343382.png)

## 修改前端 2.8-2.9

​		这两天改下前端，有点丑。

## 管理系统 2.10
