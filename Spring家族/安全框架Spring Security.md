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

### 基于数据库表进行认证

用户数据通常会存储在关系型数据库中，并通过JDBC进行访问。为了配置Spring Security使用以JDBC为支撑的用户存储，我们可以使用`jdbcAuthentication()`方法，所需的最少配置如下所示：

```java
@Autowired
DataSource dataSource;

@Override
protected void configure(AuthenticationManagerBuilder auth)
                                                   throws Exception {
  auth
    .jdbcAuthentication()
      .dataSource(dataSource);
}
```

我们必须要配置的只是一个`DataSource`，这样的话，就能访问关系型数据库了。在这里，`DataSource`是通过自动装配的技巧得到的。

每当用户进入系统时，都会通过默认的sql语句进行认证与鉴权，Spring Security中默认的SQL为

```java
public static final String DEF_USERS_BY_USERNAME_QUERY =
        "select username,password,enabled " +
        "from users " +
        "where username = ?";
public static final String DEF_AUTHORITIES_BY_USERNAME_QUERY =
        "select username,authority " +
        "from authorities " +
        "where username = ?";
public static final String DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY =
        "select g.id, g.group_name, ga.authority " +
        "from groups g, group_members gm, group_authorities ga " +
        "where gm.username = ? " +
        "and g.id = ga.group_id " +
        "and g.id = gm.group_id";
```

在第一个查询中，我们获取了用户的用户名、密码以及是否启用的信息，这些信息会用来进行用户认证。接下来的查询查找了用户所授予的权限，用来进行鉴权，最后一个查询中，查找了用户作为群组的成员所授予的权限。

如果你能够在数据库中定义和填充满足这些查询的表，那么基本上就不需要你再做什么额外的事情了。但是，也有可能你的数据库与上面所述并不一致，那么你就会希望在查询上有更多的控制权。如果是这样的话，我们可以按照如下的方式配置自己的查询：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
                                                   throws Exception {
  auth
    .jdbcAuthentication()
      .dataSource(dataSource)
      .usersByUsernameQuery(
        "select username, password, true " +
        "from Spitter where username=?")
      .authoritiesByUsernameQuery(
        "select username, 'ROLE_USER' from Spitter where username=?");
}
```

在本例中，我们只重写了认证和基本权限的查询语句，但是通过调用`group-AuthoritiesByUsername()`方法，我们也能够将群组权限重写为自定义的查询语句。

将默认的SQL查询替换为自定义的设计时，很重要的一点就是要遵循查询的基本协议。所有查询都将用户名作为唯一的参数。认证查询会选取用户名、密码以及启用状态信息。权限查询会选取零行或多行包含该用户名及其权限信息的数据。群组权限查询会选取零行或多行数据，每行数据中都会包含群组ID、群组名称以及权限。

### 拦截请求

对每个请求进行细粒度安全性控制的关键在于重载`configure(HttpSecurity)`方法。如下的代码片段展现了重载的`configure(HttpSecurity)`方法，它为不同的URL路径有选择地应用安全性：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()//几种不同颗粒度的鉴权方式，按顺序定制访问需要
      .antMatchers("/spitters/me").authenticated()
      .antMatchers(HttpMethod.POST, "/spittles").authenticated()
      .anyRequest().permitAll();
}
```

`configure()`方法中得到的`HttpSecurity`对象可以在多个方面配置HTTP的安全性。在这里，我们首先调用`authorizeRequests()`，然后调用该方法所返回的对象的方法来配置请求级别的安全性细节。其中，第一次调用`antMatchers()`指定了对“/spitters/me”路径的请求需要进行认证。第二次调用`antMatchers()`更为具体，说明对“/spittles”路径的HTTP POST请求必须要经过认证。最后对`anyRequests()`的调用中，说明其他所有的请求都是允许的，不需要认证和任何的权限。

`antMatchers()`方法中设定的路径支持Ant风格的通配符。在这里我们并没有这样使用，但是也可以使用通配符来指定路径，如下所示：

```
.antMatchers("/spitters/**").authenticated();
```

我们也可以在一个对`antMatchers()`方法的调用中指定多个路径：

```
.antMatchers("/spitters/**", "/spittles/mine").authenticated();
```

`antMatchers()`方法所使用的路径可能会包括Ant风格的通配符，而regexMatchers()方法则能够接受正则表达式来定义请求路径。例如，如下代码片段所使用的正则表达式与“/spitters/`**`”（Ant风格）功能是相同的：

