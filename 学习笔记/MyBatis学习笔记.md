#  MyBatis 学习笔记

## 一丶入门实例

### 项目结构

![](E:\JavaNote\img\MyBatis\项目结构.png)

### pom. xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>MyBatisDemo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.7</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.26</version>
        </dependency>


        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>com/mantou/mapper/*.xml</include>
                </includes>
            </resource>
        </resources>
    </build>

</project>
```



### PersonMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mantou.mapper.PersonMapper">
    <select id="selectById" parameterType="int" resultType="com.mantou.entity.Person">
        select * from person where id = #{id}
    </select>
</mapper>
```

### conf.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--    指定使用开发环境还是测试环境-->
    <environments default="development">
    <!--        开发环境配置-->
        <environment id="development">
<!--            事务的提交方式-->
            <transactionManager type="JDBC"/>
<!--            数据源类型：-->
<!--            unpooled：传统的JDBC模式-->
<!--            pooled:使用数据库连接池-->
<!--            JNDI:从tomcat中获取一个内置的数据库连接池-->
            <dataSource type="POOLED">
<!--                数据库配置信息-->
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://192.168.0.153:3306/mantou?serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="mantou"/>
            </dataSource>
        </environment>
    <!--测试环境配置-->
        <environment id="Test">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://192.168.0.153:3306/mantou?serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="mantou"/>
            </dataSource>
        </environment>


    </environments>
    <mappers>
        <mapper resource="com/mantou/mapper/PersonMapper.xml"/>
    </mappers>
</configuration>
```

### PersonTest

```Java
  import com.mantou.entity.Person;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;
import java.io.IOException;
import java.io.Reader;


public class PersonTest {
    @Test
    public void PersonTest01() throws IOException {
        //加载MyBatis配置文件
        Reader resourceAsReader = Resources.getResourceAsReader("conf.xml");
        //SqlSessionFactory -- connection
        SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(resourceAsReader);
        String statement = "com.mantou.mapper.PersonMapper.selectById";
        SqlSession sqlSession = ssf.openSession();
        Person person = sqlSession.selectOne(statement, 1);
        System.out.println(person);
        sqlSession.close();
        resourceAsReader.close();
    }
}
```

### MyBatis约定

1.输入参数parameterType和输出参数resultType ,在形式上只能有一个

2.如果输入参数是简单类型（8个基本类型+String）是可以使用任何占位符#{xxxx}

## 二丶基于动态代理的CRUD

原则：约定优于配置



