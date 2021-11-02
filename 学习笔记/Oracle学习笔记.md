# Oracle数据库学习笔记

## 1.基础概念

### 1.Oracle版本

oracle8i/9i:internet，开始走向网络

oracle10g/11g:网格计算，提高访问速度

oracle12c:cloud，云计算

各版本只是在部署运维时差异较大

### 2.Oracle服务器的特点

1. 基于关系型的数据库(RDBMS)：关系型就是二维表，通过行和列定位数据（非关系型数据库一般是键值对）
2. 组织结构：一个PGA对应一个客户端，然后传递给SGA
3. 两阶段提交:客户端提交到PGA，PGA再提交到SGA，SGA对操作进行分析合并发送给数据库，两阶段提交可以减少提交次数，减轻对数据库的压力

![](img\oracle\oracle组织结构.png)

### 3.oracle安装

1.用户:

超级管理员:sys/chang_on_install

普通管理员:system/manager

普通用户:scott/tiger

2.启动:

启动类型

OracleServiceORCL  服务名

3.登录：

普通方式登录：sqlplus 用户名/密码

DBA角色登录数据库:sqlplus 用户名/密码 as sysdba

## 2.Oracle使用

### 1.基本概念

实体:相当于java中的类

记录:相当于java的对象，就是表中的行

字段:相当于java的属性，就是表中的列

表:s行和列以及数据

基本命令:

```sql
#查看当前用户
show user; 
#查看有哪些表
select * from tab; 
#设置输出结果显示宽度
set linesize 300
#设置分页
set pages 100 ;
#设置某个字段的宽度
col 字段名 for a10     #字符
col 字段名 for 9999    #写几个9就占几位
#查看emp表的表结构
desc emp;  
#起别名，如果别名中有特殊符号需要用引号
select empno as "编号",ename "姓名",job 工作 from emp; 
#清屏
host cls 
#修改上一条命令
select * form emp ;
c /form/from
/
#编辑上一条语句
ed
#在上一条语句后追加
a order by desc ;				 #a是append追加命令
/ 
```

### 2.查询和单行函数

#### 1.范围查询

```sql
# between 小 and  大  ,一般比较数字和日期
select * from emp where sal between 2000 and 3000;
```

#### 2.模糊查询

```sql
#通配符_表示任意一个字符，%表示任意多个字符，like后可以加字符串/日期/数字
select * from emp where ename like '%C%';
#查询姓名中第二个字母是M的员工信息
select * from emp where ename like '_M%';
#姓名中包含M的员工信息
select * from emo where ename like '%M%';
#姓名长度大于6的员工信息 >=7
select * from emp where ename like '_______%';
#查询姓名包含下划线的
select * from emp where ename like '%\_%' escape '\';                                  '
#not in 不能出现null，否则查询结果集必然为空
select * from emp where deptno not in (10,20,30);
#错误实例,因为子查询的结果包含null
select * from emp mgr not in (select mgr from emp);
```

#### 3.排序

```sql
#order by 后可以写字段名，也可以写表达式/序号
#升序（默认升序）
select * from emp order by sal asc;
#降序
select * from emp order by sal desc;
#表达式,没实际意义
select empno,ename,sal from emp order by sal+1000 desc;
#序号
select empno,ename,sal from emp order by 3 desc;
#排序时，null是最大值，可以加条件把null值放到最后
select empno,ename,sal from emp order by sal desc nulls last;
#在上一条sql上追加
select * from emp ;              #注意emp后加上空格，不然追加就连在一起了
a order by desc ;				 #a是append追加命令
/                                #/是执行命令
#多列排序,先根据sal降序，如果一样，再根据hiredate降序
select * from emp order by sal desc , hiredate asc;
```

#### 4.oracle自带的函数

