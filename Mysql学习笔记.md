# Mysql学习笔记

## 一丶了解SQL

### 1.什么是数据库？

数据库是一个以某种有组织的方式存储的数据的集合，可以想象成为一个文件柜。

数据库就是保存有组织的数据的容器。

数据库软件是DBMS数据库管理系统，只是他替我们访问数据库。

## 二丶操作数据库(DDL)

### 1.查询数据库

```mysql
#查询所有数据库名称
show databases;
#查看某个数据库的字符集和创建数据库的语法
show create database 数据库名称;
```

### 2.创建数据库

```mysql
#创建数据库并判断是否已经创建
create database if not exists 数据库名称;
#创建数据库并指定字符集
create database 数据库名称 character set gbk;
#创建数据库判断是否存在并指定字符集
create database if not exists 数据库名称 character set 字符集名称 ;
```

### 3.修改数据库

```mysql
#修改数据库的字符集
alter database 数据库名称 character set utf8;
```

### 4.删除数据库

```mysql
#删除数据库并判断是否存在
drop database if exists 数据库名称;
```

### 5.使用数据库

```mysql
#查询当前正在使用的数据库名称
select database();
#使用数据库
use 数据库名称;
```

## 三丶操作表(DDL)

### 1.创建表

```mysql
#创建表
create table 表名 (
	列明1 数据类型1 约束 comment ‘注释’,
    列明2 数据类型2 约束 comment ‘注释’，
    .....
    列明n 数据类型n 约束 comment ‘注释’
);
#示例
CREATE TABLE student (
	id INT PRIMARY KEY COMMENT 'xueshengbianhao',
	`name` VARCHAR(10) COMMENT 'xingming',
	age INT COMMENT 'nianling',
	score DOUBLE(4,1) COMMENT 'fenshu' , 
	brithday DATE COMMENT 'chushengriqi' , 
	insert_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'tianjiashijian'
);
#复制表
create table 新表名 like 旧表名 ;
```

数据类型：

​	int：整数类型

​	double：小数类型

​	date: 日期，只包含年月日

​	datetime: 日期，包含年月日时分秒

​	timestamp：时间戳类型，包含年月日时分秒，如果不赋值，默认使用当前系统时间

​	varchar: 字符串

​			name  varchar(20):姓名最大20个字符

​			zhangsan 是8个字符  张三 是2个字符

### 2.查询表

```mysql
#查询某个数据库中所有的表名称
show tables;
#查询表结构
desc 表名;
```

### 3.修改表

```mysql
#修改表名
alter table 表名 rename to 新表名 ;
#修改表的字符集
alter table 表名 character set 字符集名称;
#添加一列
alter table 表名 add 列名 数据类型 ;
#修改列的名称，类型
alter table 表名 change 列名 新列名 数据类型 ;
alter table 表名 modify 列名 数据类型 ;
#删除列
alter table 表名 drop 列名 ;
```

### 4.删除表

```mysql
#删除表并判断是否存在
drop table if exists 表名;
```

## 四丶操作数据(DML)

### 1.添加数据

```mysql
#添加数据
insert into 表名(列名1,列名2,.....,列名n) values(值1,值2,.....，值n);
```

### 2.修改数据

```mysql
#修改数据
update 表名 set 列名1 = 值1 , 列名2 = 值2 , ..... , 列名n = 值n where 条件;
#注意: 如果不加任何的条件，则会将表中的所有记录全部修改。
```

### 3.删除数据

```mysql
#删除数据
delete from 表名 where 条件 ;     #如果不加条件删除表中所有数据
#删除表中所有的记录,删除表并重新创建
truncate table 表名 ;
```

## 五丶查询表中的记录(DQL)

示例表:

```mysql
#学生表
create table student (
	id int primary key comment '编号',
    name varchar(20) comment '姓名',
    age int comment '年龄',
    sex varchar(5) comment '性别',
    address varchar(100) comment '地址',
	math int comment '数学',
    english int comment '英语'
);
#添加信息
insert into student(id,`name`,age,sex,address,math,english) values (1,'马云',55,'男','杭州',66,78),(2,马化腾',45,'女','深圳',98,87),(3,'马景涛',55,'男','香港',56,77),(4,'柳岩',20,'女','湖南',76,65),(5,'柳青',20,'男','湖南',86,null),(6,'刘德华',57,'男','香港',99,99),(7,'马德',22,'女','香港',99,99);
```



### 1.基础查询

