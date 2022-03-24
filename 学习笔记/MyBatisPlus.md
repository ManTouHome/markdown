# MyBatisPlus学习笔记

## 一、简介

官网：http://mp.baomidou.com/

参考教程：http://mp.baomidou.com/guide/

[MyBatis-Plus](https://github.com/baomidou/mybatis-plus)（简称 MP）是一个 [MyBatis](http://www.mybatis.org/mybatis-3/) 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

## 二、特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer2005、SQLServer 等多种数据库
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 XML 热加载**：Mapper 对应的 XML 支持热加载，对于简单的 CRUD 操作，甚至可以无 XML 启动
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **支持关键词自动转义**：支持数据库关键词（order、key......）自动转义，还可自定义关键词
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作
- **内置 Sql 注入剥离器**：支持 Sql 注入剥离，有效预防 Sql 注入攻击

快速开始参考：http://mp.baomidou.com/guide/quick-start.html

测试项目： mybatis_plus

数据库：mybatis_plus

------

## 一、创建并初始化数据库

### 1、创建数据库：

mybatis_plus

### 2、创建 `User` 表

其表结构如下：

| id   | name   | age  | email              |
| ---- | ------ | ---- | ------------------ |
| 1    | Jone   | 18   | test1@baomidou.com |
| 2    | Jack   | 20   | test2@baomidou.com |
| 3    | Tom    | 28   | test3@baomidou.com |
| 4    | Sandy  | 21   | test4@baomidou.com |
| 5    | Billie | 24   | test5@baomidou.com |

其对应的数据库 Schema 脚本如下：

```mysql
DROP TABLE IF EXISTS user;
CREATE TABLE user
(
	 id BIGINT(20) NOT NULL COMMENT '主键ID',
	 name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
	 age INT(11) NULL DEFAULT NULL COMMENT '年龄',
	 email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
	 PRIMARY KEY (id)
 );
```

其对应的数据库 Data 脚本如下：

```mysql
DELETE FROM user;
INSERT INTO user (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```

## 二、初始化工程

使用 Spring Initializr 快速初始化一个 Spring Boot 工程

Group：com.atguigu

Artifact：mybatis-plus

版本：2.2.1.RELEASE

## 三、添加依赖

### 1、引入依赖

```
spring-boot-starter`、```spring-boot-starter-test
添加：mybatis-plus-boot-starter`、```MySQL、````lombok````、
在项目中使用Lombok可以减少很多重复代码的书写。比如说getter/setter/toString等方法的编写
```

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
		<exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <!--mybatis-plus-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.0.5</version>
    </dependency>
    <!--mysql-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!--lombok用来简化实体类-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>	
```

**注意：**引入 `MyBatis-Plus` 之后请不要再次引入 `MyBatis` 以及 `MyBatis-Spring`，以避免因版本差异导致的问题。

### 2、idea中安装lombok插件

**（1）idea2018版本**

![img](file:///E:/BaiduNetdiskDownload/%E5%B0%9A%E7%A1%85%E8%B0%B7/%E5%9C%A8%E7%BA%BF%E6%95%99%E8%82%B2--%E8%B0%B7%E7%B2%92%E5%AD%A6%E8%8B%91/%E9%A1%B9%E7%9B%AE%E7%AC%94%E8%AE%B0%E8%AF%BE%E4%BB%B6/day01/day01%E7%AC%94%E8%AE%B0/day01%E9%A1%B9%E7%9B%AE%E3%80%90%E9%A1%B9%E7%9B%AE%E4%BB%8B%E7%BB%8D%E5%92%8CMyBatisPlus%E5%85%A5%E9%97%A8%E3%80%91/2%20MyBatis%E5%85%A5%E9%97%A8/index_files/303533a2-0148-4329-a770-c4af5114b22c.png)

**（2）idea2019版本**

![img](file:///E:/BaiduNetdiskDownload/%E5%B0%9A%E7%A1%85%E8%B0%B7/%E5%9C%A8%E7%BA%BF%E6%95%99%E8%82%B2--%E8%B0%B7%E7%B2%92%E5%AD%A6%E8%8B%91/%E9%A1%B9%E7%9B%AE%E7%AC%94%E8%AE%B0%E8%AF%BE%E4%BB%B6/day01/day01%E7%AC%94%E8%AE%B0/day01%E9%A1%B9%E7%9B%AE%E3%80%90%E9%A1%B9%E7%9B%AE%E4%BB%8B%E7%BB%8D%E5%92%8CMyBatisPlus%E5%85%A5%E9%97%A8%E3%80%91/2%20MyBatis%E5%85%A5%E9%97%A8/index_files/ba5bc431-832b-4bcb-81fe-893d02ad3c6f.png)

## 四、配置

在 `application.properties` 配置文件中添加 MySQL 数据库的相关配置：

**注意：**driver和url的变化

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus?serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=123456
```

**注意：**

1、这里的 url 使用了 ?serverTimezone=GMT%2B8 后缀，因为Spring Boot 2.1 集成了 8.0版本的jdbc驱动，这个版本的 jdbc 驱动需要添加这个后缀，否则运行测试用例报告如下错误：

java.sql.SQLException: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more 

2、这里的 driver-class-name 使用了  com.mysql.cj.jdbc.Driver ，在 jdbc 8 中 建议使用这个驱动，之前的 com.mysql.jdbc.Driver 已经被废弃，否则运行测试用例的时候会有 WARN 信息



## 五、编写代码

### 1、主类

在 Spring Boot 启动类中添加 `@MapperScan` 注解，扫描 Mapper 文件夹

**注意：**扫描的包名根据实际情况修改

```java
@SpringBootApplication
@MapperScan("com.atguigu.mybatisplus.mapper")
public class MybatisPlusApplication {
    ......
}
```

### 2、实体

创建包 entity 编写实体类 `User.java`（此处使用了 [Lombok](https://www.projectlombok.org/) 简化代码）

```
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

查看编译结果

![img](file:///E:/BaiduNetdiskDownload/%E5%B0%9A%E7%A1%85%E8%B0%B7/%E5%9C%A8%E7%BA%BF%E6%95%99%E8%82%B2--%E8%B0%B7%E7%B2%92%E5%AD%A6%E8%8B%91/%E9%A1%B9%E7%9B%AE%E7%AC%94%E8%AE%B0%E8%AF%BE%E4%BB%B6/day01/day01%E7%AC%94%E8%AE%B0/day01%E9%A1%B9%E7%9B%AE%E3%80%90%E9%A1%B9%E7%9B%AE%E4%BB%8B%E7%BB%8D%E5%92%8CMyBatisPlus%E5%85%A5%E9%97%A8%E3%80%91/2%20MyBatis%E5%85%A5%E9%97%A8/index_files/d4e6f667-b630-474a-b743-9d4cea9731fd.jpg)

Lombok使用参考：

https://blog.csdn.net/motui/article/details/79012846 

### 3、mapper

创建包 mapper 编写Mapper 接口： `UserMapper.java`

```java
public interface UserMapper extends BaseMapper<User> {
}
```

## 六、开始使用

添加测试类，进行功能测试：

1

```
@RunWith(SpringRunner.class)
```

2

```
@SpringBootTest
```

3

```
public class MybatisPlusApplicationTests {
```

4

```

```

5

```
    @Autowired
```

6

```
    private UserMapper userMapper;
```

7

```

```

8

```
    @Test
```

9

```
    public void testSelectList() {
```

10

```
        System.out.println(("----- selectAll method test ------"));
```

11

```
        //UserMapper 中的 selectList() 方法的参数为 MP 内置的条件封装器 Wrapper
```

12

```
        //所以不填写就是无任何条件
```

13

```
        List<User> users = userMapper.selectList(null);
```

14

```
        users.forEach(System.out::println);
```

15

```
    }
```

16

```
}
```

**注意：**

IDEA在 userMapper 处报错，因为找不到注入的对象，因为类是动态创建的，但是程序可以正确的执行。

为了避免报错，可以在 dao 层 的接口上添加 @Repository 注

控制台输出：

1

```
User(id=1, name=Jone, age=18, email=test1@baomidou.com)
```

2

```
User(id=2, name=Jack, age=20, email=test2@baomidou.com)
```

3

```
User(id=3, name=Tom, age=28, email=test3@baomidou.com)
```

4

```
User(id=4, name=Sandy, age=21, email=test4@baomidou.com)
```

5

```
User(id=5, name=Billie, age=24, email=test5@baomidou.com)
```

通过以上几个简单的步骤，我们就实现了 User 表的 CRUD 功能，甚至连 XML 文件都不用编写！

#  七、配置日志

查看sql输出日志

1

```
#mybatis日志
```

2

```
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```