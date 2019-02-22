---
title: shiro学习记录一
date: 2019-02-20 11:32:38
description: 文章访问需要密码
password: llz721097
categories: 
    - java
    - shiro
tags: 
    - java
    - shiro 
---

# 概念
shiro是一个基于java的开源的安全管理框架，可以完成认证，授权，会话管理，加密，缓存等功能

![](https://note.youdao.com/yws/api/personal/file/1B9854CEC5A7489392FD7DEA6D0D2A61?method=download&shareKey=cda02e6dd1bb4c2453a3d206ad39bf95)

- `Authentication`：认证，验证用户是否合法，即登录
- `Authorization`：授权，授予谁具有访问某些资源的权限
- `Session Management`：会话管理，用户登录户用户信息通过`Session Management`进行管理
- `Cryptography`：加密，提供了常见的加密算法
- `Web Support`：web应用程序支持，shiro可以很方便的集成到web应用中
- `Caching`：缓存，支持多种缓存，如：ehcache，还支持缓存数据库redis等
- `Concurrency`：并发支持，支持多线程并发访问
- `Testing`：测试
- `Run As`：支持一个用户在允许的前提下使用另一个身份登录
- `Remember Me`：记住我

# 架构
![](https://note.youdao.com/yws/api/personal/file/290B4EC1FFB04D3A9AD32030DDFDD284?method=download&shareKey=cdd5d25535486340fee33f0e14a2284a)

- `subject`：主体，可以是用户，也可以是第三方程序等，subject用于获取`主体信息`，`Principals`和`Credentials`
- `Security Manager`：安全管理器，是shiro架构的核心，来协调管理shiro各个组件之间的工作
- `Authenticator`：认证器，负责验证用户的身份
- `Authorizer`：授权器，负责为合法的用户指定其权限，控制用户可以访问哪些资源
- `Realms`：域
    - 用户通过shiro来完成相关的安全工作，shiro是不会维护数据信息的
    - 在shiro工作过程中，数据的查询和获取工作是通过`Realm`从不同的数据源获取的
    - `Realm`可以获取数据库信息，文本信息等
    - shiro中可以有一个`Realm`也可以有多个



# 用户认证
## 1. Authentication
用户认证，验证用户是否合法，需要提交`Principals`（身份）和`Credentials`（凭证）给shiro

- `Principals`：用户的身份信息，是`Subject`的标识属性，能唯一标识`Subject`，如电话号码，电子邮箱，身份证号码等
- `Credentials`：凭证，是只被`subject`知道的秘密值，可以是密码，也可以是数字证书等
- `Principals`和`Credentials`最常见的组合：用户名和密码。在shiro中通常使用`UsernamePasswordToken`来指定身份和凭证信息

## 2. shiro中用户认证流程：

![](https://note.youdao.com/yws/api/personal/file/44034AB2C7BD4A2F9B830C3EFE35E74E?method=download&shareKey=55471bf62608b0cc1169903aaa06d9ba)

## 3. 代码实现
### 3.1 导入jar包

```xml
<!--pom.xml-->
<dependency>
  <groupId>org.apache.shiro</groupId>
  <artifactId>shiro-core</artifactId>
  <version>1.2.3</version>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.25</version>
</dependency>
<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
  <version>1.2.17</version>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.7.25</version>
  <scope>test</scope>
</dependency>
```

### 3.2 创建shiro.ini配置文件

```
[users]
zhangsan=111
```

### 3.3 编写测试demo

```java
try {
    //1.创建SecurityManager工厂，读取相应的配置文件
    Factory<SecurityManager> factory= new IniSecurityManagerFactory("classpath:shiro.ini");
    //2.通过SecurityManager工厂获取SecurityManager实例
    SecurityManager securityManager = factory.getInstance();
    //3.将SecurityManager设置到运行环境中
    SecurityUtils.setSecurityManager(securityManager);
    //4.通过SecurityUtils获取主体subject
    Subject subject = SecurityUtils.getSubject();
    //5.模拟登录用户：zhangsan/111
    UsernamePasswordToken token = new UsernamePasswordToken("zhangsan","112");
    //6.进行用户身份验证
    subject.login(token);
    //7.通过subject判断用户是否通过验证
    if(subject.isAuthenticated()){
        System.out.println("登录成功");
    }
} catch (AuthenticationException e) {
    e.printStackTrace();
    System.out.println("用户名或密码错误");
}
```


## 4. 常见异常信息及处理
在认证过程中有一个父异常为：`AuthenticationException`，该异常有几个子类，分别对应不同的异常情况

![](https://note.youdao.com/yws/api/personal/file/BF12ED1C898A418F8023BABA243696E6?method=download&shareKey=fcc0111a5ce7769ff236eb1784d2dec0)

- `DisabledAccountException`：账户失效异常
- `ExcessiveAttemptsException`：尝试次数过多
- `UnknownAccountException`：用户不正确
- `ExpiredCredentialsException`：凭证过期
- `IncorrectCredentialsException`：凭证错误

虽然shiro为每种异常都提供了准确的异常类，但提示给用户的异常信息需要模糊，有助于安全

# 使用jdbcRealm完成身份认证
默认shiro使用的是`iniRealm`，如果需要使用其他Realm，需要进行相关配置，`shiro.ini`配置文件中：

- `[users]`：定义一组静态的用户帐户，这在大部分拥有少数用户帐户或用户帐户不需要在运行时被动态地创建的环境下是很有用的

```
[users]
zhangsan=111
lisi=222,role1,role2
```

- `[main]`：配置应用程序的SecurityManager实例及任何它的依赖组件（如 Realms）

```
[main]
myRealm=com.llz.realm.MyRealm
#依赖注入
securityManager.realm=$myRealm
```

- `[roles]`：允许把定义在`[users]`中的角色与权限关联起来。另外，这在大部分拥有少数用户帐户或用户帐户不需要在运行时被动态地创建的环境下是很有用的

```
[roles]
role1=user:add,user:delete
```

shiro的jdbcRealm类中：

![](https://note.youdao.com/yws/api/personal/file/A9CB2D44EAB2481F8B9339DABF895536?method=download&shareKey=2ffc3c772a6ca2bf32cb8ec6b6223fc6)

所以需要为jdbcRealm设置`dataSource`，同时在指定的`dataSource`中有用户表`users`，该表中至少有`username`，`password`，`password_salt`字段

```
# shiro.ini
[main]
dataSource=com.mchange.v2.c3p0.ComboPooledDataSource
dataSource.driverClass=com.mysql.jdbc.Driver
dataSource.jdbcUrl=jdbc:mysql://localhost:3306/shiro_test
dataSource.user=root
dataSource.password=llz721097
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
jdbcRealm.dataSource=$dataSource
securityManager.realm=$jdbcRealm
```

```java
Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
SecurityManager securityManager = factory.getInstance();
SecurityUtils.setSecurityManager(securityManager);
Subject subject = SecurityUtils.getSubject();
UsernamePasswordToken token = new UsernamePasswordToken("lisi","222");
try {
    subject.login(token);
    if(subject.isAuthenticated()){
        System.out.println("验证通过");
    }
} catch (AuthenticationException e) {
    e.printStackTrace();
    System.out.println("验证失败");
}
```

# 认证策略AuthenticationStrategy

实现类 | 描述
---|---
AtLeastOneSuccessfulStrategy | 只要有一个Realm验证成功即可，和FirstSuccessfulStrategy不同，将返回所有Realm身份校验成功的认证信息
FirstSuccessfulStrategy | 只要有一个Realm验证成功即可，只返回第一个Realm身份验证成功的认证信息，其他的忽略
AllSucessfulStrategy | 所有Realm验证成功才算成功，且返回所有Realm身份认证成功的认证信息，如果有一个失败就失败了


默认使用的是`AtLeastOneSuccessfulStrategy`，可以配置其他认证策略：

```
#配置验证器
authenticationStrategy=org.apache.shiro.authc.pam.AllSuccessfulStrategy
securityManager.authenticator.authenticationStrategy=$authenticationStrategy
```

# 自定义Realm实现身份认证
- `jdbcRealm`实现了从数据库中获取用户验证信息，但灵活性差，若需要实现一些特殊需求时需要自定义Realm
- `Realm`是一个接口，定义了根据token获取认证信息的方法，shiro实现了一系列的realm
 
![](https://note.youdao.com/yws/api/personal/file/8B639EC9FE7D44458022C2B8A2CFEFBE?method=download&shareKey=e100b59d7ba3f374a8266c256388e2b5)

- `AuthenticatingRealm`：实现获取身份信息的功能
- `AuthorizingRealm`：实现获取权限信息的功能
- 自定义Realm需要继承`AuthenticatingRealm`，这样提供了身份认证的自定义方法也可以实现授权的自定义


## 散列算法(加密算法)
shiro内部实现了较多的散列算法，如：`MD5`，`SHA`等，且提供了加盐功能，md5的demo如下：

```java
//加密
Md5Hash md5 = new Md5Hash("1111");
System.out.println(md5);
//加盐
md5 = new Md5Hash("1111","llz");
System.out.println(md5);
//迭代次数
md5 = new Md5Hash("1111","llz",2);
System.out.println(md5);
```

## 认证时使用散列算法实现

```
#shiro.ini
[main]
# 配置凭证匹配器
credentialsMatcher=org.apache.shiro.authc.credential.Md5CredentialsMatcher
# 设置算法名称
credentialsMatcher.hashAlgorithmName=md5
# 设置迭代次数
credentialsMatcher.hashIterations=2

myRealm=com.llz.shiro.MyRealmDemo
myRealm.credentialsMatcher=$credentialsMatcher

securityManager.realm=$myRealm
```


```java
public class MyRealmDemo extends AuthorizingRealm {
    /**
     * 获取认证信息，完成身份认证且返回认证信息
     * 认证成功返回true，失败返回null
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        Object username = authenticationToken.getPrincipal();//获取身份信息,用户输入的用户名
        //之后根据用户名查询数据库密码(加密后的)和盐值
        String pwd = "b59c67bf196a4758191e42f76670ceba";
        String salt = "llz";
        //将从数据库中查询的信息封装到SimpleAuthenticationInfo中
        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(username,pwd,ByteSource.Util.bytes(salt),getName());
        return info;
    }

    //获取授权信息
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }
}
```

