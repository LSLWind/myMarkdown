## 概述

Apache Shiro 是一个强大灵活的开源安全框架，可以完全处理身份验证、授权、加密和会话管理。 

Shiro的主要功能包括：

- **Authentication（认证）：**用户身份识别，通常被称为用户“登录”
- **Authorization（授权）：**访问控制。比如某个用户是否具有某个操作的使用权限。
- **Session Management（会话管理）：**特定于用户的会话管理,甚至在非web 或 EJB 应用程序。
- **Cryptography（加密）：**在对数据源使用加密算法加密的同时，保证易于使用。

同时提供以下功能：

- **Web支持：**Shiro的Web支持API有助于保护Web应用程序。
- **缓存：**缓存是Apache Shiro API中的第一级，以确保安全操作保持快速和高效。
- **并发性：**Apache Shiro支持具有并发功能的多线程应用程序。
- **测试：**存在测试支持，可帮助您编写单元测试和集成测试，并确保代码按预期得到保障。
- **“运行方式”：**允许用户承担另一个用户的身份(如果允许)的功能，有时在管理方案中很有用。
- **“记住我”：**记住用户在会话中的身份，所以用户只需要强制登录即可。

 Shiro不会去维护用户、维护权限，这些需要我们自己去设计/提供，然后通过相应的接口注入给Shiro 

 Shiro 架构包含三个主要的理念：Subject,SecurityManager和 Realm。Subject→SecurityManager→Realm

- **Subject：**当前用户，Subject 可以是一个人，但也可以是第三方服务、守护进程帐户、时钟守护任务或者其它–当前和软件交互的任何事件。
- **SecurityManager：**管理所有Subject，SecurityManager 是 Shiro 架构的核心，配合内部安全组件共同组成安全伞。
- **Realms：**用于进行权限信息的验证，我们自己实现。Realm 本质上是一个特定的安全 DAO：它封装与数据源连接的细节，得到Shiro 所需的相关的数据。在配置 Shiro 的时候，你必须指定至少一个Realm 来实现认证（authentication）和/或授权（authorization）。

## Shiro认证-Authentication

**导入依赖**

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.0</version>
</dependency>
```

shiro的认证过程大致为：创建SecurityManager→Subject提交认证→SecurityManager进行认证→Authenticator认证→Realm验证

导入junit做测试

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

```java
public class AuthenticationTest {
    SimpleAccountRealm simpleAccountRealm=new SimpleAccountRealm();

    //0.构建用户
    @Before
    public void addUser(){
        simpleAccountRealm.addAccount("lsl","123");//增加用户
    }

    /**
     * 测试认证
     */
    @Test
    public void testAuthentication(){
        //1.构建SecurityManager环境
        DefaultSecurityManager defaultSecurityManager=new DefaultSecurityManager();
        defaultSecurityManager.setRealm(simpleAccountRealm);//设置要保证安全的Realm
        //2.主体Subject提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager);//设置安全管理环境
        Subject subject=SecurityUtils.getSubject();//获取一个Subject

        UsernamePasswordToken token=new UsernamePasswordToken("lsl","123");//获取用户token
        subject.login(token);//要求用户登录


        // subject.isAuthenticated()方法返回一个boolean值,用于判断用户是否认证成功
        System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出true
        subject.logout(); // 登出
        System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出false
    }
}
```

具体的，认证流程为：

1. 首先调用 Subject.login(token) 进行登录，其会自动委托给 Security Manager，调用之前必须通过 SecurityUtils.setSecurityManager() 设置；
2. SecurityManager 负责真正的身份验证逻辑；它会委托给 Authenticator 进行身份验证；
3. Authenticator 才是真正的身份验证者，Shiro API 中核心的身份认证入口点，此处可以自定义插入自己的实现；
4. Authenticator 可能会委托给相应的 AuthenticationStrategy 进行多 Realm 身份验证，默认 ModularRealmAuthenticator 会调用 AuthenticationStrategy 进行多 Realm 身份验证；
5. Authenticator 会把相应的 token 传入 Realm，从 Realm 获取身份验证信息，如果没有返回 / 抛出异常表示身份验证失败了。此处可以配置多个 Realm，将按照相应的顺序及策略进行访问。

## Shiro授权-Authorization

与认证过程类似

```java
public class AuthenticationTest {

