---
title: shiro学习记录一
date: 2019-02-22 10:24:23
description: 文章访问需要密码
password: llz721097
categories: 
    - java
    - shiro
tags: 
    - java
    - shiro 
---

# 缓存管理
- 由于每次权限检查都会去数据库获取权限，效率很低，可以通过设置缓存解决
- shiro可以和`ehcache`或`redis`，此处以`ehcache`为例



## 1.导入jar包
```xml
<dependency>
  <groupId>org.apache.shiro</groupId>
  <artifactId>shiro-ehcache</artifactId>
  <version>1.4.0</version>
</dependency>
<dependency>
  <groupId>net.sf.ehcache</groupId>
  <artifactId>ehcache</artifactId>
  <version>2.10.4</version>
</dependency>
```

## 2.编写ehcache配置文件
shiro默认有一个`ehcache.xml`文件，也可以自己添加自定义配置

## 3.spring-shiro.xml添加ehcache配置
```xml
<!--配置缓存管理器-->
<bean id="ehCacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
    <!--若是自定义ehcache配置文件，则添加该属性指定配置文件，否则可省略-->
    <property name="cacheManagerConfigFile" value="classpath:ehcache.xml"/>
</bean>

<!--配置securityManager-->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="realm" ref="userRealm"/>
    <property name="cacheManager" ref="ehCacheManager"/>
</bean>
```

## 3.清理缓存
如果在运行过程中，主体的权限发生了改变，需要从spring容器中调用realm的清理缓存的方法，在自定义的realm中添加如下代码：

```java
//清理缓存
protected void clearCache() {
    Subject subject = SecurityUtils.getSubject();
    super.clearCache(subject.getPrincipals());
}
```

# Session管理
在之前`spring-shiro.xml`配置文件中添加session管理配置

```xml
<!--配置sessionManager-->
<bean id="sessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
    <!--失效时间，单位：ms-->
    <property name="globalSessionTimeout" value="300000"/>
    <!--是否删除无效session-->
    <property name="deleteInvalidSessions" value="true"/>
</bean>

<!--配置securityManager-->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="realm" ref="userRealm"/>
    <property name="cacheManager" ref="ehCacheManager"/>
    <property name="sessionManager" ref="sessionManager"/>
</bean>
```

# RememberMe
在`spring-shiro.xml`配置文件加入如下配置：

1. 在authc过滤器中设置前台传递表单中记住我的checkbox的name值 

```xml
    
<!--配置authc过滤器-->
<bean id="authc" class="org.apache.shiro.web.filter.authc.FormAuthenticationFilter">
    <!--配置前台传的用户名密码的name属性值，不配置默认为username和password-->
    <property name="usernameParam" value="name"/>
    <property name="passwordParam" value="pwd"/>
    <!--记住我，设置前台传递的表单中记住我的name值为rememberMe-->
    <property name="rememberMeParam" value="rememberMe"/>
</bean>
```

2. 设置`记住我`管理器

```xml
<!--rememberMe cookie配置-->
<bean id="rememberMeCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
    <!--设zcookie存活时间，单位：s-->
    <property name="maxAge" value="604800"/>
    <!--设置cookie名称-->
    <property name="name" value="rememberMe"/>
</bean>


<!--记住我管理器配置-->
<bean id="rememberMeManager" class="org.apache.shiro.web.mgt.CookieRememberMeManager">
    <property name="cookie" ref="rememberMeCookie"/>
</bean>


<!--配置securityManager-->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="realm" ref="userRealm"/>
    <property name="cacheManager" ref="ehCacheManager"/>
    <property name="sessionManager" ref="sessionManager"/>
    <property name="rememberMeManager" ref="rememberMeManager"/>
</bean>
```

3. 在`shiroFilter`中配置哪些资源通过记住我可以再次访问

```xml
<!--配置shiro的过滤器链-->
<property name="filterChainDefinitions">
    <value>
        /login=authc
        /logout=logout
        /js/**=anon
        /css/**=anon
        /images/**=anon
        /index=user     <!--取值为user的即为通过记住我可以再次访问-->
        /**=authc
    </value>
</property>
```


4. 用户类实现序列化接口，该类的引用类也需要实现序列化接口
 