1. 单行函数(一次操作一行)，输入多行返回多行

   a.字符函数

   ```sql
   #注意:dual是一个单行或者单列的虚拟表。
   #大小写转换以及首字母大写
   select 'hElLo WOrlD' 原字符, lower('hElLo WOrlD') 全小写, upper('hElLo WOrlD') 全大写, initcap('hElLo WOrlD') 首字母大写 from dual ;
   #字符串截取,从第三位开始截取三位(是从1开始数，不是0)
   select substr('hello world',3,3) from dual ;
   #字符数和字节数,输出结果为 2 和 6 ，utf8格式一个汉字占三个字节，gbk一个汉字占两个字节
   select length('北京') , lengthb('北京') from dual ;
   #（了解）查看当前系统的编码格式
   select * from nls_database_parameters ;
   #在一个字符串中找另一个字符串出现的位置
   select instr('helloword','ll') from dual ;
   #填充占位,输出结果*****hello,hello*****
   select lpad('hello',10,'*') 左填充, rpad('hello',10,'*') 右填充 from dual ;
   #去掉左右空格
   select trim('    hello world     ') from dual ;
   #去掉左右任意字符
   select trim('X' from 'XXXXXXhello wordXXXXXXXXXXX') from dual ;
   #替换字符串中的字符
   select replace ('hello','l','*') from dual ;
   ```

   b.数值函数

   ```sql
   #四舍五入round(数字，位数)保留n位小数进行四舍五入
   select round(3.1231,2) 一, round(3.4512,1) 二 from dual ;
   #保留n位小数，舍尾
   select trunc(3.165,1) from dual ;
   #求余
   select mod(13,3) from dual ;
   ```

   c.日期函数

   ```sql
   #当前时间
   select sysdate from dual ;
   #日期格式化，将日期转成字符串
   select to_char(sysdate,'yyyy-mm-dd') from dual ;
   #日期可以加减数字，日期和日期只能减不能加
   select sysdate + 1 from dual ;
   selecr sysdate - 1 from dual ;
   select ename,(sysdate - hiredate) from emp ;
   #计算员工工龄，格式:入职日期，天，星期，月，年
   select ename 姓名,hiredate 入职日期 , (sysdate - hiredate) 天 , (sysdate - hiredate)/7 星期, (sysdate - hiredate)/30 月, (sysdate - hiredate)/365 年 from emp ;
   #精确计算相差的月份
   select ename,months_between(sysdate,hiredate) from emp ;
   #精确计算相差的年份
   select ename,trunc(months_between(sysdate,hiredate)/12) from emp ;
   #加月份
   select add_months(sysdate,12) from dual ;
   #当月最大是多少天
   select last_day(sysdate) from dual ;
   #下一个星期几是那一天，只能写星期几而不能写周几
   select next_day(sysdate,'星期五') from dual ;
   #对时间四舍五入以及舍尾
   select round(sysdate,'month'), round(sysdate,'year') from dual ;
   ```

   d.转换函数

   ```sql
   #将字符串数值转换为数值
   select to_number('12')+to_number('13') from dual ;
   #以指定的格式展示工资
   select to_char(sal,'000,0') from emp ;
   #将字符串日期转换成日期
   select to_date('2021-01-01 13:24:12','yyyy-mm-dd HH24:mi:ss') from dual ;
   ```

   e.通用函数

   ```sql
   #如果oracle第一个参数为空那么显示第二个参数的值，如果第一个参数的值不为空，则显示第一个参数本来的值。
   select ename,NVL(comm,-1) from emp ;
   #如果该函数的第一个参数为空那么显示第二个参数的值，如果第一个参数的值不为空，则显示第三个参数的值。
   select ename,NVL2(comm,-1,1) from emp ;
   #如果两个字符串相等返回null，不为null返回第一个字符串
   select nullif('hello','world') from dual ;
   #从左往后找第一个不为null的值
   select comm,sal,coalesce(comm,sal) from emp ;
   #decode(字段,条件1，返回值1，条件2，返回值2，....,最后表达式)类似ifelse
   select ename,job,sal 涨前,decode(job,'MANAGER',NVL(sal + 1000,1000),'PRESIDENT',NVL(sal + 2000,2000), NVL(sal + 200,200))涨后 from emp ;
   ```

