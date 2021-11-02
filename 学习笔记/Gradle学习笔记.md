# Gradle学习笔记

## 1.Gradle基础概念

### 1.Gradle介绍

[Gradle下载地址](https://gradle.org/releases/)

[Gradle中文文档](https://dongchuan.gitbooks.io/gradle-user-guide-/content/)

Gradle是一个开源的自动化构建工具，建立在Ant和Maven的概念基础上，并引入了基于Groovy的特定领域语言。

### 2.Groovy简介

[Groovy下载地址](https://groovy.apache.org/download.html)

Groovy时基于JVM虚拟机的一种敏捷的动态语言，他是一种成熟的OOP编程语言，既可以用于面向对象，又可以用作纯粹的脚本语言，同时具有闭包和动态语言中的其他特性。

#### 1.与java的不同点

1. 完全兼容java语法，可做脚本也可做类
2. 分号是可选的，但是一般不加分号
3. 类，方法，字段都是公开的，没有访问权限限制
4. 默认生成名值对
5. 字段不定义访问权限时，编译器自动给字段添加getter/setter方法
6. 字段可以用点来获取，也可用set/get
7. 方法可省略return，自动检索最后一行的结果作为返回值
8. 空值比较不会有空指针异常

#### 2.Groovy的高级特性

1. assert断言：可以用assert代替之前的Java断言语句
2. 可选类型：可使用JS弱类型，使用def来表示任意类型
3. 方法调用：调用带参方法时可以省略括号
4. 字符串定义：单引号，双引号，三个单引号
5. 集合API：集合的定义和使用更加简单，但是也兼容Java的API
6. 闭包：代码块可以赋值给一个变量也可以作为一个参数传递给一个方法

#### 3.代码展示与Java的不同点

Student.groovy:

```groovy
//首先定义一个Student.groovy类
package com.mantou.groovy
//不用权限修饰符，默认使用的时public
class Student {     //可以省略getter/setter
    private String userName   //可省略分号
    private String email
    int age   //无权限修饰时，默认生成getter/setter
    String getUserName(){
        userName     //可以省略return
    }
    void setUserName(String userName){
        this.userName = userName
    }
}
```

Test.groovy:

```groovy
package com.mantou.groovy
//不需要定义类，以脚本形式书写代码
Student student = new Student()
//调用getter/setter方法
student.setUserName("馒头")
println student.getUserName()
//使用.的方式来赋值和获取字段值,s设置为私有的字段也能通过点获取
student.email = 'ManTou'
println student.email
//调用无权限修饰符的Setter/Getter
student.setAge(10)
println student.getAge()
//调用默认带有的具名构造器,def可以表示任何类型和var类似
def student2 = new Student(userName:'大馒头',email: '150@163.com',age:22)
println student2.getUserName()
```

测试类运行结果:

![](img\gradle\与Java的区别运行结果.png)

Student.groovy的简写:

```groovy
package com.mantou.groovy

class student2 {
    //可以直接简写成这样,不用权限修饰，也不用定义类型，会自动生成Setter/Getter
    //def也可以省略
    def userName
    def email
    def age
}
```

#### 4.代码展示Groovy的高级特性

Grammar.groovy：

```groovy
package com.mantou.groovy
println '====================基本语法定义========================='
//变量的声明
def name = 'ManTou'       //也可以把def省略
age = 20
//调用带参数的方法时可以省略括号
println(name + "," + age)
println name + "," + age
//不会有空指针异常
age = null
println age.equals(null)
//断言assert
//assert age == 19
println '====================字符串的定义========================='
str1 = '字符串1'     //定义普通字符串
str2 = "字符串2,${str1}"     //可以引用变量
//按格式定义字符串
str3 = '''
name:ManTou
age:20
'''
println str1
println str2
println str3
println '====================集合的定义========================='
/**
 * List集合，使用[]来声明集合
 */
list = ['mantou','zhangsan','lisi']
//给list集合添加元素
list.add('wangwu')    //Java语法添加元素
list << 'zhaoliu'     //groovy语法添加元素
println list
/**
 * Map映射，使用[key:value]
 */
map = ['username':'mantou','age':18]
//给map赋值
map.put('email','150@163.com')
//使用点key的方式赋值
map.gender='boy'
println map
println '====================闭包的定义========================='
c1 = {
    println "不带参数的闭包"
}
c2 = {
    val->
        println "带参数的闭包：${val}"
}
//闭包是给方法使用的
//定义指定类型接受不能带参数的闭包的方法,方法的def不能省略
def method1(Closure clo){
   // clo()      //java语法
    clo.call()   //groovy语法
}
//定义不指定类型接受带参数的闭包的方法
def method2(closure){
   // closure("mantou")
    closure.call("mantou")
}
//调用方法来传入已经定义好的闭包
method1 c1
method2 c2
//调用方法时直接定义新的闭包作为实际参数
method1 {println '直接定义闭包'}
method2 {val-> println "直接定义带参数的闭包:${val}"}
```

运行结果:

![](img\gradle\groovy特性运行结果.png)

## 2.Gradle的使用

gradle版本和IDEA版本要匹配，尽量IDEA版本在gradle版本之后

build.gradle

```groovy
plugins {
    id 'java'
}
group 'com.mantou'
version '1.0-SNAPSHOT'
/**
 * 指定所使用的仓库
 * mavenCentral()  表示中央仓库
 * mavenLocal()  表示本地仓库
 */
repositories {
    // mavenCentral()
    mavenLocal()
    maven {
        url 'http://maven.aliyun.com/nexus/content/groups/public/'
    }
}
/**
 * gradle的作用域
 */
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.6.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine'
    // https://mvnrepository.com/artifact/org.mybatis/mybatis
    compile group: 'org.mybatis', name: 'mybatis', version: '3.5.6'
    // https://mvnrepository.com/artifact/org.springframework/spring-context
    compile group: 'org.springframework', name: 'spring-context', version: '5.3.2'
}

test {
    useJUnitPlatform()
}
```

Gradle的继承和聚合:

父工程：

```groovy
allprojects{      #使用allprojects被所有子模块继承
    apply plugin: 'java'
    group 'com.mantou'
    version '1.0-SNAPSHOT'

    repositories {
        mavenLocal()
        maven {
            url 'http://maven.aliyun.com/nexus/content/groups/public/'
        }
    }

    dependencies {
        testImplementation 'org.junit.jupiter:junit-jupiter-api:5.6.0'
        testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine'
    }

    test {
        useJUnitPlatform()
    }
}
```

子工程1:

```groovy
dependencies {
    compile project (":gradledao")    #依赖gradledao子模块
}
```

settings.gradle

```groovy
rootProject.name = 'gradleparent'
include 'gradlechild'
include 'gradledao'
```



