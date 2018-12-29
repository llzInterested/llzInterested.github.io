---
title: MySQL学习记录之----DQL语句
date: 2017-10-24 09:36:45
categories: 
    - 数据库
    - MySQL
tags: 
    - 数据库
    - MySQL 
---
# DQL
## 常用API
### 单行函数
#### 字符函数
##### +号
+号只能作为运算符
- 2边操作数一方为字符型，一方为数值型，则将字符型转换为数值型，若转换成功，继续做加法运算，转换失败，则将字符型数值转为0
- 一方为null，则结果为null
##### concat()
拼接
```sql
SELECT concat(last_name,first_name) as 姓名 FROM emps;
```
##### ifnull(expr1,expr2)
- expr1：要判断是否为空的字段
- expr2：如果为null返回expr2的值
##### length(expr)
若为utf-8，中文占3个字节，若为gbk，中文占2个字节
- expr：字段名
##### upper(expr)和lower(expr)
大小写转化
##### substr(str,begin,length)
<font color="red">
索引从1开始
</font>     

其中length参数可省略
##### instr(str,substr)
返回子串substr在str中第一次出现的起始索引，如果没有返回0
##### trim()
- 只传一个参数str，则去掉str的前后空格
- trim(char from str)：去掉str中前后的char字符，char可以为多个字符
##### Lpad(str,length,char)
使用指定的char字符将str左填充到长度为length(字符填充到str左边),若str本身长度大于length，则只保留length长度的str，后面部分去掉
##### Rpad(str,lengtg,char)
右填充，其余同上
##### replace(str,substr,str2)
使用str2替换在str中的所有substr
#### 数值函数
##### rand()
产生0到1之间的随机数
##### round(num，digit)
四舍五入
- 若省略digit，则四舍五入到整数
- 若指定digit，则表示四舍五入到第几位小数
##### ceil(num)
上取整
##### floor(num)
下取整
##### truncate(num,digit)
截断，截取小数点前digit位
- digit=0，只取整数部分
- digit<0
```sql
select TRUNCATE(11.234,-1)   ------>   10
select TRUNCATE(11.234,-2)   ------>   0
```
##### mod(num1,num2)
相当于num1 % num2，取余，结果为num1 - num1 / num2 * num2 
#### 日期函数
##### now()
返回当前系统日期和时间
##### curdate()
返回当前系统日期，不包含时间
##### curtime()
返回当前时间，不包含日期
##### 获取指定部分的年，月，日，时，分，秒
- year(now()) ： 获取当前的年份
- month(now()) ： 获取当前月份
- monthname(now()) ： 获取当前月份的英文表示
- day(now()) ： 获取当前的日
- hour(now()) ： 获取当前的时
- minute(now()) ： 获取当前的分
- second(now()) ： 获取当前的秒
##### str_to_date(str,fmt)
将日期格式的str字符串转换成指定格式fmt的日期
```sql
str_to_date('9-13-1999','%m-%d-%Y')     ---->   1999-09-13
```
| 序号 | 格式符 | 功能 |
| :-: | :-: | :-: |
| 1| %Y| 四位的年份|
| 2| %y| 二位的年份|
| 3| %m| 月份(01,02...11,12)|
| 4| %c| 月份(1,2...11,12)|
| 5| %d| 日(01,02...)|
| 6| %H| 小时(24小时制)|
| 7| %h| 小时(12小时制)|
| 8| %i| 分钟(00,01...59)|
| 9| %s| 秒(00,01...59)|
##### date_format(date,fmt)
将日期date转换为指定格式fmt的字符串
```sql
date_format('2018/6/6','%Y年%m月%d日')      ---->   2018年06月06日
```
#### 其他函数
##### version()
查看当前数据库版本号
##### database()
查看当前使用的数据库
##### user()
查看当前用户
#### 流程控制函数
##### if(exp1,exp2,exp3)
若exp1表达式为真，则执行exp2，否则执行exp3
##### case
- 语法1如下：
```sql
case 要判断的字段或表达式
when 常量1 then 要显示的值1或语句1
when 常量2 then 要显示的值2或语句2
...
else 要显示的值n或语句n
end
```
- 语法2如下：
```sql
case 
when 条件1 then 要显示的值1或语句1
when 条件2 then 要显示的值2或语句2
...
else 要显示的值n或语句n
end
```
### 分组函数
用作统计使用，又称聚合函数或统计函数或组函数   