2. 多行函数又叫组函数或聚合函数(一次操作多行)，输入多行返回一行

   ```sql
   #求有几个部门，count函数会自动排除null
   select count(distinct deptno) from emp ;
   #计算员工的工资
   select count(*) 员工总数, sum(sal) 总工资, max(sal) 最大工资,min(sal) 最小工资, avg(sal) 平均工资 from emp ;
   ```

   分组:

   ```sql
   #每个部门的平均工资
   select deptno，avg(sal) 平均工资 from emp group by deptno ;
   #先根据部门分组，如果部门一样再根据job分组,就是求各个部门中各个工作的平均工资
   select deptno,job,avg(sal) from emp group by deptno,job ;
   #对行筛选用where，对组筛选用having，having后可以使用组函数，where后边不能用组函数
   select deptno,job,avg(sal) from emp group by deptno,job  having avg(sal) > 3000;
   ```

   **注意:分组查询时，不在分组函数中的列，必须在group by中。**

### 3.查询语句的注意事项

1.select控制列，where控制行

2.字符/字符串/日期：需要用**单引号**引起来，不能用双引号

3.关键字对大小写不敏感，但是对于数据严格区分大小写

4.如果查找null，不能用=或!=或者<>,必须使用is或者is not

5.where条件的执行顺序是从右往左执行，例如 where mgr = 7788 and job = 'CLEARK' 会先执行job语句

6.关于null的其他问题：

​	null的计算:任何数字加上null结果为null。可以使用NVL/NVL2解决

7.对查询结果去重:在要去重的字段前加上distinct

8.连接符:concat或者 ||

```sql
#连接符
select concat('hello','world') from dual ;
select 'hello' || 'world' from dual ;
```

9.日期:

```sql
#修改oracle默认的日期格式
 select * from v$nls_parameters ;
 alter session set NLS_DATE_FORMAT = 'yyyy-mm-dd' ;
```

### 4.SQL语句的分类

DQL：数据库查询语言select

DML:   数据操作语言update insert delete 

DDL:    数据定义语言create/drop  /truncate /alter table

DCL:     数据控制语言 grant,revoke

### 5.DML数据操作语言

#### 1.增加语句

```sql
#增加数据insert
#格式:insert into 表名(字段名1,字段名2,.....) values(字段值1,字段值2,......)
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno) values(8888,'ManTou','Java',7698,'5-1月-21',10000,500,10) ;
#SQL99标准可以省略字段名，但是插入的数据必须是完整的字段且顺序也必须一致
insert into emp values(8888,'ManTou','Java',7698,'5-1月-21',10000,500,10) ;
#如果插入的数据不完整，可以写上部分字段名
insert into emp(empno,ename,job) values(8888,'ManTou','Java') ;
#动态输入插入的值(&),但是输入的字符串/日期也可要加单引号
insert into emp(empno,ename,job) values(&empno,&ename,&job) ;
#批量插入数据,创建新表(批量插入之前是不存在的)
create table mytab as select * from emp ;
#快速创建表结构
create table mytab2 as selecr * from emp where 1 = 0 ;
#批量插入数据,在旧表中插入(已经存在的表)
insert into mytab2(empno,ename,sal) select empno,ename,sal from emp ;
#批量插入数据，使用事务
begin
 insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno) values(1234,'ManTou','Java',7698,'5-1月-21',10000,500,10) ;
 insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno) values(1235,'ManTou','Java',7698,'5-1月-21',10000,500,10) ;
end ;
/
#插入海量数据,可以通过数据泵\SQL Loader\外部表
```

#### 2.删除语句

