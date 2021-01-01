# Maven学习笔记:

## 1.Mavend的基础概念

### 1.maven的作用

1.管理jar包

​	i.增加第三方jar

​	ii.jar包之间的依赖关系

2.将项目拆分成若干个模块

### 2.maven的概念

maven是一个基于java平台的**自动化构建工具**，发展顺序：make---ant---maven---gradle

自动化构建: 原材料(java、js、html等) ----->产品(可发布的项目)

本地仓库: 自己本地的jar包仓库，如果本地有所需要的jar则直接使用，没有则去找中央仓库等

中央仓库: 连接外网的管理jar包的服务器

中央仓库镜像: 就是对中央仓库的分流操作，减轻中央仓库的压力

[中央仓库地址](https://mvnrepository.com/)

### 3.maven的主要功能

1.清理:删除编译的结果

2.编译:把Java文件编译成class文件

3.测试:针对于项目中的关键点进行测试，也可用项目中的测试代码去测试开发代码

4.报告:将测试结果进行显示

5.打包:将项目中包含的多个文件压缩成一个文件，用于安装或部署

6.安装:将打成的包放到本地仓库

7.部署:将打成的包放到服务器上准备运行

##  2.下载配置maven

a.配置JAVA_HOME

b.配置MAVEN_HOME

c.配置Path

d.配置本地仓库,找到maven目录/conf/settings.xml，设置

​	默认本地仓库:

​		${user.home}/.m2/repository

​	设置本地仓库: 

```xml
<localRepository>本地仓库目录</localRepository>
```

​	设置远程仓库镜像地址：

```xml
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```

## 3.使用maven

软件开发原则:约定>配置>硬编码

### 1.maven约定的目录结构

```目录结构
项目
└── src
|   ├── main -- 程序代码  
|    |    ├── java -- java代码
|    |    └── resources -- 资源、配置
|    └── test -- 测试代码 
|         ├── java -- java代码 
|         └── resources -- 资源、配置
└── pom.xml --项目对象模型   
```

### 2.pom.xml示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.weina</groupId>     #大项目的项目名
    <artifactId>weina</artifactId>   #子模块的名称
    <version>0.0.1-SNAPSHOT</version> #版本号
    <name>weina</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
      
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

```

### 3.maven常用命令

注意：第一次执行会下载maven的基础组件，运行mvn命令必须在有pom文件的目录下运行

1.编译程序

```maven
mvn compile   #只编译main目录中的文件，不会编译test中的文件
```

2.测试程序

```maven
mvn test
```

3.项目打包

```maven
mvn package    #把打成的jar包放到了target目录下
```

4.安装

```maven
mvn install    #将开发的模块放入本地仓库，供其他模块使用

注意:放入本地仓库的位置由pom.xml中配置的groupId，artifactId，version决定
```

5.清理

```maven
mvn clean   #删除target目录，即删除编译的文件
```

### 4.依赖

```xml
<!-- https://mvnrepository.com/artifact/junit/junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13</version>
    <scope>test</scope>   #依赖范围
</dependency>
```

1.依赖范围(scope标签的值)

|      | complie（默认） | test | provided |
| :--: | :-------------: | :--: | :------: |
| 编译 |        √        |  ×   |    √     |
| 测试 |        √        |  √   |    √     |
| 运行 |        √        |  ×   |    ×     |

2.依赖排除

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.2</version>
    <exclusions>              #依赖排除，排除依赖关系
        <exclusion>            #排除依赖的spring-beans包，即spring-beans不会引入项目中
            <groupId>org.springframework</groupId>
    		<artifactId>spring-beans</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

3.依赖的传递性

如果A依赖B，B依赖C，则如果依赖范围是compile可以依赖到C，否则依赖不到C

4.整合多个项目

如果A项目依赖于B项目，先将B项目执行install命令，把B的jar包放到本地仓库，然后再在A项目中引入B项目jar包的依赖

### 5.maven生命周期

生命周期执行顺序:a,b,c,d,e,f

当我们执行c命令，实际执行的是a,b.c命令

**生命周期包含三个阶段:**

clean lifecycle:清理

​	pre-clean     clean      post-clean

default lifecycle:默认

site lifecycle:站点

​	pre-site    site     post-site    site-deploy

 