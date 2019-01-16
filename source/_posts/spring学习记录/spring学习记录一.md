---
title: spring学习记录一
date: 2019-01-07 16:33:49
categories: 
    - java
    - Spring
tags: 
    - java
    - Spring 
---

# 基础
## 从HelloWorld开始
1. 创建一个HelloWorld类
```java
public class HelloWorld {
    private String name;

    public HelloWorld(){
        System.out.println("构造函数执行");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        System.out.println("set方法执行");
        this.name = name;
    }

    public void say(){
        System.out.println("Hello："+this.name);
    }
    
    public static void main(String[] args) {
        //1.创建spring的IOC容器对象
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        //2.从IOC容器中获取bean实例
        HelloWorld helloWorld = (HelloWorld) context.getBean("helloWorld");
        helloWorld.say();
    }
}
```
2. 创建spring的配置文件applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置bean-->
    <bean id="helloWorld" class="com.llz.beans.HelloWorld">
        <property name="name" value="spring"/>
    </bean>
</beans>
```
3. 执行HelloWorld的main()方法，可以看到结果如下：
> 构造函数执行  
> set方法执行   
> Hello：spring

当把main()方法中最后2行注释后发现，前2行还是会打印，即<font color="red">实例化applicationContext容器时就会创建HelloWorld对象并给属性赋值</font>

## IOC && DI
### IOC
- 反转资源获取的方向
- 传统的资源查找方式是组件向容器发起请求查找资源.然后容器适时的返回资源
- 应用IOC后, 则是容器主动地将资源推送给它所管理的组件,组件所要做的仅是选择一种合适的方式来接受资源

#### IOC容器
Spring提供2种类型的IOC容器实现：
1. `BeanFactory`：IOC容器的基本实现，是spring框架的基础设施，面向spring本身
2. `ApplicationContext`：提供更多高级特性，是`BeanFactory`的子接口，面向使用spring的开发者

##### ApplicationContext
![继承树](https://note.youdao.com/yws/api/personal/file/F0D1C30C90954C3FBAE2BDD4F453D2BC?method=download&shareKey=be264240dc366c2456ff9cc14ffe7587)
`ApplicationContext`主要实现类是以下2个：
- `ClassPathXmlApplicationContext`：从类路径下加载配置文件
- `FileSystemXmlApplicationContext`：从文件系统中加载配置文件

其中
- `ConfigurableApplicationContext`新增2个主要方法：`refresh()` 和 `close()`，让`ApplicationContext`具有启动、刷新和关闭上下文的能力
- `ApplicationContext`在初始化上下文时就实例化所有单例的Bean


### DI
- IOC 的另一种表述方式
- 组件以一些预先定义好的方式(例如:setter方法)接受来自如容器的资源注入

#### 依赖注入方式
##### 属性注入(最常用)
- 通过`setter`方法注入Bean的属性值或依赖的对象
- 属性注入使用 `<property>`元素, 使用 `name` 属性指定 Bean 的属性名称，`value` 属性或 <value> 子节点指定属性值 

```xml
<bean id="helloWorld" class="com.llz.beans.HelloWorld">
    <property name="name" value="spring"/>
</bean>
```

##### 构造方法注入
- 该方法注入Bean的属性值或依赖的对象，它保证了Bean实例在实例化后就可以使用
- 构造器注入在`<constructor-arg>`元素里声明属性,`<constructor-arg>` 中没有 `name` 属性

```java
public class Person {
    private String name;
    private String address;
    private int age;

    public Person(String name, String address, int age) {
        this.name = name;
        this.address = address;
        this.age = age;
    }
}
```

```xml
<bean id="person" class="com.llz.beans.Person">
    <!--可以通过index索引来匹配，也能通过type类型来匹配,index和type属性也可以省略，省略时按顺序匹配-->
    <constructor-arg value="张三" index="0"/>
    <constructor-arg value="北京" index="1"/>
    <constructor-arg value="18" type="int"/>
