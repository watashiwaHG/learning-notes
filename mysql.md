# Mysql(管理员身份)
## 查看mysql版本
```
//在cmd中
mysql --version
mysql --V

//在mysql中
select version();
```
## 启动
```
net start mysql80
```
## 关闭
```
net stop mysql80
```
## 登录
```
mysql -h localhost -P 3306 -u root -p密码

//连接本地端口号3306的mysql数据库
mysql -u root -p密码  
```
## 退出
```
exit
```
## 展示数据库
```sql
show databases;
```
## 使用数据库
```sql
use 数据库名称;
```
## 查看数据库中的表
```sql
show tables from 数据库名称;

//查看当前数据库中的表
show tables;
```
## 查看当前数据库
```sql
select database();
```
## 查看表结构
```sql
desc 表名;
```
***
## 查询语句
#### 1.简单查询
```sql
select 字段 from 表名;
select 常量值;
select 表达式;
select 函数;
//`name`中的``可以将字段与关键字区分开，类似于转义
```
#### 起别名 as
```sql
select 100*2 as 结果;
select 100*2 结果;
//当别名与关键字冲突用双引号或者单引号（推荐双引号）
select username as "out put" from user;
```
#### 去重 distinct
```sql
select distinct username from user;
```
#### +号的作用
```sql
//仅仅是运算符，与字符串相加会试图将字符串转换为数字，否则为0，与null相加是null
select 20+"50";
select 11+"好的";
select 20+null;
```
#### 2.条件查询
```sql
select 查询列表 from 表名 where 筛选条件
```
#### 条件运算符
```sql
<  >  =  !=  <>  <=  >=
```
#### 逻辑运算符
```sql
&&    ||    !
and   or   not
```
#### 模糊查询
```sql
like
// %为任意多个字符 _任意单个字符 \转义 或者指定转义符escape
select username from user where username like '%$_h%' escape '$';

between and
// 包含临界值 两个临界值有顺序 等效于>= and <=

in
// in (列表) 判断是否属于列表中的某一项 等效于or

is null | is not null
//判断null值

<=>
//安全等于 既可以判断普通值，也可以判断null值
```
#### 3.排序查询
#### order by
```sql
// desc 降序  asc 升序  不写默认升序
//一般放在语句的最后  limit 除外
select * from user where username = 'hg' order by 'money' desc ;
```
#### 4.分组查询
#### group by 
```sql
查询的字段只能是group by后的字段或者使用分组函数的字段
select max(money),age from user 
group by age;

对表中已有字段的条件筛选应使用where在group by前(where 不支持别名)
select max(money),age from user 
where sex = '男'
group by age;

对分组过后出现的字段进行条件筛选应在group by后使用having
select max(money),age from user 
where sex = '男'
group by age
having age > 20
order by age desc;
```
#### 5.连接查询
#### （1）内连接(sql92)
```sql
笛卡尔积
select usernanme,cardname from user,card;

等值连接
select usernanme,cardname from user as u,card as c
where u.cardid = c.id;
```
#### （2）sql99
```sql
select 字段 
from 表1 别名 【连接类型】
join 表2 别名 
on 连接条件
【where 筛选条件】
【group by 分组】
【having 筛选条件】
【order by 排序列表】

分类：
内连接：inner
外连接
    右外：right 【outer】 
    左外：left 【outer】
    全外：full 【outer】
交叉连接：cross（笛卡尔乘积）
```
#### 6.子查询
```sql
分类：
按子查询出现的位置：
        select后面：
            仅仅支持标量查询
        from后面：
            支持表查询
        where或having后面：
            标量子查询
            列子查询
            行子查询
        exists后面（相关子查询）
            表子查询
按结果集的行列数不同：
        标量子查询（结果集只有一行一列）
        列子查询（结果集只有一列多行）
        行子查询（结果集有一行多列）
        表子查询（结果集一般为多行多列）

多行子查询操作符：
        in / not in
        any | some
        all

select后面
select d.*,(
    select count(*)
    from employees e
    where e.department_id = d.department_id
)个数
from employees d;

exists后面（相关子查询）
结果为0或1
代表是否有结果
select department_name
from department d
where exists(
    select * 
    from department e
    where d.department_id = e.department_id
)
先查外表数据，再与子查询匹配
```
#### 7.分页查询
#### limit
```sql
放在语句最后，执行顺序也是最后
limit 0,5;
从索引为0的数据开始，5条
```
#### 8.联合查询
#### union
```sql
//会去重，列数要相同，以最开始的查询的字段名为初始字段名
select id,cname from user_chinese
union
select uid,name from user_usa
```
***
## 数据操纵语言DML（Data Manipulation Language）
### 插入语句
##### insert
```sql
//方式一
insert into user(id,name,girlfriend)
values(11,'hg',);
//方式二
insert into user
set id = 1,name = 'hg';
//方式一支持多行插入
insert into user(id,name,girlfriend)
values(1,'hg',),
(2,'yxh',),
(3,'ybc',)
//方式一支持子查询
insert into user(id,name,girlfriend)
select 1,'hg',1;
```
### 修改语句
#### update
```sql
update 表名
set 字段名=值,字段名=值,....
where 筛选条件

update 表1 别名
inner|left|right join 表2 别名
on 连接条件
set 列=值,...
where 筛选条件;
```
### 删除语句
#### delete
```sql
//方式一：可以回滚
//1.单表删除
delete from 表名
where 筛选条件
//2.多表删除
delete 表1的别名,(表2的别名)看是否两个表都要删除
from 表1 别名
inner|left|right join 表2 别名
on 连接条件
where 筛选条件;
//方式二:truncate
//删除表中所有数据，重置自增长,不能回滚
truncate table 表名
```
***
## 数据库模式定义语言DDL(Data Definition Language)
### 库的管理
#### 1.库的创建
```sql
create database [if not exists] 库名;
```
#### 2.库的修改（字符集）
```sql
alter database 库名 character set gbk;
```
#### 3.库的删除
```sql
drop database [if exists] 库名;
```
### 表的管理
#### 1.表的创建
```sql
create table if not exists 表名(
    列名 列的类型[(长度) 约束],
    列名 列的类型[(长度) 约束],
    ....
)
```
#### 2.表的修改
```sql
修改列名
alter table 表名 change column 原列名 新列名 新列名类型
修改列类型、约束
alter table 表名 modify column 列名 新类型
添加新列
alter table 表名 add column 列名 类型 [first|after 字段名]
删除列
alter table 表名 drop column 列名
修改表名
alter table 表名 rename to 新表名
```
#### 3.表的删除
```sql
drop table if exists 表名
```
#### 4.表的复制
```sql
仅仅复制表的结构
create table 表名 like 旧表名;
复制表的结构和数据
create table 表名
select * from 旧表名;
```
***
## 数据类型
### 整型
```sql
默认有符号，unsigned无符号
create table t_int(
    t1 int,
    t2 int unsigned
)

整型的范围是根据关键字决定的，tinyint、smallint、mediumint、int、bigint
int(50) 长度代表的是显示的长度，与zerofill搭配在左边填充0，并且变为无符号
create table t_int(
    t1 int(10) zerofill
)
```
### 小数
#### 1.浮点型
```sql
float(M,D)
double(M,D)
M为数的总位数，D为小数位数
一般不作规定
```
#### 2.定点型（限制整数和小数位数）
```sql
dec(M,D)
decimal(M,D)
M为数的总位数，D为小数位数
如果不做规定默认(10,0)
```
### 字符型
#### 1.较短的文本
```sql
        写法        M的意思                         特点            空间的耗费    效率
char    char(M)     最大的字符数，可以省略，默认为1   固定长度的字符   比较耗费      高

varchar varchar(M)  最大的字符数，不可以省略          可变长度的字符   比较节省     低

其他：
binary和varbinary用于保存较短的二进制
enum用于保存枚举
set用于保存集合
```
### 日期型
```sql
分类：
date只保存日期
time只保存时间
year只保存年

datetime保存日期+时间
timestamp保存日期+时间

特点：
            字节    范围            时区等的影响
datetime    8       1000-9999       不受

timestamp   4       1970-2038       受
```
## 约束
```sql
分类：六大约束

not null 非空
default 默认
primary key 主键(非空，唯一)
unique 唯一（允许为空）
check 检查（mysql中不支持）
foreign key 外键

主键和唯一的对比：
        保证唯一性    是否允许为空    一个表中可以有多少个     是否允许组合
主键        ✔           ✖                 最多1                ✔

唯一        ✔           ✔                 可以多个              ✔

列级约束：
都支持，foreign key没效果
表级约束：
除了not null，default都支持
语法：在各个字段的最下面
[constraint 约束名] 约束类型(字段名)
                    foreign key(字段名) references 表名(字段名)
```
## 标识列（自增长列）
```sql
用法和约束差不多,但必须用在key上，类型必须为数
auto_increment

修改步长
set auto_increment_increment = 3

通过插入数据修改起始
```
## 视图
```sql
对常用的查询语句进行封装

创建视图
create view 视图名
as
查询语句;

使用视图
select 字段名 from 视图名
where 筛选条件;

修改视图
方法1：
create or replace view 视图名
as
查询语句

方法2：
alter view 视图名
as
查询语句

删除视图
drop view 视图名,视图名..

查看视图结构
desc view 视图名;

可以对视图进行增删改，会对原表进行操作，但视图中不能有分组，不能是常量
```
## 变量
### 系统变量
```sql
1.查看所有的系统变量
show global|session variables;

2.查看满足条件的部分系统变量
show global|[session] variables like '';

3.查看指定的某个系统变量的值
select @@global|[session].系统变量名

4.为某个系统变量赋值
方式一：
set global|[session] 系统变量 = 值;

方式二：
set @@global|[session].系统变量名 = 值;
```
### 用户变量
```sql
1.声明并初始化
set @用户变量名=值;
set @用户变量名:=值;
select @用户变量名:=值;

2.赋值（更新）
set @用户变量名=值;
set @用户变量名:=值;
select @用户变量名:=值;

select 字段 into @用户变量名 from 表;

3.使用（查看用户变量的值）
select @用户变量名;
```
### 局部变量
```sql
应用在begin end中的第一句话
1.声明
declare 变量名 类型;
declare 变量名 类型 default 值;

2.赋值（更新）
set 局部变量名=值;
set 局部变量名:=值;
select @局部变量名:=值;

select 字段 into 局部变量名 from 表;

3.使用
select 局部变量名;
```
## 存储过程
```sql
1.创建(存储过程体每个语句都要有分号,如果只有一句begin end可以省略)
delimiter $//先修改结束标记
create procedure 存储过程名(参数模式in|out|inout 参数名 参数类型)
begin
    存储过程体;
end $

2.调用语法
call 存储过程名(实参列表)$;

3.删除语法
drop procedure 存储过程名;

4.查看存储过程信息
show create procedure 存储过程名;
```
## 函数
```sql
1.创建
delimiter $//先修改结束标记
create function 函数名(参数名 参数类型) returns 返回类型
begin
    函数体;
    return 返回值;
end $

2.调用
select 函数名(参数列表)$

3.查看函数信息
show function 函数名$

4.删除语法
drop function 函数名$
```
***
## 流程控制结构
### 分支结构
#### 1.if函数
```sql
三元
if(表达式1,表达式2,表达式3)
```
#### 2.case结构
```sql
case 变量|表达式|字段
when 要判断的值 then 返回的值1或语句1;
when 要判断的值 then 返回的值1或语句2;
...
else 要返回的值n或语句n;
end case;

case 
when 要判断的条件1 then 返回的值1或语句1;
when 要判断的条件1 then 返回的值1或语句2;
...
else 要返回的值n或语句n;
end case;
```
#### 3.if结构
```sql
应用在begin end中:
if 条件1 then 语句1;
elseif 条件2 then 语句2;
...
else 语句n;
end if;
```
### 循环结构
#### 1.while
```sql
[标签：]while 循环条件 do
    循环体;
end while [标签];
```
#### 2.loop
```sql
搭配leave(相当于break)和iterate(相当于continue)
[标签:] loop
    循环体;
end loop [标签];
```
#### 3.repeat
```sql
[标签:] repeat
    循环体;
until 结束循环的条件
end repeat [标签];
```
***
## 常用函数
### 单行函数
#### 1.字符函数
#### length求字符串的字节数
```sql
select length('hg');
```
#### contcat字符串拼接
```sql
//与null拼接结果依然是null
select concat(username,psw) from user;
```

