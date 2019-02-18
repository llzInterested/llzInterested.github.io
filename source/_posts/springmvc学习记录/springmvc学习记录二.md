---
title: springmvc学习记录二
date: 2019-01-21 14:35:29
description: 文章访问需要密码
password: llz721097
categories: 
    - java
    - SpringMVC
tags: 
    - java
    - SpringMVC 
---

# jsp
## 9大内置对象

名称 | 类型 | 含义 | 获取方式
---|---|---|---|
request | HttpServletRequest |封装所有请求信息 | 方法参数中 
response | HttpServletResponse | 封装所有响应信息 | 方法参数中
session | HttpSession | 封装所有会话信息 | req.getSession()
application | ServletContext | 所有信息 | getServletContext() / request.getServletContext() / session.getServletContext()
out | PrintWriter | 输出对象 | response.getWriter()
exception | Exception | 异常对象 |
config | ServletConfig | 配置信息 |
page | Object | 当前页面对象 |
pageContext | pageContext | 获取其他对象 | 

## 4大作用域
1. `page`：在当前页面不会重新实例化
2. `request`：在一次请求中是同一个对象，下次请求重新实例化
3. `session`：
    - 一次会话，只要客户端传递的`Jsessionid`不变，`session`就不会重新实例化(不超过默认时间)
    - 实际有效时间：
        - 浏览器关闭，Cookie失效
        - 默认时间，在时间范围内无任何交互，该时间在tomcat的web.xml中配置
        
        ```xml
            <session-config>
                <!--默认30分钟-->
                <session-timeout>30</session-timeout>
            </session-config>
        ```
        
4. `application`：只有在tomcat启动项目时实例化，关闭tomcat时销毁`application`






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

## 2.返回方法上添加注解@ResponseBody

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

### @ResponeseBody作用
- 如果返回值满足`key-value`形式(对象，map，list)
    - 会将响应头设置为`application/json;charset=utf-8`
    - 把转换后的内容输出流的形式响应给客户端
- 当返回值不满足`key-value`形式，如`String`类型时
    - 会将响应头设置为`text/html`
    - 把返回值以流的形式直接输出
    - 如果返回值出现中文，会乱码，解决方式为在`@RequestMapping`注解中添加`produces`属性，`produces`表示响应头中`Content-Type`的取值

```java
@RequestMapping(value="demo1",produces = "text/html;charset=utf-8")
@ResponseBody
public String demo1(){
    return "输出中文字符";
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
    <!--单位：字节-->
    <property name="maxUploadSize" value="1024000"/>
</bean>              
```

## 3.controller中获取上传文件
```java
@RequestMapping("testFileUpload")
public String testFileUpload(@RequestParam("file")MultipartFile file) throws IOException {
    //获取文件名
    String originalFilename = file.getOriginalFilename();
    //获取文件名后缀
    String suffix = originalFilename.substring(originalFilename.lastIndexOf("."));
    String uuid = UUID.randomUUID().toString();
    String path = req.getServletContext().getRealPath("image") + File.separator + uuid + suffix;
    FileUtils.copyInputStreamToFile(file.getInputStream(),new File(path));
    return "index";
}
```


# 异常映射
当想自定义出现什么异常时跳转到指定页面

```xml
<!--异常映射-->
<bean id="simpleMappingExceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings">
        <props>
            <!--当出现上传文件大小超过限制的异常时跳转到根目录的error.jsp页面-->
            <prop key="org.springframework.web.multipart.MaxUploadSizeExceededException">error</prop>
        </props>
    </property>
</bean>
```


# 文件下载
1. 访问资源时响应头如果没有设置`Content-Disposition`，浏览器默认按照inline值进行处理

> `inline`：能显示就显示，不能显示就下载

2. 只需要修改响应头中`Content-Disposition="attachment;filename=文件名"`

> `attachment`：下载，以附件形式下载
> `filename="值"`：是下载时显示的下载文件名


## 1.导入jar包

```xml
<dependency>
  <groupId>commons-fileupload</groupId>
  <artifactId>commons-fileupload</artifactId>
  <version>1.3.3</version>
</dependency>
<dependency>
  <groupId>commons-io</groupId>
  <artifactId>commons-io</artifactId>
  <version>2.4</version>
</dependency>
```

## 2.读取文件返回字节流

```java
@RequestMapping("download")
public void download(String fileName, HttpServletRequest req, HttpServletResponse res) throws IOException {
    res.setHeader("Content-Disposition","attachment;filename=b.txt");
    ServletOutputStream os = res.getOutputStream();
    //获取files文件夹的绝对路径
    String path = req.getServletContext().getRealPath("files");
    File file = new File(path, fileName);
    //读取文件转换为字节流
    byte[] bytes = FileUtils.readFileToByteArray(file);
    os.write(bytes);
    os.flush();
    os.close();
}
```


```html
<a href="/download?fileName=a.txt">下载</a>
```


# 拦截器
1. 和过滤器(`Filter`)较像的技术
2. 发送请求时被拦截器拦截，在控制器的前后添加额外功能
    - AOP是在特定方法前后扩充(对`serviceImpl`)
    - 拦截器是拦截请求，针对的是控制器方法(对`controller`)

## SpringMVC拦截器对比Filter
1. 拦截器只能拦截`Controller`
2. `Filter`可以拦截任何请求  

## 拦截器实现步骤
### 1.自定义拦截器类实现HandlerInterceptor接口
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
     * 可以释放资源,记录出现过的异常
     */
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("afterCompletion");
    }
}
```


### 2.配置文件中配置自定义拦截器
```xml
<!--拦截器配置-->
<mvc:interceptors>
    <!--方法1：自定义拦截器，默认拦截所有请求-->
    <bean class="com.llz.interceptor.MyInterceptor"/>
    
    
    <!--方法2：配置拦截器(不)作用的路径-->
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

