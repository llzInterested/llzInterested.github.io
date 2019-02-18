---
title: spring学习记录二
date: 2019-01-16 15:12:29
description: 文章访问需要密码
password: llz721097
categories: 
    - java
    - Spring
tags: 
    - java
    - Spring
---

# 注解扫描
1. 配置文件中添加自动扫描配置


```xml

<!--
    base-package：
        指定一个需要扫描的基类包，Spring容器将会扫描这个基类包里及其子包中的所有类
        当需要扫描多个包时, 可以使用逗号分隔
    context:exclude-filter：指定排除指定表达式的组件
-->
<context:component-scan base-package="com.llz">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
</context:component-scan>

```

```xml

<!--
    context:include-filter：指定包含指定表达式的组件，需和 use-default-filters="false"配合使用 
-->
<context:component-scan base-package="com.llz" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
</context:component-scan>
```

其中`<context:include-filter>`和`<context:exclude-filter>`中`type`类型如下：

![](https://note.youdao.com/yws/api/personal/file/11A1688299C944EBA55624C069D7CF56?method=download&shareKey=9b4807aa7aa9bb0560f32818d6a29646)


# AOP
![](https://note.youdao.com/yws/api/personal/file/43DB8473B61348968A0B73845DC95C10?method=download&shareKey=c1a021bdad8e5afaf310a1a25edc1c9f)

## 需要的jar包
```xml
<dependency>
    <groupId>aopalliance</groupId>
    <artifactId>aopalliance</artifactId>
    <version>1.0</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.13</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.0.0.RELEASE</version>
</dependency>
```

## 名词
- *切面(Aspect)*: 横切关注点(跨越应用程序多个模块的功能)被模块化的特殊对象，即<font color="blue">功能就是切面</font>，如上截图，有日志切面和验证切面
- *通知(Advice)*:  切面必须要完成的工作，即切面中的每个方法
- *目标(Target)*: 被通知的对象，即业务逻辑
- *代理(Proxy)*: 向目标对象应用通知之后创建的对象
- *连接点（Joinpoint）*：程序执行的某个特定位置：如类某个方法调用前、调用后、方法抛出异常后等。连接点由两个信息确定：方法表示的程序执行点；相对点表示的方位(方法执行前还是执行后)
- *切点（pointcut）*：
    - 即原有的功能
    - 切点和连接点不是一对一的关系，一个切点匹配多个连接点，切点通过`org.springframework.aop.Pointcut`接口进行描述，它使用类和方法作为连接点的查询条件

## AOP实现 
### 1.启用AspectJ注解
1. 导入jar包`aopalliance.jar`、`aspectj.weaver.jar` 和 `spring-aspects.jar`
2. 在配置文件中加入aop命名空间并加入`<aop:aspectj-autoproxy>`
3. 把横切关注点的代码抽象到切面的类中
    - 切面首先是一个IOC中的bean，加入`@component`注解
    - 之后加入`@Aspect`注解声明这是个切面
4. 在类中声明各种通知

```java
//把这个类声明为一个切面，先放入IOC容器，再声明为切面
@Aspect
@Component
public class LogginAspect {
    //声明该方法是前置通知，在目标方法开始之前执行
    @Before("execution(public int com.llz.aop.CalculatorImpl.add(int,int))")
    public void beforeMethod(){
        String methodName = joinPoint.getSignature().getName();
        List<Object> args = Arrays.asList(joinPoint.getArgs());
        System.out.println("方法名称："+methodName+"传的参数"+args);
    }
}
```

#### 通知类型
- *@Before*: 前置通知, 在方法执行之前执行
- *@After*: 后置通知, 在方法执行之后执行(无论是否出现异常)，在后置通知中还不能访问目标方法执行的结果(因为方法可能会出异常) 
- *@AfterRunning*: 返回通知, 在方法返回结果之后执行,返回通知中可以访问方法的返回值
- *@AfterThrowing*: 异常通知, 在方法抛出异常之后，可以访问异常对象，且可以指定在出现特定异常时执行通知代码
- *@Around*: 环绕通知, 围绕着方法执行，功能最强，包含了前置，后置，返回，异常通知

```java
try{
    //前置通知
    result = method.invoke(target,args);
    //返回通知
}catch(Exception e){
    e.printStackTrace();
    //异常通知
}

//后置通知
```

环绕通知demo如下：
```java
/**
     * 环绕通知需带ProceedingJoinPoint类型的参数
     * 环绕通知类似于动态代理的全过程：ProceedingJoinPoint类型的参数可以决定是否执行目标方法
     * 环绕通知必须有返回值，且返回值为目标方法的返回值
     * @param pjd
     * @return
     */
    @Around("execution(public int com.llz.aop.CalculatorImpl.add(int,int))")
    public Object aroundMethod(ProceedingJoinPoint pjd){
        Object result = null;
        String methodName = pjd.getSignature().getName();
        try {
            System.out.println("前置通知，方法名"+methodName+"，方法参数"+Arrays.asList(pjd.getArgs()));
            result = pjd.proceed();     //执行目标方法
            System.out.println("返回通知，返回结果："+result);
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            System.out.println("异常通知，异常为"+throwable);
        }
        System.out.println("后置通知");
        return result;
    }
```

#### 切入点表达式
- `execution * com.llz.spring.ArithmeticCalculator.*(..)`      
    - 匹配 ArithmeticCalculator 中声明的所有方法
    - 第一个 `*` 代表任意修饰符及任意返回值
    - 第二个 `*` 代表任意方法
    - `..` 匹配任意数量的参数         
    - 若目标类与接口与该切面在同一个包中, 可以省略包名
- `execution public * ArithmeticCalculator.*(..)`：匹配 ArithmeticCalculator 接口的所有公有方法
- `execution public double ArithmeticCalculator.*(..)`:匹配 ArithmeticCalculator 中返回 double 类型数值的方法
- `execution public double ArithmeticCalculator.*(double, ..)`: 匹配第一个参数为double类型的方法,`..`匹配任意数量任意类型的参数
- `execution public double ArithmeticCalculator.*(double, double)`: 匹配参数类型为 double, double 类型的方法

##### 重用切入点表达式
当同一个切点表达式可能会在多个通知中重复出现时使用

1. 通过 `@Pointcut`注解将一个切入点声明成简单的方法,切入点的方法体通常是空的
2. 切入点方法的访问控制符同时也控制着这个切入点的可见性
3. 其他通知可以通过方法名称引入该切入点


![](https://note.youdao.com/yws/api/personal/file/C278D6F97F914D1EA1175402B97F7247?method=download&shareKey=54ad0474098878f2beaeb8a0b6a71771)


#### 指定切面优先级
在切面类上加注解`@Order(num)`,其中num为数字，数值越小优先级越高

#### 基于xml配置声明切面
```xml
<!--aop配置-->
<aop:config>
    <!--配置切点表达式-->
    <aop:pointcut id="pointcut" expression="execution(* com.llz.aop.*(..))"/>
    <!--配置切面及通知-->
    <aop:aspect ref="指向切面类" order="2">
        <aop:before method="前置通知的方法" pointcut-ref="pointcut"/>
        <aop:after method="后置通知的方法" pointcut-ref="pointcut"/>
        <aop:after-returning method="切面类中返回通知的方法" pointcut-ref="pointcut" returning="result"/>
        <aop:after-throwing method="切面类中异常通知的方法" pointcut-ref="pointcut" throwing="e"/>
    </aop:aspect>
</aop:config>
```

### 2.schema-base方式
[schema-base方式实现aop](https://blog.csdn.net/qq_41617744/article/details/80279675)


# Spring的事务管理
## 声明式事务
- `声明式事务`：事务控制代码已经由spring写好，程序员只需要声明出哪些方法需要进行事务控制和如何进行事务控制
- `编程式事务`：由程序员编写事务控制代码

## 基于注解使用
### 1.需要导入的jar包
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.0.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>4.0.0.RELEASE</version>
</dependency>
```

### 2.配置事务管理器
```xml
<!--配置事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<!--启用事务注解-->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

### 3.在需要使用事务的方法前加注解@Transactional

## 基于配置文件使用

### 1.配置事务管理器
```xml
<!--配置事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

### 2.配置事务属性
```xml
<!--配置事务属性-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--<tx:method name="*"/>-->
        <tx:method name="get*" read-only="true"/>
        <tx:method name="find*" read-only="true"/>
        <tx:method name="purchase" propagation="REQUIRES_NEW"/>
    </tx:attributes>
</tx:advice>

<!--配置事务切入点，并管理事务属性-->
<aop:config>
    <aop:pointcut expression="execution(* com.llz.*)" id="txPointCut"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"/>
</aop:config>
```


## 事务的传播行为propagation
- 当事务方法被另一个事务方法调用时,必须指定事务应该如何传播. 例如: 方法可能继续在现有事务中运行, 也可能开启一个新事务, 并在自己的事务中运行
- 事务的传播行为可以由传播属性指定.Spring定义了7种类传播行为,可以在`@Transactional` 注解的 `propagation` 属性中定义

![](https://note.youdao.com/yws/api/personal/file/1AAF8457F8CD49298E71457819E11010?method=download&shareKey=67e372e88e599acff9c637b901f3f3c7)

### REQUIRED
demo：当购买2本书但只够付一本书的钱时，2本书都买不了
![](https://note.youdao.com/yws/api/personal/file/BC5E5C44BD934C2B9229BA1318282169?method=download&shareKey=94087f647726c08bbb5b49d807f4c61c) 

### REQUIRED_NEW
demo：当购买2本书但只够付一本书的钱时，可以买第一本
![](https://note.youdao.com/yws/api/personal/file/739AC9E31DF344C0BFFDA736392110DD?method=download&shareKey=046e4479d63c96d009113aebfeb77dbf)


## 事务的隔离级别isolation
在多线程或并发访问下如何保证访问到的数据具有完整性

### 存在的问题
#### 脏读
1. 一个事务A读取到另一个事务B中未提交的数据
2. B事务中数据可能改变了，此时A事务读取的数据可能和数据库中数据不一致
3. 读取脏数据的过程叫脏读

#### 不可重复读
- 主要针对某行数据(或行中某列)
- 主要针对修改操作
- 2次读取在同一个事务内
- 当事务A第一次读取事务后，事务B对事务A读取的数据进行修改，事务A中再次读取的数据和之前读取的数据不一致，这个过程叫不可重复读

#### 幻读
- 主要针对新增或删除
- 2次事务的结果
- 事务A按照特定条件查询出结果，事务B新增了一条符合条件的数据，事务A中查询的数据和数据库中数据不一致，事务A好像出行了幻觉，这种情况叫幻读

### 隔离级别

![](https://note.youdao.com/yws/api/personal/file/C0B141A9A05E4F5EA6A074B7087B9B26?method=download&shareKey=aa13c8bd22906c20185b1365dd464ae4)

- 事务的隔离级别可以在`@Transactional`注解的`isolation`属性中定义
- 事务的隔离级别要得到底层数据库引擎的支持,而不是应用程序或者框架的支持.
- Oracle 支持的 2种事务隔离级别：`READ_COMMITED`,`SERIALIZABLE`
- Mysql 支持 4种事务隔离级别.




## 事务的回滚规则(rollbackFor,noRollbackFor)
- 默认情况下只有`未检查异常(RuntimeException和Error类型的异常)`会导致事务回滚. 而`受检查异常`不会
- 事务的回滚规则可以通过 `@Transactional` 注解的 `rollbackFor` 和`noRollbackFor`属性来定义.这两个属性被声明为 `Class[]` 类型的, 因此可以为这两个属性指定多个异常类
    - `rollbackFor`:  遇到时必须进行回滚
    - `noRollbackFor`: 一组异常类，遇到时必须不回滚

demo如下：
![](https://note.youdao.com/yws/api/personal/file/9735A6AD9F4749FAB2EE12EAC3E04B56?method=download&shareKey=ddb5aab764fca8b150364713e1ce0b12)


## 事务的超时(timeout)和只读属性(readOnly)
- 由于事务可以在行和表上获得锁,因此长事务会占用资源,并对整体性能产生影响. 
- 如果一个事物只读取数据但不做修改,数据库引擎可以对这个事务进行优化.
- `超时事务属性`:事务在强制回滚之前可以保持多久.这样可以防止长期运行的事务占用资源.单位：s
- `只读事务属性`:表示这个事务只读取数据但不更新数据,这样可以帮助数据库引擎优化事务

demo如下：
![](https://note.youdao.com/yws/api/personal/file/BC6EBBB45DDF44C99C202B9406ADA305?method=download&shareKey=c7afdc5e50227573008d2bd65eb22c05)


# Spring常用注解总结
## @Component
创建类对象，相当于配置<bean/>

## @Service
现在与@Component功能相同，建议写在ServiceImpl类上

## @Repository
现在与@Component功能相同，建议写在数据访问层类上

## @Controller
现在与@Component功能相同，写在控制器类上

## @Resource(jdk中提供的)
- 默认按照`byName`注入
- 若没有名称对象，按照`byType`注入
- 建议对象名称和spring容器中对象名相同

## @Autowired(Spring提供的)
默认按照`byType`注入

## @Value()
获取properties文件中内容

## @PointCut()
定义切点

## @Aspect()
定义切面类

## @Before()
前置通知

## After()
后置通知

## AfterRuturning()
返回通知

## AfterThrowing()
异常通知

## @Arround()
环绕通知

