```sql
#格式:delete from 表名 ;
#删除表中的全部数据,少量数据delete效率高,海量数据truncate效率高
delete from mytab2 ;  #不能回退  原理:一行一行的删除，支持闪回（只是换了一个地方存,相当于放到回收站了）
truncate table 表名 ; #不能回退，不属于DML   原理:删除整个表，重新创建一张表，不支持闪回（相当于清空回收站）
#事务回退
rollback ;
#按条件删除
delete from emp where empno = 1235 ;
#测试执行时间
#打开执行时间
set timing on
#关闭执行时间
set timing off
#整理碎片,方法一
alert table emp move ;
#整理碎片,方法二,导出导入
```

delete与truncate的区别:

1.delete能过够回退,truncate不能回退

2.少量数据delete效率高,海量数据truncate效率高

3.delete一行一行的删除，truncate删除整个表，重新创建一张表

4.delete支持闪回（只是换了一个地方存,相当于放到回收站了）,truncate不支持闪回（相当于清空回收站）

5.delete会产生碎片，truncate不会产生碎片

#### 3.修改语句

```sql
#格式:update 表名 set 字段名1 = 字段值1，字段名2= 字段值2，... where  ...
update emp set ename = 'mt' where empno = '9999' ;
```

### 6.DDL数据库定义语言

#### 1.创建表

```sql
#创建表
create table mytab3 (
	id number ,
    name varchar2(10) ,
    age number 
) ;
```

注意事项:

​	1.权限和空间问题

​	2.表名的规定：

​		a.表名必须以字母开头不能是数字

​		b.表名只能包含:大小写字母、数字、_ 、$、#

​		c.长度1-3个字符

​		d.不能与数据库中的其他对象重名(表，视图，索引，触发器，存储过程....)

​		e.不能与保留字重名

​				查看保留字:DBA账户

​				select * from v$reserved_words order by keyword asc ;

#### 2.修改表

```sql
#追加新列
alter table mytab3 add phone varchar2(10) ;
#修改列的长度
alter table mytab3 modify phone varchar2(20) ;
#修改列的类型
alter table mytab3 modify phone number ;
#删除列
alter table mytab3 drop column phone ;
#重命名列
alter table mytab3 rename column phone to phone_num ;
```

注意事项:blob/clob类型不能修改，如果非要修改，要先删再加

#### 3.删除表

```sql
#删除表,放到了回收站
drop table mytab3 ;
#彻底删除
drop table mytab3 purge ;
#查看回收站
show recyclebin ;
#清空回收站
purge recyclebin ;
#还原回收站,闪回技术
```

### 7.多表连接查询

#### 1.交叉连接/笛卡尔积

```sql
#所有情况的组合，不推荐使用
select * from emp,dept ;
```

#### 2.内连接

多张表通过相同字段进行匹配，只显示匹配成功的数据

```sql
#第一种写法
select * from emp e,dept d where e.deptno = d.deptno ;
#第二种写法
select * from emp 3 inner join dept d on e.deptno = d.deptno;
```

#### 3.外连接

```sql
#左外连接
#以左表为基准，左表的数据全部显示，去匹配右表的数据，如果匹配成功则全部显示，匹配不成功，无数据部分用NULL填充
#第一种写法,oracle独有
select * from emp e,dept d where e.deptno = d.deptno(+) ;
#第二种写法
select * from emp e left join dept d on e.deptno = d.deptno ;
#右外连接
#以右表为基准，右表的数据全部显示，去匹配左表的数据，如果匹配成功则全部显示，匹配不成功，无数据部分用NULL填充
#第一种写法,oracle独有
select * from emp e,dept d where e.deptno(+) = d.deptno ;
#第二种写法
select * from emp e right join dept d on e.deptno = d.deptno ;
#全外连接
#全外连接 = 左外 + 右外 + 去重 ;
select * from emp e full join dept d on e.deptno = d.deptno ;
```

#### 4.自连接

```sql
#将一张表通过别名视为不同的表
#查询员工姓名和他的上级领导的姓名
select a.ename,b.ename from emp a,emp b where a.empno = b.mgr ;
```

