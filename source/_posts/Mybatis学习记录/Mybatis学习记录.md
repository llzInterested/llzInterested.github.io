---
title: Mybatis学习记录
date: 2019-02-11 10:23:54
categories: 
    - java
    - Mybatis
tags: 
    - java
    - Mybatis 
---

# resultMap
## resultMap实现关联单个对象(N+1方式) 
### association
`N + 1`查询方式：先查询出某个表的全部信息，根据这个表的信息查询另一个表的信息

实现demo如下(查询员工时附带员工对应的部门信息)：

```java
//Dept类：
public class Dept {
    private Integer id;
    private String dept_name;
}
```

```java
//Emp类
public class Emp{
    private Dept dept;
    private Integer id;
    private String name;
    private Integer salary;
    private Integer dept_id;
}
```

```xml
<!-- DeptMapper.xml -->
<mapper namespace="com.llz.mapper.DeptMapper">
    <select id="selectById" resultType="com.llz.beans.Dept" parameterType="int">
            select * from dept where id=#{0}
    </select>
</mapper>
```

```xml
<!-- EmpMapper.xml -->
<mapper namespace="com.llz.mapper.EmpMapper">
    <resultMap id="empMap" type="com.llz.beans.Emp">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="salary" property="salary"/>
        <result column="dept_id" property="dept_id"/>
        <association property="dept" select="com.llz.mapper.DeptMapper.selectById" column="dept_id"/>
    </resultMap>
    <select id="selectAllEmp" resultMap="empMap">
        select * from emp
    </select>
</mapper>
```

## resultMap实现关联集合对象(N+1方式) 
### 1.collection
实现demo如下(查询部门时附带该部门下所有员工的信息)：

```java
//Dept类
public class Dept {
    private Integer id;
    private String dept_name;
    private List<Emp> emps;
}
```

```java
//Emp类
public class Emp {
    private Integer id;
    private String name;
    private Integer salary;
    private Integer dept_id;
}
```

```xml
<!-- DeptMapper.xml -->
<mapper namespace="com.llz.mapper.DeptMapper">
    <resultMap id="deptMap" type="com.llz.beans.Dept">
        <id column="id" property="id"/>
        <result column="dept_name" property="dept_name"/>
        <collection property="emps" select="com.llz.mapper.EmpMapper.selectByDeptId" column="id"/>
    </resultMap>
    <select id="selectAllDept" resultMap="deptMap">
        select * from dept
    </select>
</mapper>
```

```xml
<!-- EmpMapper.xml -->
<mapper namespace="com.llz.mapper.EmpMapper">
    <select id="selectByDeptId" resultType="com.llz.beans.Emp">
        select * from emp where dept_id = #{0}
    </select>
</mapper>
```

### 2.联合查询
```java
//Dept类
public class Dept {
    private Integer id;
    private String dept_name;
    private List<Emp> emps;
}
```

```java
//Emp类
public class Emp {
    private Integer id;
    private String name;
    private Integer salary;
    private Integer dept_id;
}
```

```xml
<!-- DeptMapper.xml -->
<resultMap id="deptMap" type="com.llz.beans.Dept">
    <id column="did" property="id"/>
    <result column="dept_name" property="dept_name"/>
    <collection property="emps" ofType="com.llz.beans.Emp">
        <id column="eid" property="id"/>
        <result column="name" property="name"/>
        <result column="salary" property="salary"/>
        <result column="dept_id" property="dept_id"/>
    </collection>
</resultMap>

<select id="selectAllDeptAndEmp" resultMap="deptMap">
  select d.id did,dept_name,e.id eid,name,salary,dept_id from dept d left join emp e on d.id = e.dept_id
</select>
```

# 运行流程
demo：

```java
InputStream is = Resources.getResourceAsStream("mybatis.xml");
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
SqlSession session = factory.openSession();
DeptMapper mapper = session.getMapper(DeptMapper.class);
List<Dept> depts = mapper.selectAllDeptAndEmp();
for(Dept d : depts){
    System.out.println(d);
}
```

过程中涉及到的类：
1. `Resources`:Mybatis中IO流的工具类，加载配置文件
2. `SqlSessionFactoryBuilder`：构建器，创建`SqlSessionFactory`接口的实现类
3. `XMLConfigBuilder`：Mybatis全局配置文件内容构建器类，负责读取流内容并转换位java代码
4. `Configuration`：封装了全局配置文件所有配置信息
5. `DefaultSqlSessionFactory`：是`SqlSessionFactory`接口的实现类
6. `Transaction`：事务类，每个`SqlSession`会带有一个`Transaction`对象
7. `TransactionFactory`：事务工厂，负责生产`Transaction`
8. `Executor`：
    - Mybatis的执行器，负责执行SQL命令
    - 相当于JDBC的`statement`对象(或`preparedStatement`,`CallableStatement`对象)
    - 默认的执行器`SimpleExecutor`
    - 批量操作`BatchExecutor`
    - 通过`openSession(参数控制)`

![](https://note.youdao.com/yws/api/personal/file/005EB22A3A1C49A09CCFE6A873EC50C1?method=download&shareKey=348e91144c68acf35b0cb91383e23e3a)

9. `DefaultSqlSession`：`SqlSession`接口的实现类
10. `ExceptionFactory`：Mybatis的异常工厂



```
graph TB
Resources加载全局配置文件 --> 实例化SqlSessionFactoryBuilder构建器

实例化SqlSessionFactoryBuilder构建器 --> 由XMLConfigBuilder解析配置文件流

由XMLConfigBuilder解析配置文件流 --> 把配置信息存放在Configuration中

把配置信息存放在Configuration中 --> 实例化SqlSessionFactory接口的实现类DefaultSqlSessionFactory

实例化SqlSessionFactory接口的实现类DefaultSqlSessionFactory --> 由TransactionFactory创建一个Transaction事务对象

由TransactionFactory创建一个Transaction事务对象 --> 创建执行器Executor

创建执行器Executor --> 创建SqlSession接口的实现类DefaultSqlSession

创建SqlSession接口的实现类DefaultSqlSession --> 实现CRUD

实现CRUD --> 事务提交
```



