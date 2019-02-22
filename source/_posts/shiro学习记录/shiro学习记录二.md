---
title: shiro学习记录一
date: 2019-02-21 15:27:58
description: 文章访问需要密码
password: llz721097
categories: 
    - java
    - shiro
tags: 
    - java
    - shiro 
---

# 授权
给身份认证通过的人授予可以访问某些资源的权限

- `权限粒度`：分`粗粒度`和`细粒度`
    - `粗粒度`：对资源类型的权限管理，如：菜单、url连接、用户添加页面、用户信息、类方法、页面中按钮
    - `细粒度`：对资源实例的权限管理，资源实例即资源类型的具体化
- shiro一般管理的是粗粒度的权限，如：菜单，按钮，url
- 一般细粒度的权限通过业务控制
- `角色`：权限的集合
- 权限表示规则：`资源:操作:实例`，可以使用通配符表示，如：
    - `user:add` 表示对user有添加的权限
    - `user:*` 表示对user具有所有操作的权限
    - `user:delete:100`：表示对user标识为100的记录有删除权限


## shiro权限流程
![](https://note.youdao.com/yws/api/personal/file/3D1048E1F5594C8F954E472EA1DDA5C5?method=download&shareKey=f1483d4253290d0f85ebbb7a689c9f83)

## 实现
```
#shiro.ini
[users]
zhangsan=111,role1
lisi=222,role2

[roles]
role1=user:add,user:update
role2=user:*
```

```java
Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
SecurityManager securityManager = factory.getInstance();
SecurityUtils.setSecurityManager(securityManager);
Subject subject = SecurityUtils.getSubject();
UsernamePasswordToken token = new UsernamePasswordToken("zhangsan","111");
try {
    subject.login(token);
} catch (AuthenticationException e) {
    e.printStackTrace();
    System.out.println("认证不通过");
}
//1. 基于角色的授权
//1.1 返回bool值
// 1.1.1有该角色返回true，否则返回false
//subject.hasRole("role2");
//1.1.2判断是否具有多个角色
//subject.hasRoles(Arrays.asList("role1", "role2"));

//1.2  抛出异常
//1.2.1 可以通过checkRole检测是否具有某个角色，若不具有该角色则抛出AuthorizerException
//subject.checkRole("role2");
//1.2.2 也可以同时检测多个角色
//subject.checkRoles("role1","role2");

//2. 基于资源的权限
//2.1 返回bool值
//2.1.1 单个权限检查
//subject.isPermitted("user:delete");
//2.1.2 判断是否具有多个权限,全有返回true，否则返回false
//subject.isPermittedAll("user:add","user:update","user:delete");

//2.2 抛出异常
//2.2.1 检测认证用户是否具有某个权限，没有则抛出异常
//subject.checkPermission("user:delete");
//2.2.2 同时检测多个权限，全有不报异常，否则抛异常
//subject.checkPermissions("user:add","user:update","user:delete");

```

## 权限检查方式
### 1.编程式
```java
if(subject.hasRole("管理员")){
    //操作某些资源
}
```

### 2.注解式
在执行指定的方法时检测是否具有该权限

```java
@RequiresRoles("管理员")
public void test(){
    //管理员进行某些操作
}
```

### 3.标签


## 自定义realm实现授权
- 仅仅通过配置文件指定权限不灵活，实际应用中大多数情况是将用户信息，权限信息保存到数据库，需要从数据库中获取相关数据
- 可以使用shiro提供的`jdbcRealm`实现，也可以自定义`realm`实现
- 自定义`realm`需要继承`AuthorizingRealm`

```
#shiro.ini
[main]
myRealm=com.llz.shiro.MyAuthorizingRealmDemo
securityManager.realm=$myRealm
```


```java
public class MyAuthorizingRealmDemo extends AuthorizingRealm {
    //授权信息
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String username = principalCollection.getPrimaryPrincipal().toString();
        System.out.println("用户名：" + username);
        //根据用户名查询数据库查询对应的权限信息
        List<String> permission = new ArrayList<String>();
        permission.add("user:add");
        permission.add("user:update");
        permission.add("user:delete");
        permission.add("user:find");
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        for(String p : permission){
            info.addStringPermission(p);
        }
        return info;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
         //获取身份信息
        User user = (User)principalCollection.getPrimaryPrincipal();
        //根据身份信息查数据库获取权限信息，此处模拟
        List<String> permission = new ArrayList<String>();
        permission.add("user:add");
        permission.add("user:update");
        permission.add("user:delete");
        permission.add("user:find");

        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        for(String p : permission){
            info.addStringPermission(p);
        }
        return info;
    }
    }
}
```

# shiro整合ssm
## 1. 在`web.xml`中添加shiro配置

```xml
<!--配置shiro filter 通过代理配置，对象由spring容器创建，但交给servlet容器管理-->
<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <!--表示bean的生命周期由servlet管理-->
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
    <init-param>
        <!--表示在容器中bean的id，如果不配置，默认和filter的name一致-->
        <param-name>targetBeanName</param-name>
        <param-value>shiroFilter</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 2. 在shiro配置文件`spring-shiro.xml`中添加：

```xml
<mvc:annotation-driven/>
<context:component-scan base-package="com.llz"/>

<!--配置凭证匹配器-->
<bean id="hashedCredentialsMatcher" class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
    <property name="hashAlgorithmName" value="md5"/>
    <property name="hashIterations" value="2"/>
</bean>
    
<!--自定义的realm-->
<bean id="myRealm" class="com.llz.shiro.MyAuthorizingRealmDemo"/>

<bean id="securityManager" class="org.apache.shiro.mgt.DefaultWebSecurityManager">
    <property name="realm" ref="myRealm"/>
</bean>

<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <!--配置securityManager-->
    <property name="securityManager" ref="securityManager"/>
    <!--配置登录的url，若不配置该属性，默认是根路径下的login.jsp
        当访问需要认证的资源时，如果没有认证，会自动跳转到该url
    -->
    <property name="loginUrl" value="/login"/>
    <!--配置认证成功后跳转到哪个url，通常不设置，如果不设置，默认认证成功后跳转到上一个url-->
    <property name="successUrl" value=""/>
    <!--配置用户没有权限访问资源时跳转的url-->
    <property name="unauthorizedUrl" value=""/>
    <!--配置shiro的过滤器链-->
    <property name="filterChainDefinitions">
        <value>
            /login=authc
            /logout=logout
            /js/**=anon
            /css/**=anon
            /images/**=anon
            /**=authc
        </value>
    </property>
</bean>

<!--配置authc过滤器，可不配置-->
<bean id="authc" class="org.apache.shiro.web.filter.authc.FormAuthenticationFilter">
    <!--配置前台传的用户名密码的name属性值，不配置默认为username和password-->
    <property name="usernameParam" value="name"/>
    <property name="passwordParam" value="pwd"/>
</bean>

<!--配置logout过滤器，可不配置-->
<bean id="logoutFilter" class="org.apache.shiro.web.filter.authc.LogoutFilter">
    <!--退出后跳转的url，不配置默认跳转到根路径下-->
    <property name="redirectUrl" value="/login"/>
</bean>
```

其中`filterChainDefinitions`中filter shiro提供的如下：



filterName | class | 描述
---|--- | ---
anon | org.apache.shiro.web.filter.authc.AnonymousFilter | 匿名  
authc | org.apache.shiro.web.filter.authc.FormAuthenticationFilter | 需要认证
authcBasic | org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter
logout | org.apache.shiro.web.filter.authc.LogoutFilter
noSessionCreation | org.apache.shiro.web.filter.session.NoSessionCreationFilter
perms | org.apache.shiro.web.filter.authz.PermissionAuthorizationFilter
port | org.apache.shiro.web.filter.authz.PortFilter
rest | org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter
roles | org.apache.shiro.web.filter.authz.RolesAuthorizationFilter
ssl	| org.apache.shiro.web.filter.authz.SslFilter
user | org.apache.shiro.web.filter.authz.UserFilter


## 3. 自定义的Realm实现

```java
public class UserRealm extends AuthorizingRealm {
    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        String username = authenticationToken.getPrincipal().toString();
        //从数据库查密码和盐值,此处模拟
        String pwd = "a72d5e367fac185e60670613d3be8aa9";
        String salt = "llz";
        User user = new User(1,username,pwd,salt,1);
        return new SimpleAuthenticationInfo(user,pwd,ByteSource.Util.bytes(salt),getName());
    }

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }
}
```



## 4. 添加LoginController

```java
@RequestMapping("/login")
public ModelAndView login(HttpServletRequest req){
    ModelAndView mv = new ModelAndView("login");
    String classname = (String)req.getAttribute("shiroLoginFailure");
    if(UnknownAccountException.class.getName().equals(classname)){
        mv.addObject("msg","用户名或密码错误");
    }else if(IncorrectCredentialsException.class.getName().equals(classname)){
        mv.addObject("msg","用户名或密码错误");
    }else{
        mv.addObject("msg","系统异常");
    }
    return mv;
}
```

## 5. 开启aop代理，使用shiro注解

```xml
<!--spring-shiro.xml-->
<!--开启aop代理 shiro注解-->
<aop:config proxy-target-class="true"></aop:config>
<bean id="authorizationAttributeSourceAdvisor" class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
    <property name="securityManager" ref="securityManager"/>
</bean>
```

## 6.在需要鉴权的方法上添加注解
- `@RequiresPermissions`：
- 

```java
@RequestMapping("/main")
@RequiresPermissions("user:delete")
public String toMainPage(){
    return "main";
}
```

## 7.进行异常处理

```xml
<!--异常映射-->
    <bean id="simpleMappingExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="exceptionMappings">
            <props>
                <prop key="org.apache.shiro.authz.UnauthorizedException">error</prop>
            </props>
        </property>
    </bean>
```