注意:自连接比较费性能，因为是把一张表当作了两张表,可以使用层次连接优化。

#### 5.层次连接

```sql
select level,empno,ename,mgr from emp 
connect by prior empno = mgr          #下层=上层
start with mgr is null               #从哪开始，mgr = n ,n是上层节点的值
order by level ;
```

缺点:需要验证

### 8.子查询

```sql
#比scott员工工资高的员工信息
select * from emp where sal > (select sal from emp where ename = 'SCOTT') ;
#在select后用子查询，但子查询返回的是一个值而不是多个,就一个常量列
select empno 一,ename 二,(select job from emp where empno =7369) 三 from emp ;
#可以用这个代替上边的sql语句
select empno 一,ename 二,'CLARK'  三 from emp ;
#在having后边用子查询
#查询最低工资比30号部门的最低工资高的部门编号
select deptno,min(sal) from emp group by deptno  having min(sal) > (select min(sal) from emp where deptno = 30) ;
#在from后使用子查询，相当于修改了表结构
select * from (select empno,ename,sal*12 from emp) ;
#不是同一张表的子查询
#查询销售部的员工信息
select * from emp where deptno = (select deptno from dept where dname = 'SALES') ;
#如果问题中出现满足一个即可或者存在一个就可以可以使用any
select * from emp where sal > any(select sal from emp where deptno = 30) ; 
#如果问题中出现全部就可以可以使用all
select * from emp where sal > all(select sal from emp where deptno = 30) ;
#子查询结果中包含null
#查询不是领导的员工信息(子查询时进行排空操作)
select * from emp where empno not in (select mgr from emp where mgr is not null ) ;
```

注意:

​	1.子查询可以出现的位置:select、from、where、having，不能写在group by后边

​	2.主查询和子查询可以是或不是同一张表

​	3.子查询中也可以使用单行(=,<,>)或多行操作符(in)

​	4.子查询中null的问题，xxx != null会一条数据都查不出来，null只能用is/is not，最好子查询的结果不要有null,如果子查询结果中有null，要进行排空

​	5.一般不在子查询中排序，除非TOP-N问题(分页)

### 9.伪列

伪列:不属于任何一张表，但是会被所有的表共享

#### 1.rownum逻辑序列

特点:不能sql语句在执行时rownum的值不一致

​		在相同sql语句执行时，rownum的值不变（第一次查询产生rownum，之后不变）

​		可以用于删除重复数据

```sql
#查询工资最高的前3条员工信息
#错误案例
select ename,sal from emp where rownum <= 3 order by sal desc  ;
#错误原因：排序之前查询出rownum，之后就会保持不变，再进行排序会产生临时表，但是rownum不会变，所以取到的值是序号123的数据而不是前三个。
#正确做法
#用子查询解决
select rownum,ename,sal from (select * from emp order by sal desc nulls last) where rownum <= 3 ;
```

​		top-n：前n个数据解决套路:rownum+子查询排序

​		select rownum,..... from (select * from xxx oder by ... desc) where rownum <= n ;

#### 2.rowid物理序列

特点：根据插入的顺序依次递增，一共18位，前6位：数据对象编号

​	      往后数三位：数据文件编号

​		 再后六位：数据块编号

​		最后三位：行号

删除重复数据思路:

​	根据编号分组(将重复数据放到一组，然后在每组中保留一个即可)

```sql
#删除重复数据
delete from student where rowid not in (select min(rowid) from student group by stuno );
```

## 3.习题

技巧:不要进行阅读理解，而是进行翻译。

