---
title: springmvc学习记录二
date: 2019-01-21 14:35:29
categories: 
    - java
    - SpringMVC
tags: 
    - java
    - SpringMVC 
---

# 处理JSON
## 1.导入jar包
```xml
<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-core-asl</artifactId>
  <version>1.9.13</version>
</dependency>
<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-mapper-asl</artifactId>
  <version>1.9.13</version>
</dependency>
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.8.1</version>
</dependency>
```

## 2.添加springmvc配置文件
```xml
<mvc:annotation-driven/>
```

## 3.返回方法上添加注解@ResponseBody
```java
@ResponseBody
@RequestMapping("testJson")
public List<User> testJson(){
    User u1 = new User(1,"张三","123");
    User u2 = new User(2,"李四","123");
    List<User> list = new ArrayList<>();
    list.add(u1);
    list.add(u2);
    return list;
}
```

# 文件上传
## 1.添加jar包
```xml
<dependency>
  <groupId>commons-io</groupId>
  <artifactId>commons-io</artifactId>
  <version>2.4</version>
</dependency>
<dependency>
  <groupId>commons-fileupload</groupId>
  <artifactId>commons-fileupload</artifactId>
  <version>1.3.1</version>
</dependency>
```

## 2.添加springmvc配置文件
```xml
<!--文件上传，配置MultipartResolver-->
<bean class="org.springframework.web.multipart.commons.CommonsMultipartResolver" id="multipartResolver">
    <property name="defaultEncoding" value="UTF-8"/>
    <property name="maxUploadSize" value="1024000"/>
</bean>
```

## 3.controller中获取上传文件
```java
@RequestMapping("testFileUpload")
public String testFileUpload(@RequestParam("file")MultipartFile file) throws IOException {
    System.out.println("文件名"+file.getOriginalFilename());
    System.out.println("文件输入流"+file.getInputStream());
    return "index";
}
```

# 拦截器
## 1.自定义拦截器类实现HandlerInterceptor接口
```java
public class MyInterceptor implements HandlerInterceptor {
    /**
     * 在目标方法前被调用
     * 若返回值为true，则继续调用后续的拦截器和目标方法
     * 若返回值为false，则不会再调用后续拦截器和目标方法
     * 可以考虑做权限，日志，事务等
     */
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("preHandle");
        return true;
    }

    /**
     * 调用目标方法之后，渲染视图之前
     * 可以对请求域中的属性或视图做出修改
     */
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle");
    }

    /**
     * 渲染视图之后被调用
     * 可以释放资源
     */
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("afterCompletion");
    }
}
```


## 2.配置文件中配置自定义拦截器
```xml
<!--拦截器配置-->
<mvc:interceptors>
    <!--自定义拦截器-->
    <bean class="com.llz.interceptor.MyInterceptor"/>
    <!--配置拦截器(不)作用的路径-->
    <mvc:interceptor>
        <!--<mvc:exclude-mapping path=""/>-->
        <mvc:mapping path="/hello"/>
        <bean class="com.llz.interceptor.SecondInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

## 多个拦截器执行顺序
- `preHandle`顺序执行
- `postHandle`和`afterCompletion`反序执行 


![](https://note.youdao.com/yws/api/personal/file/B3F324623ABC42CC99F10B56102CA972?method=download&shareKey=d58281599ec139bc8e4a598c10a9af52)

当第二个拦截器的`preHanlde`返回`false`后执行顺序如下：

```
graph TB
FirstInterceptor#prehandle --> SecondInterceptor#prehandle
SecondInterceptor#prehandle --> FirstInterceptor#afterCompletion
```

# SpingMVC整体运行流程
![](https://note.youdao.com/yws/api/personal/file/DE4D538F6C984A37A8E4D3CF745E497F?method=download&shareKey=ed2538746a7983a63c43a74bf1134598)





