    SimpleAccountRealm simpleAccountRealm = new SimpleAccountRealm();

    @Before // 在方法开始前添加一个用户,让它具备admin和user两个角色
    public void addUser() {
        simpleAccountRealm.addAccount("lsl", "123456", "admin", "user");
    }

    @Test
    public void testAuthentication() {
        // 1.构建SecurityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(simpleAccountRealm);

        // 2.主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager); // 设置SecurityManager环境
        Subject subject = SecurityUtils.getSubject(); // 获取当前主体

        UsernamePasswordToken token = new UsernamePasswordToken("lsl", "123456");
        subject.login(token); // 登录

        // subject.isAuthenticated()方法返回一个boolean值,用于判断用户是否认证成功
        System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出true
        // 判断subject是否具有admin和user两个角色权限,如没有则会报错
        subject.checkRoles("admin","user");
    }
}
```

## 自定义Realm

 实际进行权限信息验证的是我们的 Realm，Shiro 框架内部默认提供了两种实现，一种是查询`.ini`文件的`IniRealm`，另一种是查询数据库的`JdbcRealm`。

自定义Realm

```java
public class MyRealmTest extends AuthorizingRealm {
    Map<String,String> userMap=new HashMap<>();
    {
        userMap.put("lsl","123");
        super.setName("MyRealm");//自定义Realm名称
    }

    /**
     * 模拟从数据库汇总获取权限数据
     */
    private Set<String> getPermissionsByUserName(String userName) {
        Set<String> permissions = new HashSet<>();
        permissions.add("user:delete");
        permissions.add("user:add");
        return permissions;
    }
    /**
     * 模拟从数据库中获取角色数据
     */
    private Set<String> getRolesByUserName(String userName) {
        Set<String> roles = new HashSet<>();
        roles.add("admin");
        roles.add("user");
        return roles;
    }
    /**
     * 模拟从数据库取凭证的过程
     */
    private String getPasswordByUserName(String userName) {
        return userMap.get(userName);
    }