```sql
1.查询所有员工的年工资、所在部门的名称，按年薪从低往高排序。
select d.dname 部门名称 , e.sal*12 + nvl(e.comm,0) 年工资 from emp e , dept d where e.deptno = d.deptno order by 年工资 ;
2.查询所有员工的编号、姓名，及其上级领导的编号、姓名。显示结果按领导的年工资降序。
select e.empno 员工编号, e.ename 员工姓名 ,m.empno 领导编号 ,m.ename 领导姓名 from emp e left join emp m on e.mgr = m.empno order by m.sal*12 + nvl(m.comm,0) desc ;

select e.empno 员工编号, e.ename 员工姓名 ,m.empno 领导编号 ,m.ename 领导姓名 from emp e , emp m where e.mgr = m.empno(+) order by m.sal*12 + nvl(m.comm,0) desc ;
3.查询非销售人员的工作名称，以及从事同一工作员工的月工资之和，要求月工资之和大于5000，输出结果按月工资之和降序排列。
select job 工作名称,sum(sal) 员工工资之和 from emp group by job having sum(sal) > 5000 and job != 'SALESMAN' order by 员工工资之和 desc ;
4.查询所有领取奖金和不领取奖金的员工人数、平均工资。
select count(*) 员工人数,avg(sal) 平均工资 from emp where comm is not null and comm > 0 
union
select count(*) 员工人数,avg(sal) 平均工资 from emp where comm is null or comm = 0 ;
5.查询总工资、各个部门的总工资、各个部门中各个工作的总工资
select null,null,sum(sal) from emp 
union
select deptno,null,sum(sal) from emp group by deptno
union
select deptno,job,sum(sal) from emp group by deptno,job ;
#使用增强order by
select deptno,job,sum(sal) from emp group by rollup(deptno,job) ;
6.查询每种工作的最低工资、以及领取该工资的员工姓名。
#全值连接,模拟外键=主键
select e.ename,t.minsal,t.job from emp e,(select min(sal) minsal,job from emp group by job) t where t.minsal = e.sal and t.job = e.job ;
7.查询出工资不超过2500的人数最多的部门名称。
#查询工资超过2500最多的人数
select max(count(*)) from emp where sal <= 2500 group by deptno ;
#查询出工资不超过2500的人数最多的部门名称
select d.deptno,d.dname from emp e,dept d where e.deptno = d.deptno group by d.deptno ,d.dname having count(*) = (select max(count(*)) from emp where sal <= 2500 group by deptno) ;
8.查询出管理员工人数最多的人的名字和他管理的人的名字。
#查询管理最多的人的管理人数
select max(count(*)) from emp group by mgr ;
#查询出管理员工人数最多的人的编号
select mgr from emp group by mgr having count(*) = (select max(count(*)) from emp group by mgr) ;
#查询出管理人数最多的领导名字和编号
select ename,empno from emp where empno = (select mgr from emp group by mgr having count(*) = (select max(count(*)) from emp group by mgr)) ;
#查询出管理人人数最多的领导的员工姓名和领导编号
select ename,mgr from emp where mgr = (select mgr from emp group by mgr having count(*) = (select max(count(*)) from emp group by mgr)) ;
#最终语句
select * from (select ename,empno from emp where empno = (select mgr from emp group by mgr having count(*) = (select max(count(*)) from emp group by mgr))) t1 , (select ename,mgr from emp where mgr = (select mgr from emp group by mgr having count(*) = (select max(count(*)) from emp group by mgr))) t2 where t1.empno = t2.mgr ;
#简化
select b.ename ,e.ename from emp e,emp b where e.mgr = b.empno and e.mgr = (select mgr from emp group by mgr having count(*) = (select max(count(*)) from emp group by mgr)) ;
```

交并补

使用联合查询，列数和类型必须一致。

1.union（并集）：返回各个查询的所有记录，不包括重复记录。

2.union all(并集)： 返回各个查询的所有记录，包括重复记录。

3.intersect(交集)：返回两个查询共有的记录。

4.minus（补集）：返回包含在第一个查询中，但不包含在第二个查询中的记录。

增强group by：

​	group by rollup (a,b);相当于:

​					group by a,b;

​					group by a；

