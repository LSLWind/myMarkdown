## 基础

Spring Security是为基于Spring的应用程序提供声明式安全保护的安全性框架。Spring Security提供了完整的安全性解决方案，它能够在Web请求级别和方法调用级别处理身份认证和授权。

Spring Security从两个角度来解决安全性问题。它使用Servlet规范中的Filter保护Web请求并限制URL级别的访问。Spring Security还能够使用Spring AOP保护方法调用——借助于对象代理和使用通知，能够确保只有具备适当权限的用户才能访问安全保护的方法。

### 配置

**注入框架**

Spring Security借助一系列Servlet Filter来提供各种安全性功能。`DelegatingFilterProxy`是一个特殊的Servlet Filter，它本身所做的工作并不多。只是将工作委托给一个`javax.servlet.Filter`实现类，这个实现类作为一个`<bean>`注册在Spring应用的上下文中，可在web.xml中直接配置它

```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>
        org.springframework.web.filter.DelegatingFilterProxy
    </filter-class>
</filter>
```

也可以通过扩展类进行配置

```java
import org.springframework.security.web.context.
                             AbstractSecurityWebApplicationInitializer;
/**
 * Spring Security基础配置，通过扩展AbstractSecurityWebApplicationInitializer
 * Spring会自动发现该Bean并将其注入容器当中，完成Spring Security的启动
 */
public class SecurityWebInitializer extends AbstractSecurityWebApplicationInitializer {
}
```

`AbstractSecurityWebApplicationInitializer`实现了`WebApplication-Initializer`，因此Spring会发现它，并用它在Web容器中注册`DelegatingFilterProxy`。尽管我们可以重载它的`appendFilters()`或`insertFilters()`方法来注册自己选择的Filter，但是要注册`DelegatingFilterProxy`的话，我们并不需要重载任何方法。

不管我们通过web.xml还是通过`AbstractSecurityWebApplicationInitializer`的子类来配置`DelegatingFilterProxy`，它都会拦截发往应用中的请求，并将请求委托给ID为`springSecurityFilterChain` bean。

`springSecurityFilterChain`本身是另一个特殊的Filter，它也被称为`FilterChainProxy`。它可以链接任意一个或多个其他的Filter。Spring Security依赖一系列Servlet Filter来提供不同的安全特性。但是，你几乎不需要知道这些细节，因为你不需要显式声明`springSecurityFilterChain`以及它所链接在一起的其他Filter。当我们启用Web安全性的时候，会自动创建这些Filter。

**启动框架功能**

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}
```

`@EnableWebSecurity`注解将会启用Web安全功能。但它本身并没有什么用处，Spring Security必须配置在一个实现了`WebSecurityConfigurer`的bean中，或者（简单起见）扩展`WebSecurityConfigurerAdapter`。

### 基础使用

上述配置类会给应用产生很大的影响。其中任何一种配置都会将应用严格锁定。

尽管不是严格要求的，但我们可能希望指定Web安全的细节，这要通过重载`WebSecurityConfigurerAdapter`中的一个或多个方法来实现。我们可以通过重载`WebSecurityConfigurerAdapter`的三个`configure()`方法来配置Web安全性，这个过程中会使用传递进来的参数设置行为。

| 方　　法                                  | 描　　述                                |
| ----------------------------------------- | --------------------------------------- |
| `configure(WebSecurity)`                  | 通过重载，配置Spring Security的Filter链 |
| `configure(HttpSecurity)`                 | 通过重载，配置如何通过拦截器保护请求    |
| `configure(AuthenticationManagerBuilder)` | 通过重载，配置user-detail服务           |

默认的`configure(HttpSecurity)`实际上等同于如下所示：

```java
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()
      .anyRequest().authenticated()
      .and()
    .formLogin().and()
    .httpBasic();
}
```

这个简单的默认配置指定了该如何保护HTTP请求，以及客户端认证用户的方案。通过调用`authorizeRequests()`和`anyRequest().authenticated()`就会要求所有进入应用的HTTP请求都要进行认证。它也配置Spring Security支持基于表单的登录以及HTTP Basic方式的认证。

同时，因为我们没有重载`configure(AuthenticationManagerBuilder)`方法，所以没有用户存储支撑认证过程。没有用户存储，实际上就等于没有用户。所以，在这里所有的请求都需要认证，但是没有人能够登录成功。

为了让Spring Security满足我们应用的需求，还需要再添加一点配置。具体来讲，我们需要：

- 配置用户存储；
- 指定哪些请求需要认证，哪些请求不需要认证，以及所需要的权限；
- 提供一个自定义的登录页面，替代原来简单的默认登录页。

除了Spring Security的这些功能，我们可能还希望基于安全限制，有选择性地在Web视图上显示特定的内容。

但首先，我们看一下如何在认证的过程中配置访问用户数据的服务。

### 基于内存的用户认证

因为我们的安全配置类扩展了`WebSecurityConfigurerAdapter`，因此配置用户存储的最简单方式就是重载`configure()`方法，并以`AuthenticationManagerBuilder`作为传入参数。`AuthenticationManagerBuilder`有多个方法可以用来配置Spring Security对认证的支持。通过`inMemoryAuthentication()`方法，我们可以启用、配置并任意填充基于内存的用户存储。

如下，`SecurityConfig`重载了`configure()`方法，并使用两个用户来配置内存用户存储。

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception{
        //基于内存的用户认证
        auth.inMemoryAuthentication()
                .withUser("user").password("password").roles("USER").and()
                .withUser("admin").password("password").roles("USER","ADMIN");
    }
}
```

`configure()`方法中的`AuthenticationManagerBuilder`使用构造者风格的接口来构建认证配置。通过简单地调用`inMemoryAuthentication()`就能启用内存用户存储。但是我们还需要有一些用户，否则的话，这和没有用户并没有什么区别。

因此，我们需要调用`withUser()`方法为内存用户存储添加新的用户，这个方法的参数是username。`withUser()`方法返回的是`UserDetailsManagerConfigurer.UserDetailsBuilder`，这个对象提供了多个进一步配置用户的方法，包括设置用户密码的`password()`方法以及为给定用户授予一个或多个角色权限的`roles()`方法。

在上述程序中，我们添加了两个用户，“user”和“admin”，密码均为“password”。“user”用户具有USER角色，而“admin”用户具有ADMIN和USER两个角色，and()方法能够将多个用户的配置连接起来。

除了`password()`、`roles()`和`and()`方法以外，还有其他的几个方法可以用来配置内存用户存储中的用户信息。

| 方　　法                           | 描　　述                   |
| ---------------------------------- | -------------------------- |
| `accountExpired(boolean)`          | 定义账号是否已经过期       |
| `accountLocked(boolean)`           | 定义账号是否已经锁定       |
| `and()`                            | 用来连接配置               |
| `authorities(GrantedAuthority...)` | 授予某个用户一项或多项权限 |
| `authorities(List)`                | 授予某个用户一项或多项权限 |
| `authorities(String...)`           | 授予某个用户一项或多项权限 |
| `credentialsExpired(boolean)`      | 定义凭证是否已经过期       |
| `disabled(boolean)`                | 定义账号是否已被禁用       |
| `password(String)`                 | 定义用户的密码             |
| `roles(String...)`                 | 授予某个用户一项或多项角色 |



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