```mysql
#基础查询
#1.查询多个字段
select `name`,age from student ;
#2.去除重复,去重要保证查出来的结果集一样才能去重复
select distinct address from student ;
#3.计算列并起别名,如果有null参与运算则计算结果都为null
select name 姓名,math 数学,english 英语,ifnull(math,0) + ifnull(english,0) as 总分 from student ;
```

### 2.条件查询

```mysql
#查询年龄大于20岁的人
select name,age from student where age > 20 ;
#查询年龄等于20岁的人
select name,age from student where age = 20 ;
#查询年龄不等于20岁的人
select name,age from student where age != 20 ;
#查询年龄在20和30岁之间的人
select name,age from student where age >= 20 and age <= 30 ;
select name,age from student where age between 20 and 30 ;
#查询22岁,19岁,25岁的人
select name,age from student where age = 22 or age = 19 or age = 25 ;
select name,age from student where age in (22,19,25) ;
#查询英语缺考的人,注意null值不可以使用=，<>判断，只能用is或is not
select name,age from student where english is null ;

```

### 3.模糊查询

```mysql
#查询姓马的人
select name,age from student where name like '马%' ;
#查询姓名是三个字的人
select name,age from student where name like '___' ;
#查询姓名中包含德的人
select name,age from student where name like '%德%' ;
```

### 4.排序查询

```mysql
#使用数学成绩进行排序
select * from student order by math asc ;  #升序
select * from student order by english desc ; #降序
#先按数学成绩排名，如果数学成绩一样，则按照英语成绩排列
select * from student order by math asc,english asc ;
```

### 5.聚合函数

```mysql
#聚合函数的作用:将一列数据作为一个整体，进行纵向的计算，聚合函数的计算会排除null值的
#查看班级中一共有多少个人，一般使用主键进行计算
select count(id) 人数 from student ;    #选择不包含非空的列进行计算
select count(ifnull(english,0)) from student ;   #使用ifnull进行判断
select count(*) from student ;
#查看数学成绩的最大值,最小值
select max(math) from student ;
select min(math) from student ;
#查看所有英语成绩的综合
select sum(english) from student ;   #sun会自动忽略null
#计算数学成绩的平均值
select avg(math) from student ;
```

### 6.分组函数

***注意:***

***1.分组之后查询的字段，要么是分组字段，要么是聚合函数。***

***2.where与having的区别:***

​		***a. where在分组之前限定，如果条件不满足，则不参与分组。having在分组之后进行限定,如果不满足的结果，不会被查询出来。***

​		***b. where后不可以跟聚合函数，having可以跟聚合函数。***

```mysql
#按照性别分组，分别查询男，女同学的平均分，以及男女分别有多少人
select sex,avg(math),avg(english),count(id) from student group by sex ; 
#按照性别分组，分别查询男，女同学的平均分，以及男女分别有多少人，分数低于70分的人不参与分组
select sex,avg(math),count(id) from student where math >= 70 group by sex ;
#分组之后加条件,按照性别分组，分别查询男，女同学的平均分，以及男女分别有多少人，分数低于70分的人不参与分组，且分组大于两个人
select sex,avg(math),count(id) from student where math >= 70 group by sex having count(id) > 2;
```

### 7.分页查询

注意:

​	**公式 : 开始的索引 = (当前的页码 - 1) * 每页显示的条数 ;**

```mysql
#每页显示三条记录
select * from student limit 0,3 ;   #第一页 从第0个所有查询3条
select * from student limit 3,3 ;   #第二页
select * from student limit 6,3 ;   #第三页
```

## 六丶约束

### 1.概念

​	对表中的数据进行限定，保证数据的正确性,有效性，和完整性。

### 2.主键约束(primary key)

```mysql
#主键是非空且唯一的,且一张表只能有一个字段为主键,主键就是表中记录的唯一标识
CREATE TABLE stu (
	id INT PRIMARY KEY AUTO_INCREMENT COMMENT '主键id' ,
    `name` VARCHAR(10) COMMENT '姓名'
);
#删除主键约束
alter table stu modify id int ; #这是错误的，会报错
alter table stu drop primary key ; #这才是对的
#创建表之后添加主键
alter table stu modify id int primary key ;

#删除自动增长
alter table stu modify id int ;
#创建表之后添加自动增长
alter table stu modify id int auto_increment ;
```

### 3.非空约束(not null)

