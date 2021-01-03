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
#查看emp表的表结构
desc emp;  
#起别名，如果别名中有特殊符号需要用引号
select empno as "编号",ename "姓名",job 工作 from emp; 
#清屏
host cls 
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

 