</bean>
```

- 使用该方法注入时，若属性值为字面值，可以使用`<value>`标签或`value`属性注入

```xml
<bean id="person" class="com.llz.beans.Person">
    <constructor-arg index="0">
        <value>张三</value>
    </constructor-arg>
    <constructor-arg value="北京" index="1"/>
    <constructor-arg value="18" type="int"/>
</bean>
```

- 若属性值包含特殊字符，可以使用`<![CDATA[]]>`包裹要注入的值 

```xml
<bean id="person" class="com.llz.beans.Person">
    <constructor-arg index="0">
        <value><![CDATA[张@三&]]></value>
    </constructor-arg>
    <constructor-arg value="北京" index="1"/>
    <constructor-arg value="18" type="int"/>
</bean>
```



##### 工厂方法注入(用的少)

## 配置bean
### 通过全类名(反射)的方式
第一个hellowWorld的demo就是此方式

- 这种方式要求bean中必须有无参构造器

### 静态工厂方法创建bean
```java
public class StaticFactory {
    private static Map<String,Car> cars = new HashMap<String,Car>();

    static {
        cars.put("audi",new Car("audi",300000));
        cars.put("ford",new Car("audi",400000));
    }
    //静态工厂方法
    public static Car getCar(String name){
        return cars.get(name);
    }
}
```

```xml
<!--
    通过静态工厂方法配置bean，注意不是配置静态工厂方法实例，是配置bean的实例
    class：指向静态工厂方法的全类名
    factory-method：指向静态工厂方法的名字
    constructor-arg：如果工厂方法需要传入参数，则使用constructor-arg来配置传入的参数
-->
<bean id="car" class="com.llz.beans.StaticFactory"
    factory-method="getCar">
    <constructor-arg value="audi"></constructor-arg>
</bean>
```

### 实例工厂方法创建bean
```java
public class InstanceFactory {
    private Map<String,Car> cars = null;

    public InstanceFactory(){
        cars = new HashMap<String,Car>();
        cars.put("audi",new Car("audi",300000));
        cars.put("ford",new Car("ford",400000));
    }

    public Car getCar(String name){
        return cars.get(name);
    }
}
```

```xml
<bean id="carFactory" class="com.llz.beans.InstanceFactory"></bean>
<bean id="car" factory-bean="carFactory" factory-method="getCar">
    <constructor value="audi"></constructor>
</bean>
```

### 实现FactoryBean接口创建bean
```java
public class CarFactoryBean implements FactoryBean {
    private String name;
    public void setName(String name) {
        this.name = name;
    }

    //返回bean对象
    @Override
    public Object getObject() throws Exception {
        return new Car(name,500000);
    }

    //返回bean的类型
    @Override
    public Class<?> getObjectType() {
        return Car.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

```xml
<!--
    class：指向FactoryBean的全类名
    property：配置FactoryBean的属性
    返回的是FactoryBean的getObject()方法返回的实例
-->
<bean id="car" class="com.llz.beans.CarFactoryBean">
    <property name="name" value="BMW"/>
</bean>

```

## bean的作用域
- 在 Spring 中, 可以在 `<bean>` 元素的 `scope` 属性里设置 Bean 的作用域
- 默认情况下, Spring只为每个在IOC容器里声明的Bean创建唯一一个实例, 整个 IOC 容器范围内都能共享该实例
- 所有后续的 `getBean()` 调用和 Bean 引用都将返回这个唯一的 Bean 实例.该作用域被称为`singleton`,它是所有Bean的默认作用域


类别 | 说明
---|---
singleton | 默认值，在springIOC容器初始化时创建bean实例，且在整个容器的生命周期内仅存在一个bean实例
prototype | 容器初始化时不创建bean实例，每次调用`getBean()`都会返回一个新的实例
request | 每次HTTP请求都会创建一个新的bean，该作用域仅适用于`WebApplicationContext`环境
session | 同一个HTTP Session共享一个Bean，不同的HTTP Session使用不同的Bean，该作用域仅适用于`WebApplicationContext`环境


## 使用外部属性文件
如加载外部数据库连接信息(需先导入`context`命名空间)：
```xml 
<context:property-placeholder location="classpath:db.properties"/>
```

之后使用`${val}`取值即可




















































