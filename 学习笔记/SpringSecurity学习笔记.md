# SpringSecurity学习笔记

## 一丶用户认证

### 1.设置用户名和密码

#### **第一种方式**

```properties
#通过配置文件的方式进行配置，在application.properties中进行配置
spring.security.user.name = mantou
spring.security.user.password=mantou
```

#### **第二种方式**

```java
//新建一个配置类SecurityConfig.class
package com.mantou.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //密码进行加密处理
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String pwd = passwordEncoder.encode("mantou");
        //给用户设置用户名密码
        auth.inMemoryAuthentication().withUser("mantou").password(pwd).roles("admin");
    }
    @Bean
    PasswordEncoder password() {
        return new BCryptPasswordEncoder() ;
    }
}
```

#### **第三种方式**

```java
//编写自定义实现类
//1.首先在config包中编写配置类MySecurityConfig.class
package com.mantou.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private UserDetailsService userDetailsService;
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(password());
    }
    @Bean
    PasswordEncoder password() {
        return new BCryptPasswordEncoder() ;
    }
}
//2.在service包中编写自定义实现类
package com.mantou.service;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;
import java.util.List;

@Service("userDetailsService")
public class MyUserDetailService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        //查询数据库得到用户名密码

        //角色权限
        List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
        return new User("mantou",new BCryptPasswordEncoder().encode("123"),auths);
    }
}
```

#### 示例

自定义配置类

```java
package com.mantou.config;

import com.mantou.serviceimpl.UserDetailsServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private UserDetailsServiceImpl userDetailsServiceImpl;
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsServiceImpl).passwordEncoder(password());
    }
    @Bean
    PasswordEncoder password() {
        return new BCryptPasswordEncoder() ;
    }
    //登录设置
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()       //自定义自己编写的登录页面
        .loginPage("/login.html")   //登录页面配置
        .loginProcessingUrl("/user/login")  //登录访问路径
        .defaultSuccessUrl("/test/index").permitAll()   //登录成功之后跳转的路径
        .and().authorizeRequests()
                .antMatchers("/","/test/hello","/user/login").permitAll()   //设置哪些路径不需要认证
       .anyRequest().authenticated()
       .and().csrf().disable();   //关闭csrf防护
    }
}
```

controller代码

```java
package com.mantou.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/test")
public class TestController {
    @GetMapping("/hello")
    public String Hello(){
        return "hello security !" ;
    }

    @GetMapping("/index")
    public String index(){
        return "hello index !" ;
    }
}
```

entity代码

```java
package com.mantou.entity;


import lombok.Data;

@Data
public class Users {
    private Integer id;
    private String username ;
    private String password ;
}
```

serviceImpl代码

```java
package com.mantou.serviceimpl;

import com.mantou.entity.Users;
import com.mantou.mapper.UsersMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class UserDetailsServiceImpl implements UserDetailsService {
    @Autowired
    private UsersMapper usersMapper;
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        //查询数据库得到用户名密码
        Users user = usersMapper.findUser(s);
        if (user == null) {
            throw new UsernameNotFoundException("用户没有找到!");
        }
        //角色权限
        List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
        return new User(user.getUsername(),new BCryptPasswordEncoder().encode(user.getPassword()),auths);
    }
}
```

mapper代码

```java
package com.mantou.mapper;

import com.mantou.entity.Users;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

@Mapper
public interface UsersMapper {
    @Select("select username,password from user where username =  #{username}")
    Users findUser(String username);
}
```

application.properties配置文件

```properties
server.port=8111

#spring.security.user.name = mantou
#spring.security.user.password=mantou

#Spring
spring.datasource.url = jdbc:mysql://192.168.0.153:3306/mantou?serverTimezone=UTC&useUnicode=true&characterEncoding=UTF-8&useSSL=false
spring.datasource.username = root
spring.datasource.password = mantou
spring.datasource.driver-class-name = com.mysql.cj.jdbc.Driver
```

login.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登陆页面</title>
</head>
<body>
    <form action="/user/login" method="post" >
        用户名:<input  type="text" name="username" /> <br/>
        密码:<input type="password" name="password" /> <br/>
        <button type="submit">提交</button>
    </form>
</body>
</html>
```

## 二丶用户授权

### 1.基于角色或权限进行访问控制

对config配置类的修改

```java
package com.mantou.config;