#### upper lower大写小写
```sql
select concat(upper(username),lower(psw)) from user;
```
#### substring截取字符串
```sql
//索引从1开始，从index开始截取 
select substring('watashiwahg',10);

//截取指定字符长度
select substring('watashiwahg',10,2);
```
#### instr返回子串第一次出现的索引
```sql
//类似于java中的indexOf
select instr('watashiwahg','hg');
```
#### trim去除字符串前后的字符
```sql
select trim('  hg  ');

//去除指定字符
select trim('a' from 'aaaaaahgaaaaaaaa');
```
#### lpad用指定的字符左填充指定长度
```sql
select lpad('hg',10,'*');
//rpad右填充
```
#### replace替换
```sql
select replace('hglj','lj','nb');
```
#### 2.数学函数
#### round四舍五入
```sql
select round(1.45);

//保留小数点位数
select round(1.4865,2);
```
#### ceil向上取整
```sql
select ceil(1.52);
```
#### floor向下取整
```sql 
select floor(1.2);
```
#### truncate截断
```sql
//保留小数点位数
select truncate(1.552,1);
```
#### mod取余
```sql
// a % b :  a - a / b * b
//-10 % 3 = -10 - -10 / 3 * 3 = -10 - (-9) = -1
select mod(-10,3);
```
#### 3.日期函数
#### now当前日期和时间
```sql
select now();
```
#### curdate当前日期
```sql
select curdate();
```
#### curtime当前时间
```sql
select curtime();
```
#### 指定年月日时分秒
```sql
select year('1998-1-5');
select moth();
select day();
....
```
#### str_to_date字符串转日期
```sql
select str_to_date('6-5-1993','%m-%d-%Y');
```
#### date_format日期转字符串
```sql
select date_format('2018/6/6','%Y年%m月%d日');
```
#### datediff计算相差天数
```sql
select datediff('2018-6-15','2018-6-8');
```
![](./复习/sql日期参数.png)
#### 4.其他函数
```sql
//当前版本
select version();

//当前数据库
select database();

//当前用户
select user();
```
####if控制流程函数
```sql
//类似于三元运算
select if(10>5,'yes','no');
```
####case
```sql
/*
类似于java的switch
case 要判断的字段或表达式
when 常量1 then 要显示的值1或语句1
when 常量2 then 要显示的值2或语句2
...
else 要显示的值n或语句n
end
*/
select salary 原始工资,department_id,
case department_id
when 30 then salary*1.1
when 40 then salary*1.2
when 50 then salary*1.3
else salary
end as 新工资
from employees;
/*
类似于java的if else语句
case
when 条件1 then 要显示的值1或语句1
when 条件2 then 要显示的值2或语句2
...
end
*/
select salary,
case
when salary > 20000 then 'A'
then salary > 10000 then 'B'
else 'C'
end as 工资等级
from employees;
```
### 分组函数
```sql
//对一组值进行操作，最后返回一个值 一般用作统计
/*
sum 求和、avg 平均值、max 最大值、min 最小值、count 计算个数
特点：
1.sum、avg一般用于处理数值型
max、min、count可以处理任何类型
2.已上分组函数都忽略null值
3.count(*) 统计行数 
MYISAM存储引擎下，count(*)效率更高一些
INNODB存储引擎下，count(*)和count(1)效率差不多，比count(字段)效率高
4.和分组函数一起查询的字段需为group by后面的字段
*/
select sum(salary) from employees;
//5.可以和distinct搭配
select sum(distinct age) from user
```
#### ifnull替换为null的字段
```sql
//将为null的字段替换
select ifnull(username,'hg') from user;
```
#### isnull判断字段是否为null
```sql
select isnull(username) from user;
```
## 事务
### 事务的ACID属性
![](./复习/数据库事务的ACID属性.png)
### 事务的创建
```sql
隐式事务：事务没有明显的开启和结束的标记（应该是系统自动提交了）
比如update、insert、delete语句

显示事务：事务具有明显的开启和结束的标记
前提：必须先设置自动提交功能为禁用,每次事务之前都要设置
set autocommit = 0;

步骤1：开启事务
set autocommit = 0;
[start transaction;]

步骤2：编写事务中的sql语句（select insert update delete）
语句1；
语句2；
....

步骤3：结束事务
commit; 提交事务

rollback; 回滚事务
```
### 数据库的隔离级别
![](./复习/数据库事务的隔离级别1.png)
![](./复习/数据库事务的隔离级别2.png)
### 设置隔离级别
```sql
设置当前对话的隔离级别
set session transaction isolation level read committed;

设置数据库全局的隔离级别
set global transaction isolation level read committed;
```
### 回滚点
```sql
set autocommitted = 0;
start transaction;
delete from user where id = 2;
savepoint a; #设置保存点
delete from user where id = 3;
rollback to a; #回滚到保存点,保存点之前的语句提交，之后的不提交
```
****
## 在Linux下配置文件放在/etc下
## 数据文件
windows： mysql/data
linux： /var/lib/mysql
## frm文件
存放表结构
## myd文件
存放表数据
## myi文件
存放表索引
## MySql逻辑架构简介
![](./复习/mysql逻辑架构简介.png)
1. 连接层

    最上层是一些客户端和连接服务，包含本地sock通信和大多数基于客户端/服务端工具实现的类似tcp/ip的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的的操作权限。