```
.regexMatchers("/spitters/.*").authenticated();
```

除了路径选择，我们还通过`authenticated()`和`permitAll()`来定义该如何保护路径。`authenticated()`要求在执行该请求时，必须已经登录了应用。如果用户没有认证的话，Spring Security的Filter将会捕获该请求，并将用户重定向到应用的登录页面。同时，`permitAll()`方法允许请求没有任何的安全限制。

除了`authenticated()`和`permitAll()`以外，还有其他的一些方法能够用来定义该如何保护请求。

| 方　　法                     | 能够做什么                                                   |
| ---------------------------- | ------------------------------------------------------------ |
| `access(String)`             | 如果给定的SpEL表达式计算结果为true，就允许访问               |
| `anonymous()`                | 允许匿名用户访问                                             |
| `authenticated()`            | 允许认证过的用户访问                                         |
| `denyAll()`                  | 无条件拒绝所有访问                                           |
| `fullyAuthenticated()`       | 如果用户是完整认证的话（不是通过Remember-me功能认证的），就允许访问 |
| `hasAnyAuthority(String...)` | 如果用户具备给定权限中的某一个的话，就允许访问               |
| `hasAnyRole(String...)`      | 如果用户具备给定角色中的某一个的话，就允许访问               |
| `hasAuthority(String)`       | 如果用户具备给定权限的话，就允许访问                         |
| `hasIpAddress(String)`       | 如果请求来自给定IP地址的话，就允许访问                       |
| `hasRole(String)`            | 如果用户具备给定角色的话，就允许访问                         |
| `not()`                      | 对其他访问方法的结果求反                                     |
| `permitAll()`                | 无条件允许访问                                               |
| `rememberMe()`               | 如果用户是通过Remember-me功能认证的，就允许访问              |

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()
      .antMatchers("/spitters/me").hasAuthority("ROLE_SPITTER")
      .antMatchers(HttpMethod.POST, "/spittles")
                                  .hasAuthority("ROLE_SPITTER")
      .anyRequest().permitAll();
}
```

作为替代方案，我们还可以使用`hasRole()`方法，它会自动使用“`ROLE_`”前缀：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()
      .antMatchers("/spitter/me").hasRole("SPITTER")
      .antMatchers(HttpMethod.POST, "/spittles").hasRole("SPITTER")
      .anyRequest().permitAll();
}
```

我们可以将任意数量的`antMatchers()`、`regexMatchers()`和`anyRequest()`连接起来，以满足Web应用安全规则的需要。但是，我们需要知道，这些规则会按照给定的顺序发挥作用。所以，很重要的一点就是将最为具体的请求路径放在前面，而最不具体的路径（如`anyRequest()`）放在最后面。如果不这样做的话，那不具体的路径配置将会覆盖掉更为具体的路径配置。

### 强制使用Https安全通道

在configure中添加requiresChannel()方法后的所有认证或鉴权url都会强制使用https进行传输

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()
      .antMatchers("/spitter/me").hasRole("SPITTER")
      .antMatchers(HttpMethod.POST, "/spittles").hasRole("SPITTER")
      .anyRequest().permitAll();
    .adn()
     .requiresChannel()
        .antMatchers("/spitter/form").requireSecure();
}
```

不论何时，只要是对“/spitter/form”的请求，Spring Security都视为需要安全通道（通过调用`requiresChannel()`确定的）并自动将请求重定向到HTTPS上。

### 防止跨站请求伪造CSRF

从Spring Security 3.2开始，默认就会启用CSRF防护。实际上，除非你采取行为处理CSRF防护或者将这个功能禁用，否则的话，在应用中提交表单时，你可能会遇到问题。

Spring Security通过一个同步token的方式来实现CSRF防护的功能。它将会拦截状态变化的请求（例如，非`GET`、`HEAD`、`OPTIONS`和`TRACE`的请求）并检查CSRF token。如果请求中不包含CSRF token的话，或者token不能与服务器端的token相匹配，请求将会失败，并抛出`CsrfException`异常。

**这意味着在你的应用中，所有的表单必须在一个“`_csrf`”域中提交token，而且这个token必须要与服务器端计算并存储的token一致，这样的话当表单提交的时候，才能进行匹配。**

好消息是，Spring Security已经简化了将token放到请求的属性中这一任务。如果你使用Thymeleaf作为页面模板的话，只要`<form>`标签的`action`属性添加了Thymeleaf命名空间前缀，那么就会自动生成一个“`_csrf`”隐藏域：

```html
<form method="POST" th:action="@{/spittles}">
   ...