```mysql
#创建一个stu表,给名字字段添加非空约束
CREATE TABLE stu (
	id INT PRIMARY KEY AUTO_INCREMENT COMMENT '主键id' ,
    `name` VARCHAR(10) NOT NULL COMMENT '姓名'
);
#删除非空约束
alter table stu modify `name` varchar(10) ;
#创建表完成后添加非空约束
alter table stu modify `name` varchar(10) not null ;
```

### 4.唯一约束(unique)

注意:唯一约束可以有多个null

```mysql
#值不能重复
#创建表时添加唯一约束
create table stu (
	id int primary key auto_increment comment '主键' ,
    phone_number varchar(20) unique comment '手机号'
);
#删除唯一约束
alter table stu modify phone_number varchar(20) ;   #这是错的，不能这样直接删
alter table stu drop index phone_number ; #这个才是对的
#创建表之后添加唯一约束
alter table stu modify phone_number varchar(20) unique ;
```

### 5.外键约束(foreign key)

外键可以为null,但是不可以有不存在的外键值。

```mysql
#需要准备的数据
create table emp (
	id int primary key auto_increment comment '主键',
    `name` varchar(10) not null comment '姓名' ,
    age int comment '年龄',
    dep_name varchar(32)  comment '部门名称',
    dep_location varchar(32) comment '部门地址'
);

#添加数据
insert into emp (name,age,dep_name,dep_location) values('张三',20,'研发部','广州'),('李四',21,'研发部','广州'),('王五',20,'研发部','广州'),('老王',20,'销售部','深圳'),('大王',22,'销售部','深圳'),('小王',18,'销售部','深圳');

#拆分表
#部门表和员工表
create table dept (
	id int primary key auto_increment comment '主键',
    dep_name varchar(32) not null  comment '部门名称',
    dep_location varchar(32) comment '部门地址'
);

create table emp (
	id int primary key auto_increment comment '主键',
    `name` varchar(10) not null comment '姓名' ,
    age int comment '年龄',
    dept_id int comment '部门外键'
);

#添加数据
insert into emp (name,age,dept_id) values('张三',20,1),('李四',21,1),('王五',20,1),('老王',20,2),('大王',22,2),('小王',18,2);

insert into dept (dep_name,dep_location) values('研发部','广州'),('销售部','深圳');
```

```mysql
#创建表并添加外键
create table emp (
	id int primary key auto_increment comment '主键',
    `name` varchar(10) not null comment '姓名' ,
    age int comment '年龄',
    dept_id int  comment '部门外键' ,
    constraint emp_dept_fk foreign key (dept_id) references  dept(id)
);
#删除外键
alter table emp drop foreign key emp_dept_fk ;
#创建表完成之后添加外键
alter table emp add constraint emp_dept_fk foreign key (dept_id) references  dept(id) ;

#级联更新和级联删除,先删除外键，再添加有级联更新的外键
alter table emp add constraint emp_dept_fk foreign key (dept_id) references  dept(id) on update cascade on delete cascade ;
在实际开发中使用级联需要非常的谨慎。
```

## 七丶数据库设计

### 1.多表之间的关系

​	a.一对一  

​		好办法:合成一张表

​		也可以在任意一方添加唯一外键指向另一方的主键。

​	b.一对多

​		在多的一方建立外键，指向一的一方的主键。

​	c.多对多

​		需要建立中间表，中间表最少包含两个字段，这两个字段作为第三张表的外键，分别指向两张表的主键。

### 2.数据库的三大范式

​	a.第一范式:每一列都是一个不可分割的原子项

​	b.第二范式:在第一范式的基础上,非码属性必须完全依赖于候选码

​	c.第三范式:在第二范式的基础上，任何非主属性不依赖与其他非主属性。

## 八丶数据库的备份与还原

```mysql
#备份数据库
mysqldump -u 账号 -p 数据库名称 > 保存的路径
#还原
mysql -u 账号 -p 密码   #登录数据库
create database 数据库名称    #创建数据库
use 数据库名称   #使用数据库
source sql文件的路径 #执行sql文件

```

## 九丶多表查询

### 1.内连接查询

内连接查询的两个表的交集的部分。

```mysql
#隐式内连接
select e.name,e.gender,d.name from emp e, dept d where e.dept_id = d.id ;
#显示内连接
select e.name,e.gender,d.name from emp e inner join dept d on e.dept_id = d.id ;
```

### 2.外连接查询

```mysql
#左外连接，查询的是左表所有数据以及其交集部分
select e.name,e.gender,d.name from emp e left join dept d on e.dept_id = d.id ;
#右外连接
select e.name,e.gender,d.name from emp e right join dept d on e.dept_id = d.id ;
```