​					group by null；

## 4.约束

### 1.六种约束

#### 1.检查约束(check)

#### 2.唯一约束(unique)

​	唯一，可以为null

#### 3.主键约束(primary key) 

​	主键唯一，不能为null

#### 4.外键约束(foreign key)

#### 5.非空约束(not null)

#### 6.默认约束(default)

### 2.约束命名

​	规范：constraint 约束类型_字段名

​	主键约束: PK_字段名

​	检查约束: CK_字段名

​	唯一约束: UQ_字段名

​	非空约束: NN_字段名

​	外键约束: FK_子表`_`父表

​	默认约束: 一般不需要命名

### 3.列级约束

```sql
#创建带约束的表
create table stu(
	stuno number(3) primary key ,    #主键约束
    stuname varchar2(10) not null unique , #非空且唯一
    stuaddress varchar2(20) default '北京朝阳' check(length(stuaddress)>2),#默认约束和唯一约束
    stuid number(3)
) ;
#测试默认约束,default 代表默认约束的值
insert into student values(1,'xx',default,1) ;
```

注意:

​	1.如果有多个约束，default必须放在第一位。

​	2.check约束和编写规则和where一致。

​	3.唯一约束可以是null但是不适用于null。

​	4.约束名是多个表公用的(多个表的约束名不能重复)

### 4.表级约束

表级约束没有默认和非空

```sql
#表级约束
create table student2(
	stuno number(3),
    stuname varchar2(10),
    stuaddress varchar2(20),
    stubid number(3),
    constraint PK_sno primary key(stuno),
    constraint UQ_sname_subid unique(stuname,stubid),
    constraint CK_saddress check(length(stuAddress) > 2)
)
```

### 5.外键约束

#### 1.外键使用

外键含义: a ->b字段，a是外键，a中的数据必须来源于b中。

```sql
#指定外键
create table student(
	stuno number(3) primary key,
    stuname varchar2(10),
    stuaddress varchar2(20),
    subid number(3) references sub(sid)    #外键约束
) ;
#创建课程表
create table sub(
	sid number(3) primary key,
    sname varchar2(10)
) ;
```

如果删除父表中外键所指向的列，2个策略:级联删除|级联置空

```sql
#级联删除,当删除父表中的数据时，子表会跟着删除相对应的数据
create table student(
	stuno number(3) primary key,
    stuname varchar2(10),
    stuaddress varchar2(20),
    subid number(3) references sub(sid) on delete cascade    #级联删除
) ;
#级联置空，当删除父表中的数据时，子表会把相对应的数据置空
create table student(
	stuno number(3) primary key,
    stuname varchar2(10),
    stuaddress varchar2(20),
    subid number(3) references sub(sid) on delete set null    
) ;
```

建议:

​	1.父表中没有相对应的数据时，不要向子表增加数据

​	2.不要更改父表的数据，导致子表孤立

​	3.创建外键时，直接设置成级联删除或级联置空

​	4.如果删除表先删除子表再删除父表

#### 2.追加约束

```sql
1.唯一、主键、检查、外键约束
alter table 表名 add constraint 约束名 约束类型
alter table student add constraint UQ_stuname unique(stuname) ;
2.默认、非空
alter table 表名 motidy 约束名 约束类型
```

#### 3.删除约束

```sql
#删除约束,根据约束名
alter table 表名 drop constraint 约束名 ;
#删除约束,删除默认约束
alter table 表名 modify 字段名 default null ;
```

### 6.完整性约束

保证数据的正确性、相容性、防止数据冗余等。

域完整性: 列

实体完整性：行

引用完整性：不同表之间

自定义完整性：触发器

## 5.三大范式NF

1.确保每列的原子性（不可再分）

2.宏观:每张表只描述一件事情

微观：除了主键以外的其他字段，都依赖于主键

3.微观:除了主键以外的其他字段，都不传递依赖于主键

注意:三大范式不必严格遵守。