</form>
```

如果使用JSP作为页面模板的话，我们要做的事情非常类似：

```jsp
<input type="hidden"
       name="${_csrf.parameterName}"
       value="${_csrf.token}" />
```

处理CSRF的另外一种方式就是根本不去处理它。我们可以在配置中通过调用`csrf().disable()`禁用Spring Security的CSRF防护功能，如下所示：

```java
@Override
protected void configure(HttpSecurity http) th
    rows Exception {
  http
    ...
      .csrf()
      .disable();//禁用CSRF防护功能
}
```

### 启用Remember-me功能

Spring Security使得为应用添加Remember-me功能变得非常容易，只需在`configure()`方法所传入的`HttpSecurity`对象上调用`rememberMe()`即可。

```java
@Override
  protected void configure(HttpSecurity http) throws Exception {
    http
      .formLogin()
        .loginPage("/login")
      .and()
      .rememberMe()
        .tokenValiditySeconds(2419200)
        .key("spittrKey")
  ...
}
```

默认情况下，这个功能是通过在cookie中存储一个token完成的，这个token最多两周内有效。但是，在这里，我们指定这个token最多四周内有效（2,419,200秒）。

存储在cookie中的token包含用户名、密码、过期时间和一个私钥——在写入cookie前都进行了MD5哈希。默认情况下，私钥的名为`SpringSecured`，但在这里我们将其设置为`spitterKey`，既然Remember-me功能已经启用，我们需要有一种方式来让用户表明他们希望应用程序能够记住他们。为了实现这一点，登录请求必须包含一个名为`remember-me`的参数。在登录表单中，增加一个简单复选框就可以完成这件事情：

```java
<input id="remember_me" name="remember-me" type="checkbox"/>
<label for="remember_me" class="inline">Remember me</label>
```

检测提交的表单中remember_me是否填写，如果填写则返回时携带cookie

### 退出

退出功能是通过Servlet容器中的Filter实现的（默认情况下），这个Filter会拦截针对“/logout”的请求。因此，为应用添加退出功能只需添加如下的链接即可（如下以Thymeleaf代码片段的形式进行了展现）：

```html
<a th:href="@{/logout}">Logout</a>
```

当用户点击这个链接的时候，会发起对“/logout”的请求，这个请求会被Spring Security的`LogoutFilter`所处理。用户会退出应用，所有的Remember-me token都会被清除掉。在退出完成后，用户浏览器将会重定向到“/login?logout”，从而允许用户进行再次登录。

如果你希望用户被重定向到其他的页面，如应用的首页，那么可以在`configure()`中进行如下的配置：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .formLogin()
      .loginPage("/login")
    .and()
    .logout()
      .logoutSuccessUrl("/")
  ...
}
```

在这里，和前面一样，通过`add()`连接起了对`logout()`的调用。`logout()`提供了配置退出行为的方法。在本例中，调用`logoutSuccessUrl()`表明在退出成功之后，浏览器需要重定向到“/”。

除了`logoutSuccessUrl()`方法以外，你可能还希望重写默认的`LogoutFilter`拦截路径。我们可以通过调用`logoutUrl()`方法实现这一功能：

```java
.logout()
  .logoutSuccessUrl("/")
  .logoutUrl("/signout")
```

## 保护视图

### JSP保护视图

Spring Security本身提供了一个JSP标签库，而Thymeleaf通过特定的方言实现了与Spring Security的集成。通过标签库控制视图渲染的安全性。

| JSP标签                      | 作　　用                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| `security:accesscontrollist` | 如果用户通过访问控制列表授予了指定的权限，那么渲染该标签体中的内容 |
| `security:authentication`    | 渲染当前用户认证对象的详细信息                               |
| `security:authorize`         | 如果用户被授予了特定的权限或者SpEL表达式的计算结果为true，那么渲染该标签体中的内容 |

为了使用标签库，一般需要进行声明（JSP）

```jsp
<%@ taglib prefix="security"
           uri="http://www.springframework.org/security/tags" %>
```

**访问认证信息的细节**