import com.mantou.serviceimpl.UserDetailsServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private UserDetailsServiceImpl userDetailsServiceImpl;
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsServiceImpl).passwordEncoder(password());
    }
    @Bean
    PasswordEncoder password() {
        return new BCryptPasswordEncoder() ;
    }
    //登录设置
    protected void configure(HttpSecurity http) throws Exception {
        //没有权限访问，跳转到unauth.html页面
        http.exceptionHandling().accessDeniedPage("/unauth.html");
        http.formLogin()       //自定义自己编写的登录页面
        .loginPage("/login.html")   //登录页面配置
        .loginProcessingUrl("/user/login")  //登录访问路径
        .defaultSuccessUrl("/test/index").permitAll()   //登录成功之后跳转的路径
        .and().authorizeRequests()
                //设置哪些路径不需要认证
                .antMatchers("/","/test/hello","/user/login").permitAll()   
                //hasAuthority方法
                //当前登录用户，只有具有admin权限才可以访问这个路径
                //.antMatchers("/test/index").hasAuthority("admin")
                //当前登录用户，只要具有admin或manager其中一个权限就可以访问这个路径
                //hasAnyAuthority方法
                .antMatchers("/test/index").hasAnyAuthority("admin,manager")
       .anyRequest().authenticated()
       .and().csrf().disable();   //关闭csrf防护
    }
}
```

对serviceImpl的修改

```java
package com.mantou.serviceimpl;

import com.mantou.entity.Users;
import com.mantou.mapper.UsersMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class UserDetailsServiceImpl implements UserDetailsService {
    @Autowired
    private UsersMapper usersMapper;
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        //查询数据库得到用户名密码
        Users user = usersMapper.findUser(s);
        if (user == null) {
            throw new UsernameNotFoundException("用户没有找到!");
        }
        //角色权限
        List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
        return new User(user.getUsername(),new BCryptPasswordEncoder().encode(user.getPassword()),auths);
    }
}
```

## 三丶认证授权中注解的使用

### 1.@Secured

注解含义:用户具有某个角色，可以访问方法。

a.在启动类上加上@EnableGlobalMethodSecurity(securedEnabled=true)

b.在controller的方法上面使用注解，设置角色

```java
//演示使用@Secured注解的使用,需要加前缀ROLE_
@GetMapping("/update")
@Secured({"ROLE_sale","ROLE_manager"})
public String update() {
    return "hello update";
}
```

c.在serviceImpl设置用户角色

```java
//角色权限
List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("sale,ROLE_sale");
return new User(user.getUsername(),new BCryptPasswordEncoder().encode(user.getPassword()),auths);
```

### 2.@PreAuthorize

注解含义:在进入方法之前进行校验

## 四丶用户注销

在config配置类中添加以下代码

```java
//退出的路径
http.logout().logoutUrl("/logout").logoutSuccessUrl("/test/logout").permitAll();
```

## 五丶自动登录

1.注入数据源,配置操作数据库对象

在配置类中进行

```java
@Autowired
private DataSource dataSource;
@Bean
public PersistentTokenRepository persistentTokenRepository(){
    //配置数据源
    JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
    jdbcTokenRepository.setDataSource(dataSource);
    //自动创建表
    jdbcTokenRepository.setCreateTableOnStartup(true);
    return jdbcTokenRepository;
}
```

2.修改配置类,增加自动登录相关的配置

```java
  protected void configure(HttpSecurity http) throws Exception {
        //退出的路径
        http.logout().logoutUrl("/logout").logoutSuccessUrl("/test/logout").permitAll();
        //没有权限访问，跳转到unauth.html页面
        http.exceptionHandling().accessDeniedPage("/unauth.html");
        http.formLogin()       //自定义自己编写的登录页面
        .loginPage("/login.html")   //登录页面配置
        .loginProcessingUrl("/user/login")  //登录访问路径
        .defaultSuccessUrl("/success.html").permitAll()   //登录成功之后跳转的路径
        .and().authorizeRequests()
                //设置哪些路径不需要认证
                .antMatchers("/","/test/hello","/user/login").permitAll()
                //hasAuthority方法
                //当前登录用户，只有具有admin权限才可以访问这个路径
                //.antMatchers("/test/index").hasAuthority("admin")
                //当前登录用户，只要具有admin或manager其中一个权限就可以访问这个路径
                //hasAnyAuthority方法
                .antMatchers("/test/index").hasAnyAuthority("sale,manager")

       .anyRequest().authenticated()
       //配置自动登录
       .and().rememberMe().tokenRepository(persistentTokenRepository())
       //设置有效时长，以秒为单位
       .tokenValiditySeconds(60)
       .userDetailsService(userDetailsServiceImpl)
       .and().csrf().disable();   //关闭csrf防护
    }
```

3.登录页面添加自动登录的复选框,注意name属性的值必须是remeber-me

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登陆页面</title>
</head>
<body>
    <form action="/user/login" method="post" >
        用户名:<input name="username" type="text" /> <br/>
        密码:<input type="password" name="password"> <br/>
        <input type="checkbox" name="remember-me" /> 自动登录
        <button type="submit">提交</button>
    </form>
</body>
</html>
```