    /**
     * 实际进行授权的方法
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String userName = (String) principalCollection.getPrimaryPrincipal();
        // 从数据库获取角色和权限数据
        Set<String> roles = getRolesByUserName(userName);
        Set<String> permissions = getPermissionsByUserName(userName);

        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        simpleAuthorizationInfo.setStringPermissions(permissions);
        simpleAuthorizationInfo.setRoles(roles);
        return simpleAuthorizationInfo;
    }
    /**
     * 实际进行认证的方法
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        // 1.从主体传过来的认证信息中，获得用户名
        String userName = (String) authenticationToken.getPrincipal();

        // 2.通过用户名到数据库中获取凭证
        String password = getPasswordByUserName(userName);
        if (password == null) {
            return null;
        }
        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo("lsl", password, "MyRealm");
        return authenticationInfo;
    }
}
```

从代码中可以看出，首先用户要提供信息（认证信息与权限），也就是代码初始时模拟的。用户再一次访问时，认证时从Subject中获取信息（token），交给认证方法 doGetAuthenticationInfo(AuthenticationToken authenticationToken)去进行信息的认证；授权时将用户想要的权限信息交给方法AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection)去进行权限的认证。

## shiro加密

我们在数据库中保存的密码都是明文的，一旦数据库数据泄露，那就会造成不可估算的损失，所以我们通常都会使用非对称加密，简单理解也就是**不可逆**的加密，而 md5 加密算法就是符合这样的一种算法。

如上面的 123456 用 Md5 加密后，得到的字符串：**e10adc3949ba59abbe56e057f20f883e**，就无法通过计算还原回 123456，我们把这个加密的字符串保存在数据库中，等下次用户登录时我们把密码通过同样的算法加密后再从数据库中取出这个字符串进行比较，就能够知道密码是否正确了，这样既保留了密码验证的功能又大大增加了安全性，**但是问题是：虽然无法直接通过计算反推回密码，但是我们仍然可以通过计算一些简单的密码加密后的 Md5 值进行比较，推算出原来的密码**

比如我的密码是 123456，你的密码也是，通过 md5 加密之后的字符串一致，所以你也就能知道我的密码了，如果我们把常用的一些密码都做 md5 加密得到一本字典，那么就可以得到相当一部分的人密码，这也就相当于“破解”了一样，所以其实也没有我们想象中的那么“安全”。

可以使用**加盐 + 多次加密**

既然相同的密码 md5 一样，那么我们就让我们的原始密码再**加一个随机数**，然后再进行 md5 加密，这个随机数就是我们说的**盐(salt)**，这样处理下来就能得到不同的 Md5 值，当然我们需要把这个随机数盐也保存进数据库中，以便我们进行验证。

另外我们可以通过**多次加密**的方法，即使黑客通过一定的技术手段拿到了我们的密码 md5 值，但它并不知道我们到底加密了多少次，所以这也使得破解工作变得艰难。

在 Shiro 框架中，对于这样的操作提供了简单的代码实现：

```java
String password = "123456";
String salt = new SecureRandomNumberGenerator().nextBytes().toString();
int times = 2;  // 加密次数：2
String alogrithmName = "md5";   // 加密算法

String encodePassword = new SimpleHash(alogrithmName, password, salt, times).toString();

System.out.printf("原始密码是 %s , 盐是： %s, 运算次数是： %d, 运算出来的密文是：%s ",password,salt,times,encodePassword);
```

输出：

```text
原始密码是 123456 , 盐是： f5GQZsuWjnL9z585JjLrbQ==, 运算次数是： 2, 运算出来的密文是：55fee80f73537cefd6b3c9a920993c25 
```

## SpringBoot整合Realm

**0.导入依赖**

```xml
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-web</artifactId>
        <version>1.4.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.4.0</version>
    </dependency>
```

**1.自定义核心组件Realm**

**2.配置Shiro**

写一个Shiro的配置类即可

```java
@Configuration
public class ShiroConfig {
    
    @Bean
    public ShiroFilterFactoryBean shirFilter(SecurityManager securityManager) {
        System.out.println("ShiroConfiguration.shirFilter()");
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager( securityManager);
        //拦截器.
        Map<String,String> filterChainDefinitionMap = new LinkedHashMap<String,String>();
        // 配置不会被拦截的链接 顺序判断
        filterChainDefinitionMap.put("/static/**", "anon");
        //配置退出 过滤器,其中的具体的退出代码Shiro已经替我们实现了
        filterChainDefinitionMap.put("/logout", "logout");
        //<!-- 过滤链定义，从上向下顺序执行，一般将/**放在最为下边 -->:这是一个坑呢，一不小心代码就不好使了;
        //<!-- authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问-->
        filterChainDefinitionMap.put("/**", "authc");
        // 如果不设置默认会自动寻找Web工程根目录下的"/login.jsp"页面
        shiroFilterFactoryBean.setLoginUrl("/login");
        // 登录成功后要跳转的链接
        shiroFilterFactoryBean.setSuccessUrl("/index");

        //未授权界面;
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }

    @Bean
    public MyRealm myRealm(){
        MyRealm myRealm = new MyRealm();
        return myRealm;
    }
    
    @Bean
    public SecurityManager securityManager(){
        DefaultWebSecurityManager securityManager =  new DefaultWebSecurityManager();
        securityManager.setRealm(myRealm());
        return securityManager;
    }
}
```

 Apache Shiro 核心通过 Filter 来实现，就好像 SpringMvc 通过 DispachServlet 来主控制一样。
既然是使用 Filter 一般也就能猜到，是通过 URL 规则来进行过滤和权限校验，所以我们需要定义一系列关于 URL 的规则和访问权限。 

Shiro 内置的 FilterChain

| Filter Name | Class                                                        |
| ----------- | ------------------------------------------------------------ |
| anon        | org.apache.shiro.web.filter.authc.AnonymousFilter            |
| authc       | org.apache.shiro.web.filter.authc.FormAuthenticationFilter   |
| authcBasic  | org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter |
| perms       | org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter |
| port        | org.apache.shiro.web.filter.authz.PortFilter                 |
| rest        | org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter |
| roles       | org.apache.shiro.web.filter.authz.RolesAuthorizationFilter   |
| ssl         | org.apache.shiro.web.filter.authz.SslFilter                  |
| user        | org.apache.shiro.web.filter.authc.UserFilter                 |

- anon:所有 url 都都可以匿名访问
- authc: 需要认证才能进行访问
- user:配置记住我或认证通过可以访问

**认证/授权等**

同上述，主要的验证类是Realm，通过配置类规定URL验证规则，之后验证时直接使用规则即可