### 3.子查询

概念:查询中嵌套查询，称嵌套的查询为子查询。

```mysql
#查询工资最高的员工信息
select * from emp where sal = (select max(sal) from emp) ;
```

子查询的不同情况

```mysql
#子查询的结果是单行单列的，子查询可以作为条件，使用运算符运算(<,>,<=,>=,=,<>)
#示例:
select * from emp where emp.sal < (select avg(sal) from emp) ;
#子查询的结果是多行单列的，子查询可以作为条件,使用运算符in来判断
select * from emp where dept_id in (select id from dept where name = '财务部' or name = '市场部') ;
#子查询的结果是多行多列的，子查询可以作为一张虚拟表放到from后边
select * from dept t1,(select * from emp where join_date > '2011-11-11') t2 where t1.id = t2.dept_id ;
```

## 十丶事务

### 1.事务的基本介绍

​	概念:多个操作要么同时成功要么同时失败。

​	操作:

​			开启事务: start transaction;

​			回滚:rollback;

​			提交：commit;

​	事务提交的两种方式：

​			自动提交:

​					mysql 就是自动提交的

​					一条DML语句会自动提交一次事务

​			手动提交:

​					Oracle数据库默认是手动提交事务

​					需要先开启事务，再提交

​	修改事务的默认提交方式:

​			*查看事务的默认提交方式：select @@autocommit;       -- 1表示自动提交   -- 0表示手动提交

​			*修改默认提交方式： set @@autocommit = 0;

### 2.事物的四大特征

​	*原子性:是不可分割的最小操作单位，要么同时成功，要么同时失败。

​	*持久性:事务一旦提交或回滚，数据库会持久化的保存数据。

​	*隔离性:多个事务之间，相互独立。

​	*一致性:事务操作前后，数据总量不变。

### 3.事务的隔离级别

​	*概念:

​			多个事务之间隔离的，相互独立的。但是如果多个事务操作同一批数据，则会引发一些问题，设置不同的隔离级别就可以解决这些问题。

​	*存在的问题：

​		1.脏读:一个事务，读取到另一个事务中没有提交的数据。

​		2.不可重复度:在同一个事务中，两次读取到的数据不一样。

​		3.幻读:一个事务操作数据表中的记录，另一个事务添加了一条数据，则第一个事务查询不到自己的修改。

​	*隔离级别：

​		1.read uncommited:读未提交

​				*产生的问题:脏读、不可重复读、幻读

​		2.read committed：读已提交

​				*产生的问题:不可重复读、幻读

​		3.repeatable read: 可重复读   	（Mysql默认）

​				*产生的问题：幻读

​		4.serializbale：串行化

​				*可以解决所有问题

​	*查询数据库的隔离级别:

​				select @@tx_isolation ;

​	*设置数据库的隔离级别:

​				set global transaction isolation level  级别字符串;

## 十一丶管理用户/授权

### 1.管理用户

```mysql
#添加用户
create user '用户名'@'主机名' identified by '密码' ;
create user 'mantou'@'localhost' identified WITH mysql_native_password by 'mantou' ;   -- 本地登录
create user 'mantou'@'%' identified WITH mysql_native_password by 'mantou' ;   -- 远程登录
#删除用户
drop user 'mantou'@'localhost' ;
drop user 'mantou'@'%' ;
#修改用户密码
 -- 第一种方式
update user set authentication_string = password('新密码') where user = '用户名' ;
 -- 第二种方式
set authentication_string for '用户名'@'主机名' = password('新密码') ;

alter user 'root'@'localhost' identified by 'mantou' ;     # 推荐使用
alter user 'root'@'%' identified by 'mantou' ; # 推荐使用
 -- mysql中忘记了root用户的密码
 net stop mysql   #停止mysql服务
 mysqld --skip-grant-tables  #使用无验证的方式启动mysql服务
 打开新的窗口直接输入mysql 不用用户名密码就能登陆成功。
#查询用户
use mysql ;
select * from user ;     -- 通配符%表示可以在任意主机使用用户登录数据库
```

### 2.管理权限

```mysql
#查询权限
show grants for '用户名'@'主机名' ;
#授予权限
grant 权限列表 on 数据库名.表名 to '用户名'@'主机名' ;
grant all on *.* to 'mantou'@'localhost' ;    #授予所有权限
grant all on *.* to 'mantou'@'%' ;
#撤销权限
revoke 权限列表 on 数据库名.表名 from '用户名'@‘主机名’ ;
```