2. 服务层

    第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化及部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定查询表的顺序,是否利用索引等，最后生成相应的执行操作。如果是select语句，服务器还会查询内部的缓存。如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

3. 引擎层

    存储引擎层，存储引擎真正的负责了MySql中数据的存储和提取，服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取。

4. 存储层

    数据存储层，主要是将数据存储在运行于裸设备的文件系统之上，并完成与存储引擎的交互。
## MyISAM和InnoDB存储引擎
![](./复习/InnoDB和MyISAM区别.png)
## SQL性能下降的原因
1. 查询语句写的烂
2. 索引失败 单值、复合
3. 关联查询太多join（设计缺陷或不得已的需求）
4. 服务器调优及各个参数设置（缓冲、线程数等）
## MySql索引
MySql官方对索引的定义为，索引（index）是帮助MySql高效获取数据的数据结构。可以得到索引的本质：索引是数据结构。

***排好序的快速查找数据结构。***

唯一索引默认是B+树索引
### 需要创建索引的情况
1. 主键自动建立唯一索引
2. 频繁作为查询条件的字段应该创建索引
3. 查询中与其他表关联的字段，外键关系建立索引
4. 频繁更新的字段不适合创建索引（因为每次更新不单单是更新了记录还会更新索引）
5. Where条件里用不到的字段不创建索引
6. 单键/组合索引的选择问题（在高并发下倾向创建组合索引）
7. 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度
8. 查询中统计或者分组字段（分组之前必排序）
### 不需要创建索引的情况
1. 表记录太少
2. 经常增删改的表（提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE，因为更新表时，MySql不仅要保存数据，还要保存一下索引文件）
3. 数据重复且分布平均的表字段，因此应该只为最经常查询和最经常排序的数据列建立索引。注意，如果某个数据列包含许多重复的内容，为他建立索引就没有太大的实际效果。
### Explain
查看执行计划，使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MYSQL是如何处理你的sql语句的。分析你的查询语句或是表结构的性能瓶颈。

    Explain + SQL语句
   