特点：
- sum，avg用于处理数值型，max,min,count可以处理任何类型
- 以上分组函数统计时都忽略null值
- 可以和distinct配合使用
- 和分组函数一同查询的字段要求是group by后的字段
- 当使用group by后，select后的分组函数的结果也会按照分组后的情况分别进行统计
- group by可以按多个字段分组，多字段的顺序不影响查询结果
#### sum(field)
#### avg(field)
#### max(field)
#### min(field)
#### count(field)
## 分组查询

||数据源|位置|关键字|
|---|---|---|---|
|分组前筛选|原始表|group by子句前|where|
|分组后筛选|分组后的结果集|group by子句后|having|

## 连接查询
- sql92标准：仅仅支持内连接
- sql99标准(推荐)：支持内连接 + 外连接(左外和右外) + 交叉连接
### 内连接
#### sql92语法
```sql
select 查询列表
from 表1，表2
where 查询条件
```
#### sql99语法
```sql
select 查询列表
from 表1 别名 [连接类型] join 表2 别名
on 连接条件
where 筛选条件
group by 分组
having 分组后筛选条件
order by 排序列表
```
- sql99中内连接:inner join，当内连接时inner可以省略
- sql99中左外连接:left outer join
- sql99中右外连接:right outer join
- 全外连接:full outer join
- sql99中交叉连接:cross join
#### 等值连接
连接条件用=
```sql
select emp.name,dept.name
from emp,dept
where emp.dept_id = dept.id
```
#### 非等值连接
连接条件不是=
#### 自连接
demo：查询员工和对应上级名称(员工和上级在同一张表)  
```sql
select e.emp_id,e.name,m.emp_id,m.name
from employees e,employees m
where e.manager_id = m.emp_id
```
### 外连接
#### 左外连接  left outer join
#### 右外连接  right outer join
#### 全外连接  full outer join
### 交叉连接   cross join
结果为2表的笛卡尔乘积
## 子查询
### 单行子查询
### 多行子查询（一列多行）
#### IN
等于列表中的任意一个
#### NOT IN
不等于列表中的任意一个
#### ANY/SOME
和子查询返回的某一个值比较
```sql
a > ANY(10,20,30)   //a大于任意一个值就ok
```
#### ALL
和子查询返回的所有值比较
```sql
a > ALL(10,20,30)   //a需要大于所有值
```
### 行子查询(一行多列)
demo:查询员工编号最小工资最高的员工信息
```sql
select * 
from emps
where (emps.id,salary) = (
    select min(id),max(salary)
    from emps
)
```
### exists(子查询语句)
- 使用exists(子查询语句)判断子查询结果集是否为空，不为空返回1，为空返回0
- 上述其他子查询先执行，使用exists包裹的子查询后执行
## 分页查询limit
```sql
select 查询列表
from 表
[连接类型] join 表2
on 连接条件
where 筛选条件
group by 分组字段
having 分组后的筛选
order by 排序的字段
limit offset,size
```
- offset：起始索引(从0开始)
- size：条数
## 联合查询union
- 将多条查询语句的结果合并成一个结果
- 应用场景：要查询的结果来自多个表，且多个表没有直接的连接关系，但查询的信息一致
- 多条查询语句查询列数必须一致，且每列的类型和顺序保持一致
- union关键字默认会去重，如果不想去重，使用union all
```sql
查询语句1
union
查询语句2
union
查询语句3
...
```