借助Spring Security JSP标签库，所能做到的最简单的一件事情就是便利地访问用户的认证信息。例如，对于Web站点来讲，在页面顶部以用户名标示显示“欢迎”或“您好”信息是很常见的。这恰恰是`security:authentication`能为我们所做的事情。例如：

```jsp
Hello <security:authentication property="principal.username" />!
```

其中，property用来标示用户认证对象的一个属性。可用的属性取决于用户认证的方式。但是，我们可以依赖几个通用的属性，在不同的认证方式下，它们都是可用的

| 认 证 属 性   | 描　　述                                         |
| ------------- | ------------------------------------------------ |
| `authorities` | 一组用于表示用户所授予权限的GrantedAuthority对象 |
| `Credentials` | 用于核实用户的凭证（通常，这会是用户的密码）     |
| `details`     | 认证的附加信息（IP地址、证件序列号、会话ID等）   |
| `principal`   | 用户的基本信息对象                               |

`<security:authentication>`将在视图中渲染属性的值。但是如果你愿意将其赋值给一个变量，那只需要在`var`属性中指明变量的名字即可。例如，如下展现了如何将其设置给名为`loginId`的属性：

```jsp
<security:authentication property="principal.username"
        var="loginId"/>
```

**条件性的渲染内容**

有时候视图上的一部分内容需要根据用户被授予了什么权限来确定是否渲染。对于已经登录的用户显示登录表单，或者对还未登录的用户显示个性化的问候信息都是毫无意义的。

Spring Security的`<security:authorize>`JSP标签能够根据用户被授予的权限有条件地渲染页面的部分内容。例如，在Spittr应用中，对于没有`ROLE_SPITTER`角色的用户，我们不会为其显示添加新Spitter记录的表单，只需要在要渲染的地方上main加上`<security:authorize>`标签，根据不同的属性来进行视图保护

`access`属性被赋值为一个SpEL表达式，这个表达式的值将确定`<security:authorize>`标签主体内的内容是否渲染。`hasRole('ROLE_SPITTER')`表达式来确保用户具有`ROLE_SPITTER`角色

<img src="https://i.loli.net/2020/03/07/8GYPvUOdy3mlNxk.png" alt="image.png" style="zoom: 67%;" />

也可以直接赋予access一个SpEl表达式

```jsp
<security:authorize
   access="isAuthenticated() and principal.username=='habuma'">
  <a href="/admin">Administration</a>
</security:authorize>
```

### 使用Thymeleaf保护视图

与Spring Security的JSP标签库类似，Thymeleaf的安全方言提供了条件化渲染和显示认证细节的能力。

| 属　　性             | 作　　用                                                     |
| -------------------- | ------------------------------------------------------------ |
| `sec:authentication` | 渲染认证对象的属性。类似于Spring Security的``JSP标签         |
| `sec:authorize`      | 基于表达式的计算结果，条件性的渲染内容。类似于Spring Security的``JSP标签 |
| `sec:authorize-acl`  | 基于表达式的计算结果，条件性的渲染内容。类似于Spring Security的`` JSP标签 |
| `sec:authorize-expr` | sec:authorize属性的别名                                      |
| `sec:authorize-url`  | 基于给定URL路径相关的安全规则，条件性的渲染内容。类似于Spring Security的`` JSP标签使用url属性时的场景 |

使用上述标签时，要向IOC容器注入一个Bean配置模板引擎

![image.png](https://i.loli.net/2020/03/07/lMzAkpsWnGUIC5i.png)

安全方言注册完成之后，我们就可以在Thymeleaf模板中使用它的属性了。首先，需要在使用这些属性的模板中声明安全命名空间：

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:sec=
          "http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
  ...
</html>
```

在这里，标准的Thymeleaf方法依旧与之前一样，使用`th`前缀，安全方言则设置为使用`sec`前缀。

这样我们就能在任意合适的地方使用Thymeleaf属性了。比如，假设我们想要为认证用户渲染“Hello”文本。

```html
<div sec:authorize="isAuthenticated()">
  Hello <span sec:authentication="name">someone</span>
</div>
```

`sec:authorize`属性会接受一个SpEL表达式。如果表达式的计算结果为`true`，那么元素的主体内容就会渲染。在本例中，表达式为`isAuthenticated()`，所以只有用户已经进行了认证，才会渲染`<div>`标签的主体内容。就这个标签的主体内容部分而言，它的功能是使用认证对象的`name`属性提示“Hello”文本。

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