#### 能干嘛
1. 表的读取顺序
2. 数据读取操作的操作类型
3. 哪些索引可以使用
4. 哪些索引被实际使用
5. 表之间的引用
6. 每张表有多少行被优化器查询
#### 查询结果
id , select_type , table , type , possible_keys , key , key_len , ref , rows , Extra

##### id 
select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序

1. id相同，执行顺序由上至下
2. id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
3. id相同不同，同时存在
#### select_type
1. SIMPLE 简单的select查询，查询中不包含子查询或者UNION
2. PRIMARY 查询中若包含任何复杂的子部分，最外层查询则被标记为PRIMARY
3. SUBQUERY 在SELECT或WHERE列表中包含了子查询
4. DERIVED 在FROM列表中包含的子查询被标记为DERIVED（衍生）MySql会递归执行这些子查询，把结果放在临时表里
5. UNION 若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED
6. UNION RESULT 从UNION表获取的结果的SELECT
#### table
数据关于的表
#### type
显示访问的类型

从最好到最差依次是：

system > const > eq_ref > ref > range > index > ALL
1. system 表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，这个也可以忽略不计
2. const 表示通过索引一次就找到了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySql就能将该查询转换为一个常量
3. eq_ref 唯一性索引扫描，对于每个索引建，表中只有一条记录与之匹配。常见于主键或唯一索引扫描（连表查询，条件为主键或唯一索引字段）
4. ref 非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，他返回所有匹配某个单独值的行，然而，他可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体
5. range 只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。一般就是在你的where语句中出现了between、<、>、in等的查询。这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引
6. index Full Index Scan，index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）
7. all Full Table Scan，将遍历全表以找到匹配的行。
备注：一般来说，得保证查询至少达到range级别，最好能达到ref