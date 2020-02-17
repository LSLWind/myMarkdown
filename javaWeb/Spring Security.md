## Spring Boot集成Spring Security

 Spring Security，是一个基于Spring AOP和Servlet过滤器的安全框架。它提供全面的安全性解决方案，同时在Web请求级和方法调用级处理身份确认和授权。 

**maven中添加依赖**

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

spring security中默认提供了一个登陆界面，要求提供用户名和密码，初始时用户名为“user”，密码为UUID算法随机生成，也可在application.properties配置

```properties
# security
security.user.name=admin
security.user.password=admin
```

### 内存用户名密码认证

使用Spring Security需要自己进行相关配置，一般都通过基于注解的配置

* @Configuration：表示这是一个配置类，该类用于替换xml配置文件， 内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext/AnnotationConfigWebApplicationContext类进行扫描，并用于构建bean定义，初始化Spring容器。 

*  @EnableWebSecurity：开启Spring Security的功能。

* @EnableGlobalMethodSecurity(prePostEnabled = true)：开启security的注解，这样就可以在需要控制权限的方法上面使用@PreAuthorize，@PreFilter这些注解。（Spring Security默认是禁用注解的，要想开启注解，需要在继承WebSecurityConfigurerAdapter的类上加@EnableGlobalMethodSecurity注解，来判断用户对某个控制层的方法是否具有访问权限），在括号内部声明要开启的注解：

  *  @EnableGlobalMethodSecurity(securedEnabled=true)：开启@Secured 注解过滤权限 

  * @EnableGlobalMethodSecurity(jsr250Enabled=true):开启@RolesAllowed 注解过滤权限 

  * @EnableGlobalMethodSecurity(prePostEnabled=true)
         使用表达式时间方法级别的安全性 4个注解可用

    - @PreAuthorize 在方法调用之前,基于表达式的计算结果来限制对方法的访问

    - *@PostAuthorize 允许方法调用,但是如果表达式计算结果为false,将抛出一个安全性异常*
    - *@PostFilter 允许方法调用,但必须按照表达式来过滤方法的结果*
    - @PreFilter 允许方法调用,但必须在进入方法之前过滤输入值

配置类要通过继承WebSecurityConfigurerAdapter类来实现

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .inMemoryAuthentication()
                .withUser("root")
                .password("root")
                .roles("USER");
    }

}
```

方法configure()定义哪些URL需要被保护、哪些不需要。 默认配置是所有访问页面都需要认证，才可以访问。 configure(HttpSecurity http)的源码为

```java
        http
            .authorizeRequests()
                .anyRequest().authenticated()
                .and()
            .formLogin().and()
            .httpBasic();
```

通过方法名就可以看出任何请求都会先经过登录表单页面，方法 formLogin() 即定义当需要用户登录时候，转到的登录页面。 可以通过方法 configureGlobal(AuthenticationManagerBuilder auth) 在内存中存储用户名及密码，上述代码中在内存中创建了一个用户，该用户的名称为root，密码为root，用户角色为USER。 也可以配置多个用户与角色

```java
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .inMemoryAuthentication()
                .withUser("root")
                .password("root")
                .roles("USER")
            .and()
                .withUser("admin").password("admin")
                .roles("ADMIN", "USER")
            .and()
                .withUser("user").password("user")
                .roles("USER");
    }
```

### 角色权限控制

Spring Security提供了Spring EL表达式，允许我们在定义URL路径访问(@RequestMapping)的方法上面添加注解，来控制访问权限。 

