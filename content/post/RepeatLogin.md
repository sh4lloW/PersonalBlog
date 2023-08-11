---
title: "异端登录顶替"
date: 2023-08-10T08:50:18+08:00
categories:
    - Java
tags:
    - 登录校验
    - Web
draft: false
---

## 异端登录

​		这个需求出现在APP的多端登录场景，当一用户在A设备上处于登录状态时，在B设备上登录会将A设备的登录状态顶替。

​		网上的大多数方法为维护session，通过监听器或过滤器来检测session是否有效，但在项目的Security配置中，session是被禁用的：

```java
// 禁用session
.and()
.sessionManagement()
.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
```

​		在项目没有引入OAUTH2依赖的情况下，必须自行维护登录信息。

## 维护JWT

​		在原本的登录认证中，token只有userId这一个claim，用于辨识不同的用户，为了实现登录的互斥，这里选择多维护一个字段tokenId。用户在执行需要授权的操作时会携带token，每次传进来的token都会被后台解析，这里就需要加入一个判断：解析后的tokenId与当前存储的tokenId是否匹配。

​		综上所述，方案的流程为：

* 用户登录时会获得token作为令牌，因此在生成token时往里面加入tokenId这个claim，tokenId为随机生成的字符串
* 将生成的tokenId存入Redis，key为userId，value为tokenId
* 用户在执行需要授权的操作时传入token，对token进行解析，将其中的tokenId与Redis中存储的tokenId进行匹配

​		这里的基本思路为：如果用户在新设备上进行登录，那么登录时获得的token为新的token，此时的后台已在Redis中更新tokenId，那么原来旧设备的token自然就失效了，登录顶替的提示信息由APP前端给出。

​		以下为部分代码：

​		在token中添加字段并加入Redis：

```java
public String generateToken(String accountId, Long userId) {

    Date nowDate = new Date();
    Date expireDate = new Date(nowDate.getTime() + 1000 * expire);
    
    String tokenId = RandomUtil.randomNumbers(8);

    String token = Jwts.builder()
            .setHeaderParam("typ", "JWT")
            .setSubject(accountId)
            .setIssuedAt(nowDate)
            .claim("userId", userId)
            .claim("tokenId", tokenId)
            .setExpiration(expireDate)//过期时间
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    
    RedisUtil.set(RedisUtil.TOKEN_UNIQUE_ID + userId, tokenId, expire);
    
    return token;
}
```

​		在Jwt的过滤器中添加对tokenId的校验：

```java
Integer userId = (Integer) (claim.get("userId"));
        
String tokenKey = RedisUtil.TOKEN_UNIQUE_ID + userId;
String tokenValue = (String) claim.get("tokenId");

if (!ValidatorUtil.isTokenExpire(tokenKey, tokenValue)) {
    AuthValidateException e = new AuthValidateException(ResponseEnum.TOKEN_ERROR_EXPIRED);
    loginFailureHandler.onAuthenticationFailure(request, response, e);
    return;
}
```

​		校验方法：

```java
​public static boolean isTokenExpire(String key, String value) {
    String tokenId = (String) RedisUtil.get(key);
    // 过期
    if (StrUtil.isBlank(tokenId)) {
        return false;
    }
    // 比对tokenId是否匹配
    return StrUtil.equals(value, tokenId);
}
```