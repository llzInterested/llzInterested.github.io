---
title: springmvc学习记录一
date: 2019-01-18 16:50:29
categories: 
    - java
    - SpringMVC
tags: 
    - java
    - SpringMVC 
---

# 从HelloWorld开始
## 1.配置web.xml

```xml
<!--配置DispatchServlet-->
    <servlet>
        <servlet-name>springDispatchServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--当不配置contextConfigLocation来指定springmvc配置文件位置时，默认是：/WEB-INF/<servlet-name>-servlet.xml-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>springDispatchServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

## 2.创建springmvc.xml

```xml 
<!--注解扫描-->
<context:component-scan base-package="com.llz"/>
<!--配置视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="WEB-INF/views/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

## 3.创建HelloWorld类
```java
@Controller
public class HelloWorld {
    /**
     * 返回值通过视图解析器解析为实际的物理视图，对InternalResourceViewResolver这个视图解析器，会做如下解析：
     * prefix + return的值 + 后缀 得到实际物理视图，之后做转发操作
     * @return
     */
    @RequestMapping("/helloworld")
    public String hello(){
        System.out.println("hello world");
        return "helloworld";
    }
}
```

# 注解
## @RequestMapping
@RequestMapping参数有：
- `value`：请求url
- `method`：请求方法
- `params`：请求参数
- `headers`：请求头

demo：
- `@RequestMapping(value="/test",method=RequestMethod.POST)`：表示请求路径为test且请求方法为post
- `@RequestMapping(value="/test",params={"username","age!=10"},headers={"Accept-Language=zh-CN,zh;q=0.9"})`：表示请求路径为test，请求参数必须有username和age且age不为10,请求头中Accept-Language的值必须为zh-CN,zh;q=0.9


@RequestMapping支持ant风格的请求url：     
- `?`：匹配文件名中的一个字符
- `*`：匹配文件名中的任意字符
- `**`：匹配多层路径


demo:
- `/user/*/createUser`:匹配/user/<font color="blue">aaa</font>/createUser、/user/<font color="blue">bbb</font>/createUser 等 URL
- `/user/**/createUser`:匹配/user/createUser、/user/<font color="blue">aaa/bbb</font>/createUser 等 URL
- `/user/createUser??`:匹配/user/createUser<font color="blue">aa</font>、/user/createUser<font color="blue">bb</font> 等 URL

## 将请求中的一些信息绑定到入参中
### @PathVariable绑定url参数到入参中
```java
@RequestMapping("delete/{id}")
public String delete(@PathVariable("id") Integer id){
    
}
```

### @RequestParam绑定请求参数
```java
/**
 * value ：请求参数的参数名
 * required ：是否必须，默认为true
 * defaultValue ：默认值
 * @param name
 * @param pwd
 * @return
 */
@RequestMapping("/testRequestParam")
public String testRequestParam(@RequestParam("username")String name,@RequestParam(value="password",required=false) String pwd){
    System.out.println("用户名：" + name + " 密码：" + pwd);
    return "success";
}
```

### @RequestHeader绑定请求头的属性值
```java
@RequestMapping("/testRequestHeader")
public String testRequestHeader(@RequestHeader("Accept-Encoding") String encoding){
    System.out.println("读取请求头信息Accept-Encoding：" + encoding);
    return "success";
}
```


### @CookieValue 绑定请求中的 Cookie 值
```java
@RequestMapping("/testCookieValue")
public String testCookieValue(@CookieValue("JSESSIONID")String sessionId){
    System.out.println("cookie中的sessionId：" + sessionId);
    return "success";
}
```

### 使用POJO对象绑定请求参数值
Spring MVC 会按请求参数名和POJO属性名进行自动匹配，自动为该对象填充属性值。支持级联属性

```java
public class Person {
    private String name;
    private int age;
    private House house;
}

public class House {
    private String address;
    private int size;
}

@RequestMapping("/testPOJO")
    public String testPOJO(Person p){
        System.out.println(p);
        return "success";
    }
```

发起请求的url：localhost:8080/springmvcdemo/testPOJO?name=张三&age=18&house.address=北京&house.size=160

### SpringMVC的handler中支持servletAPI
- HttpServletRequest
- HttpServletResponse
- HttpSession
- java.security.Principal
- Locale
- InputStream
- OutputStream
- Reader
- Writer

## @SessionAttributes
若希望在多个请求之间共用某个模型属性数据，则可以在<font color="red">控制器类</font>上标注一个 `@SessionAttributes`,SpringMVC将在模型中对应的属性暂存到 `HttpSession` 中

```java
@SessionAttributes(value={"house"},types={Integer.class})
@Controller
public class SpringMVCTest01 {
    /**
     * @SessionAttributes 除了可以通过属性名指定需要放到会话中的属性外
     * 还可以通过模型属性的对象类型指定哪些模型属性需要放到会话中
     * @param map
     * @return
     */
    @RequestMapping("/testSessionAttributes")
    public String testSessionAttributes(Map<String,Object> map){
        House h = new House("北京",150);
        map.put("house",h);
        map.put("money",100);
        return "success";
    }
}
```

success.jsp页面中：
```html
requestScope.house : ${requestScope.house}  <br>
sessionScope.house : ${sessionScope.house}  <br>
requestScope.money : ${requestScope.money}  <br>
sessionScope.money : ${sessionScope.money}  <br>
```

## @ModelAttribute
- 在方法定义上使用`@ModelAttribute`注解：该Controller的所有方法在调用前，会先调用标注了`@ModelAttribute`的方法
- 在方法的入参前使用 `@ModelAttribute` 注解：
    - 可以从隐含对象中获取隐含的模型数据中获取对象，再将请求参数绑定到对象中，再传入入参
    - 将方法入参对象添加到模型中



# SpringMVC处理模型数据
## ModelAndView
方法返回值类型为`ModelAndView`时,方法体即可通过该对象添加模型数据

```java
/**
 *springmvc会把ModelAndView的model中数据放入到request域对象中
 */
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
    String viewname = "success";
    ModelAndView mv = new ModelAndView(viewname);
    // 添加模型数据到ModelAndView中
    mv.addObject("time",new Date());
    return mv;
}
```

success.jsp中：`time:  ${requestScope.time}`

## Map和Model
入参为`org.springframework.ui.Model`、`org.springframework.ui.ModelMap` 或`java.util.Map`时，处理方法返回时，Map中的数据会自动添加到模型中

```java
/**
 *springmvc会把Map或者Model中数据放入到request域对象中
 */
@RequestMapping("/testMap")
public String testMap(Map<String,Object> map){
    map.put("name","tom");
    map.put("age",18);
    return "success";
}
```






































