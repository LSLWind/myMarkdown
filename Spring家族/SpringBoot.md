### 基础

#### 新建应用

 可以通过 https://start.spring.io/ 这个网站来生成一个 Spring Boot 的项目。 

 当然也可以直接通过 IDEA 来生成一个 Spring Boot 的项目，具体方法和上面类似：`File->New->Project->Spring Initializr`。 之后再选择依赖Dependencies时可以勾选各种各样的依赖帮助生成，注意勾选Web，预置了非常多的依赖，开箱即用，非常方便

1. `Application.java`是项目的启动类
2. domain目录主要用于实体（Entity）与数据访问层（Repository）
3. service 层主要是业务类代码
4. controller 负责页面访问控制
5. config 目录主要放一些配置类
6. resources  放置静态资源，如(html,mapper的xml等)，包含子目录static与templates

启动类使用注解 @SpringBootApplication 声明，启动自动扫描，自动配置，自动注入。如果新建Web应用，会自动导入Tomcat服务器并进行配置

 **@SpringBootApplication = （默认属性的） @Configuration + @EnableAutoCon:figuration + @ComponentScan** 

```java
 public static void main(String[] args) {
        SpringApplication.run(FoundationApplication.class, args);
    }
```

**SpringApplication将引导用户的应用启动并相应地启动被自动配置的Tomcat Web 服务器。**

对于SpringBoot来说，确实简化了大量配置，服务器都不用自己配置，默认启用tomcat,访问静态资源的时候直接返回resources下的static下的静态资源，访问请求时由controller进行映射

#### 全局配置文件application.properties

**注意导入Spring Configuration Processor----依赖spring-boot-configuration-processor， spring默认使用yml中的配置，但有时候要用传统的xml或properties配置，就需要使用spring-boot-configuration-processor **

定义全局属性，通过注解获取信息

**自定义属性支持**

```
name="lsl"
```

然后直接在要使用的地方通过注解@Value(value=”${config.name}”)就可以绑定到你想要的属性上面

```
@Value("${lsl}") 
private  String name;
```



#### 使用Bean进行全局配置

对于某些全局配置属性，除了可以在application.properties中进行设置，也可以在Bean中进行配置，创建一个Bean，使用注解@ConfigurationProperties(prefix = "包名前缀")，目的是减少冗余，方便写。之后要在主应用中声明配置@EnableConfigurationProperties({ConfigurationBean.class})指定加载哪一个配置bean，如果不指定，在配置bean中加上@Configuration也可以：

```java
@Getter
@Setter
@Data
@ConfigurationProperties(prefix = "configure")
public class ConfigurationBean {
    private String testConfiguration="配置bean，自己设置值";
    private int testId;

}
```

```java
@SpringBootApplication
@EnableConfigurationProperties({ConfigurationBean.class})
public class FoundationApplication {
    public static void main(String[] args) {
        SpringApplication.run(FoundationApplication.class, args);
    }
}
```

原理是通过自动扫描配置bean(通过@Autowired)加载，这样的话还是通过对象属性引用值

#### 使用自定义配置文件

在src/main/resources下新建配置文件



#### @RequestMapping 中的produces

prodces用于指定返回响应的值的类型，如html,text等

```java
@Controller  
@RequestMapping(value = "/pets/{petId}", method = RequestMethod.GET, produces="application/json")  
@ResponseBody  
public Pet getPet(@PathVariable String petId, Model model) {     
    // implementation omitted  
}  
```

同样也可以用于指定编码：

```java
@Controller  
@RequestMapping(value = "/pets/{petId}", produces="MediaType.APPLICATION_JSON_VALUE"+";charset=utf-8")  
@ResponseBody  
public Pet getPet(@PathVariable String petId, Model model) {      
    // implementation omitted  
}  
```

 https://blog.csdn.net/jaryle/article/details/72965885 

#### 统一访问接口WebRequest

 统一请求访问接口，不仅仅可以访问请求相关数据（如参数区数据、请求头数据，但访问不到Cookie区数据），还可以访问会话和上下文中的数据；NativeWebRequest继承了WebRequest，并提供访问本地Servlet API的方法。 

#### 通用响应实体ResponseEntity

 ResponseEntity标识整个http相应：状态码、头部信息以及响应体内容。因此我们可以使用其对http响应实现完整配置。 一个泛型。主要用于在返回数据的时候附带响应状态码，如果不使用则需要自定义一个类，封装返回数据，响应码，其它信息等，因为Rest经常有这个需求，所以提供一个统一的类，用于附带响应状态码

```java
@GetMapping("/hello")
ResponseEntity<String> hello() {
    return new ResponseEntity<>("Hello World!", HttpStatus.OK);
}
```

其使用方式非常多样，可以指定为响应的各种信息，参考：

 https://blog.csdn.net/neweastsun/article/details/81142870 

### CORS跨域请求

 跨域资源共享([CORS](https://developer.mozilla.org/en-US/docs/Glossary/CORS)) 是一种机制，它使用额外的 [HTTP](https://developer.mozilla.org/en-US/docs/Glossary/HTTP) 头来告诉浏览器 让运行在一个 origin (domain) 上的Web应用被准许访问来自不同源服务器上的指定的资源。 通俗的说，浏览器允许当前页面上的资源是别的服务器上的资源（这应该是是一种常见的请求，资源共享）， 一个资源从与该资源本身所在的服务器**不同的域、协议或端口**请求一个资源时，资源会发起一个**跨域 HTTP 请求**。 现在的问题是， JavaScript由于安全性方面的考虑，不允许页面跨域调用其他页面的对象 ，java中的几种解决办法

**设置响应头：**

```java
response.setHeader("Access-Control-Allow-Origin", "*")
```

 Access-Control-Allow-Origin这个Header在W3C标准里用来**检查该跨域请求是否可以被通过**，如果值**为\*则表明当前页面可以跨域访问**。默认的情况下是不允许的 

**web.xml添加过滤器**

```xml
<filter>
    <filter-name>CORS</filter-name>
    <filter-class>com.thetransactioncompany.cors.CORSFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>CORS</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

导入依赖：

```xml
<dependency>
    <groupId>com.thetransactioncompany</groupId>
    <artifactId>cors-filter</artifactId>
    <version>2.5</version>
</dependency>
```

### 构建Rest API

#### RESTful Web

 RESTful Web 服务与传统的 MVC 开发一个关键区别是返回给客户端的内容的创建方式：**传统的 MVC 模式开发会直接返回给客户端一个视图，但是 RESTful Web 服务一般会将返回的数据以 JSON 的形式返回，这也就是现在所推崇的前后端分离开发。** 

1. `@RestController` **将返回的对象数据直接以 JSON 或 XML 形式写入 HTTP 响应(Response)中。**绝大部分情况下都是直接以 JSON 形式返回给客户端，很少的情况下才会以 XML 形式返回。
2. `@RequestMapping` :上面的示例中没有指定 GET 与 PUT、POST 等，因为**`@RequestMapping`默认映射所有HTTP Action**，你可以使用`@RequestMapping(method=ActionType)`来缩小这个映射。
3. `@PostMapping`实际上就等价于 `@RequestMapping(method = RequestMethod.POST)`，同样的 `@DeleteMapping` ,`@GetMapping`也都一样，常用的 HTTP Action 都有一个这种形式的注解所对应。
4. `@PathVariable` :取url地址中的参数。
5. `@RequestParam` url的查询参数值。
6. `@RequestBody`:**将 HttpRequest body 中的 JSON 类型数据反序列化为合适的 Java 类型。**
7. `ResponseEntity`: **表示整个HTTP Response：状态码，标头和正文内容**。我们可以使用它来自定义HTTP Response 的内容。

#### 前端请求

后端给前端请求的数据，格式为json，前端接收数据后再进行渲染填充，一般使用ajax

```java
    function requestTest1() {
        var info = $("#info").val();//json数据
        $.ajax({
            type: "GET",//方法类型
            dataType: "text",//预期服务器返回的数据类型
            url: "api/test1?info=" + info,//请求接口
            contentType: "application/json; charset=utf-8",//设置类型
            success: function (result) {
                $("#test1").html(JSON.stringify(result));
            },
            error: function () {
                $("#test1").html("接口异常，请联系管理员！");
            }
        });
    }
```

对于请求的API，可以根据用户id进行访问，后台可以根据@PathVariable进行实时映射：

```js
url: "/api/users/" + id
```

id可以由用户输入，也可以从cookie中获取，规定的GET,POST,PUT,DELETE分别对应于select/insert/udpate/delete，可以采用同一条路径但是不同的请求方法采取不同的动作处理

#### controller处理

普通Ajax映射处理：

```java
@RestController
@RequestMapping("/api")
public class AjaxRequestTest {
    @RequestMapping(value = "/test1",method = RequestMethod.GET)
    public String test1(String info){
        if(StringUtils.isEmpty(info)){
            return "info为空";
        }

        return "信息为："+info;
    }

    @RequestMapping(value = "/test2",method = RequestMethod.GET)
    public List<User> test2(){
        List<User> users=new ArrayList<>();
        User a=new User();
        a.setId(1);
        User b=new User();
        b.setId(2);
        users.add(a);
        users.add(b);
        return users;
    }

}
```

返回响应,body中存储相关信息数据(导入json依赖，自动转换为json格式)，由前端渲染处理。

构建Rest API时，前后端要约定好数据的传输格式，不能一股脑全部传数据，必须附带一些有用的信息给前端，如返回数据的状态码，返回数据的附加信息等，常见的，先构建状态码：

```java
public class Constants {
    public static final int RESULT_CODE_SUCCESS = 200;  // 成功处理请求
    public static final int RESULT_CODE_BAD_REQUEST = 412;  // 请求错误
    public static final int RESULT_CODE_NOT_LOGIN = 402;  // 未登录
    public static final int RESULT_CODE_PARAM_ERROR = 406;  // 传参错误
    public static final int RESULT_CODE_SERVER_ERROR = 500;  // 服务器错误
}
```

构建结果集，即返回数据的json的具体格式：

```java
@Data
@AllArgsConstructor
public class Result<T> implements Serializable {
    private static final long serialVersionUID=1L;
    private int resultCode;//响应状态码
    private String message;//附加消息
    private T data;//实际数据

    public  Result(){

    }
    public Result(int resultCode,String message){
        this.resultCode=resultCode;
        this.message=message;
    }
}
```

根据映射路径，即api请求与请求方法采取不同的动作并返回对应的结果：

```java
    @Resource
    UserDao userDao;

    @RequestMapping(value = "/users/{id}",method = RequestMethod.GET)
    public Result<User> getUserById(@PathVariable("id") Integer id){
        if(id==null||id<1){
            return new Result<>(Constants.RESULT_CODE_BAD_REQUEST,"用户id错误");
        }
        User user=userDao.getUserById(id);
        if(user==null){
            return new Result<>(Constants.RESULT_CODE_BAD_REQUEST,"该用户不存在");
        }
        return new Result<>(Constants.RESULT_CODE_SUCCESS,"OK",user);
    }
```

使用postman进行接口测试，发送 http://localhost:8080/api/users/0 ，返回json：

```json
{
  "resultCode": 412,
  "message": "用户id错误",
  "data": null
}
```

发送 http://localhost:8080/api/users/1 ,返回json

```json
{
    "resultCode": 200,
    "message": "OK",
    "data": {
        "id": 1,
        "email": "1",
        "name": "3",
        "password": "2",
        "sex": "男"
    }
}
```

即Restful API依赖后台识别url中的路径参数(@PathVariable)，根据路径参数与请求方法采取对应的动作，取出数据将数据返回结果集，以json格式返回，后台也可以读取前台发过来的json数据（@RequestBody）并将json序列化成对应的对象（注入属性），根据@PathVariable也可以填充模板，约定大于配置。

### 错误页面

错误页面即服务器找不到资源（404），请求错误（5xx）等出现错误时（对应头部响应码）所跳转到的页面，完善用户体验。错误跳转需要实现接口ErrorController，仍由控制器进行控制

#### 接口ErrorController

该接口用于发生错误时进行二次跳转，必须实现方法public String getErrorPath()，当发生错误时会根据错误路径进行二次跳转

#### HttpStauts

spring提供的表示响应状态的抽象封装，根据request的getAttribute()方法获取请求状态，然后可使用HttpStatus提供的静态方法进行抽象封装，最后根据HppsStatus进行判别，更加清晰

```java
//错误页面跳转控制
@Controller
public class ErrorPageController implements ErrorController {
    private final static String ERROR_PATH="/error";//错误跳转路径
    //必须实现返回错误路径的跳转，然后进行二次映射
    @Override
    public String getErrorPath(){
        return ERROR_PATH;
    }

    //跳转到错误路径时根据错误状态码返回不同的错误页面,produces指定返回类型
    @RequestMapping(value = ERROR_PATH,produces = "text/html")
    public ModelAndView errorHtml(HttpServletRequest request){
        //获取请求状态码，判断是否发生error
        Integer statusCode = (Integer) request
                .getAttribute("javax.servlet.error.status_code");
        HttpStatus httpStatus;//HttpStatus请求状态
        if(statusCode!=null){
            httpStatus=HttpStatus.valueOf(statusCode);//通过HttpStatus返回错误状态信息
        }else httpStatus=HttpStatus.INTERNAL_SERVER_ERROR;

        //根据具体的错误类型，返回具体的错误页面
        if (HttpStatus.BAD_REQUEST == httpStatus) {
            return new ModelAndView("error/error_400");
        } else if (HttpStatus.NOT_FOUND == httpStatus) {
            return new ModelAndView("error/error_404");
        } else {
            return new ModelAndView("error/error_5xx");
        }
    }
    
}

```

### MyBatis

传统的MyBatis使用要配置总文件，配置映射xml文件与映射接口，MyBatis读取元数据，加载开启会话，执行映射文件中的sql语句并返回结果。

MyBatis为SpringBoot提供了更方便的整合：mybatis-spring-boot-starter

官方说明：`MyBatis Spring-Boot-Starter will help you use MyBatis with Spring Boot`

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>
```

另有两个常规依赖：

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
     <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
```

#### 在application.properties中设置数据库数据源并测试

要在application.properties全局配置文件中配置相关属性：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/test?  serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.username=root   
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

 Spring Boot 会自动加载 `spring.datasource.*` 相关配置，数据源就会自动注入到 sqlSessionFactory 中，sqlSessionFactory 会自动注入到 Mapper 中。 之后就是Mapper的问题了，先测试一下数据源有无问题：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class FoundationApplicationTest {
        // 注入数据源对象
        @Autowired
        private DataSource dataSource;
        @Test
        public void datasourceTest() throws SQLException {
            // 获取数据源类型
            System.out.println("默认数据源为：" + dataSource.getClass());
            // 获取数据库连接对象
            Connection connection = dataSource.getConnection();
            // 判断连接对象是否为空
            System.out.println(connection != null);
            connection.close();
        }
}
```

#### 通过配置xml使用MyBatis

导入依赖，application.properties指定数据源，指定mapper，如下：

```xml
mybatis.mapper-locations=classpath:mapper/*.xml
```

在resources下编写对应的sql，即mapper的映射文件，这里为resources/mapper，注意指定mapper：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.lsl.foundation.mapper.UserDao">
    <resultMap type="com.lsl.foundation.entity.User" id="UserResult">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="password" column="password"/>
        <result property="email" column="email"/>
        <result property="sex" column="sex"/>
    </resultMap>

    <select id="findAllUsers" resultMap="UserResult">
        select id,email,name,password,sex from user
        order by id desc
    </select>

</mapper>
```

写对应的接口文件：

```java
public interface UserDao {
    List<User> findAllUsers();
}
```

主应用加上@MapperScan("对应的xml文件的接口的包")指定需要映射的包:

```java
@SpringBootApplication
@MapperScan("com.lsl.foundation.mapper")
public class FoundationApplication {

    public static void main(String[] args) {
        SpringApplication.run(FoundationApplication.class, args);
    }

}
```

使用UserDao进行SQL操作，直接使用@Resource或者@Autowired即可：

```java
    @Autowired
    UserDao userDao;
    @Test
    public void MyBatisMapperTest() throws Exception{
        for(User user:userDao.findAllUsers()){
            System.out.println(user.toString());
        }
    }
```

整个过程可以描述为：从application.properties中读取数据，包括数据库数据源与mapper映射源(xxx.xml)，在对应的目录下写xml文件，在xml的namespace中指定对应的接口包，在接口包下写对应的接口，然后在主应用中开启自动扫描mapper，指定接口mapper的包路径，然后直接通过@Resource或者@Autowired声明接口字段即可。**数据流动为：需要SQL接口对象操作-开启数据库连接会话-扫描接口mapper-扫描对应的xml-传递参数执行sql-返回结果并本地映射**

#### 通过注解使用MyBatis

直接去掉xml，通过注解获取sql语句，直接在对应的接口写sql，数据流动从xml改为从注解流动，application.properties中要去掉mapper指定.xml，在接口处要加上@Mapper以供识别，主应用的注解包扫描仍然需要，**数据流动为：需要SQL接口对象操作-开启数据库连接会话-扫描Mapper-通过语句注解获取SQL，根据参数注入数据执行sql-返回结果并本地映射**

```java
@Mapper
public interface UserDao {
    @Select("select id,email,name,password,sex from user where id=#{id}")
    User getUserById(@Param("id") String id);
}
```

#### 出错怎么办

1.检查依赖，一定要导入正确的依赖

2.检查application.properties属性数据源配置是否正确

3.检查是否漏写注解，是否漏写mapper

4.检查sql语句是否正确，注意多参数使用@Param，是否拼写不对应

5.检查bean是否正确，是否拼写不对应

### JdbcTemplate

jdbcTemplate是Spring对JDBC的进一步封装，使用时只需要在application.properties中配置相关的属性，然后直接自动扫描导入即可，之后就是调API执行SQL

```java
    //自动配置，因此可以直接通过 @Autowired 注入进来
    @Autowired
    JdbcTemplate jdbcTemplate;
```

## 访问静态资源

SpringBoot会在resources/static目录下扫描静态资源文件，如图片。访问静态资源要导入依赖

```xml
        <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
```

目录resources/static/img/x.png就等于访问http://127.0.0.1:8080/img/x.png

Spring Boot默认将所有的静态资源映射到以下目录

```
classpath:/static 
classpath:/public 
classpath:/resources 
classpath:/META-INF/resources
```

优先级为优先级`/META-INF/resources/>/resources/>/static/>/public/`

### 静态资源不更新

国外人员为了方便开发,Springboot启动的时候默认启动了一个服务:LiveReload。这东西干嘛用的呢,就是网页连刷新都不用刷新了,资源文件一更新直接显示在页面上了,但是这个东西呢,需要去Chrome浏览器的插件商店装个插件...不FQ是没法装的...所以在国外是个非常方便的东西,在国内就成了好多人用不了的不知名东西...

然后thymeleaf模板引擎默认开启了静态文件缓存,加快了访问速度,国外有LiveReload这个东西监听资源文件,可以实时更新了以后reload项目,显示在页面上,

但是国内没有啊,更新的静态文件就被thymeleaf缓存了,除非完全重启项目才能把项目缓存释放,否则就一直在缓存里面,就造成了不更新的现象了...

解决方法就是**在application.yml(或者是你的配置文件里),把thymeleaf的缓存关闭**

```
spring.thymeleaf.cache=false
```

## 日志

SprinbBoot整合日志很容易，内部有自己的日志Logback，但整合其他日志也非常容易

### LogBack

**调整控制台输出日志为DEBUG级别**

默认情况下，SpringBoot只会在控制台输出 `INFO`及以上级别（`WARN`、`ERROR`）的日志。如果你想输出 `DEBUG`级别的日志，可以通过以下两种方法：

1. 在运行SpringBoot应用 `jar`包时指定 `--debug`参数：

```
java -jar myApp.jar --debug
```

2. 在你的 `application.properties`中添加 `debug=true`

#### 配置

使用默认日志只需要在application.properties中配置日志级别即可，console默认输入ERROR, WARN ，INFO级别的日志。可通过修改logging.level属性来改变日志的输出级别。可以通过配置logging.file属性或logging.path属性将日志输出到文件中。当文件到达10M的时候，将新建一个文件记录日志。

**logging.level.\* :** 作为package（包）的前缀来设置日志级别。
 **logging.file :**配置日志输出的文件名，也可以配置文件名的绝对路径。
 **logging.path :**配置日志的路径。如果没有配置**logging.file**,Spring Boot 将默认使用spring.log作为文件名。
 **logging.pattern.console :**定义console中logging的样式。
 **logging.pattern.file :**定义文件中日志的样式。
 **logging.pattern.level :**定义渲染不同级别日志的格式。默认是%5p.
 **logging.exception-conversion-word :**.定义当日志发生异常时的转换字
 **PID :**定义当前进程的ID

**logging.level.\*：**

```undefined
logging.level.root= WARN
logging.level.org.springframework.security= DEBUG
logging.level.org.springframework.web= ERROR
logging.level.org.hibernate= DEBUG
logging.level.org.apache.commons.dbcp2= DEBUG 
```

**logging.file：**

Spring Boot 默认把日志输入到console，如果我们要把日志输入到文件中，需要配置logging.file 或者logging.path属性性。logging.file属性用来定义文件名。也可以是路径+文件名。

```cpp
logging.level.org.springframework.security= DEBUG
logging.level.org.hibernate= DEBUG

logging.file = mylogfile.log ##新版本属性已经改变，使用loging.file.xxx属性
```

在这种情况下mylogfile.log将在根目录中创建。我们也可以为为mylogfile.log分配一个路径，如concretepage/mylogfile.log。这种情况下我们将在相对根目录下创建concretepage/mylogfile.log。也可以为日志文件配置绝对路径

**logging.path**

配置logging.path或者logging.path属性将日志输出到文件夹中。logging.path属性用来定义日志文件路径

```undefined
logging.level.org.springframework.security= DEBUG
logging.level.org.hibernate= DEBUG

logging.path = concretepage/logs  
```

将会相对根路径下创建concretepage/logs/spring.log  ,也可以配置绝对路径

**logging.patter.console**

通过设置logging.patter.console属性我们能改变输出到console的日志样式。日志样式包括时间，日志级别，线程名，日志名以及消息。可以按我们的喜好改变日志样式。

```ruby
logging.level.org.springframework.security= DEBUG
logging.level.org.hibernate= DEBUG

logging.pattern.console= %d{yyyy-MMM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{15} - %msg%n
```

**logging.pattern.file**

改变文件中的日志样式我们需要设置logging.pattern.file属性。首先通过logging.file或logging.path属性，把日志记录到文件中。

```ruby
logging.level.org.springframework.security= DEBUG
logging.level.org.hibernate= DEBUG

logging.path = concretepage/logs
logging.pattern.file= %d{yyyy-MMM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{15} - %msg%n
logging.pattern.console= %d{yyyy-MMM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{15} - %msg%n  
```

通过logging.path属性将在根目录下创建concretepage/logs并默认使用spring.log作为文件名。logging.pattern.console是设置console的日志样式

#### 使用

日志是基于对象的，日志打印一般通过反射获取操控对象信息，在根据内部使用打印日志。以slf4j为例：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UtilsTest {

    private static final Logger logger= LoggerFactory.getLogger(UtilsTest.class);//根据对象获取日志，打印日志
    @Test
    public void logTest(){
        logger.info("这是一条 info 级别的日志");
        logger.error("这是一条 error 级别的日志");
        logger.debug("这是一条 debug 级别的日志");
        logger.warn("这是一条 warn 级别的日志");
    }
}
```

使用lombok可以简化去掉全局logger

```java
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class UtilsTest {
    @Test
    public void logTest(){
        log.info("这是一条 info 级别的日志");
        log.error("这是一条 error 级别的日志");
        log.debug("这是一条 debug 级别的日志");
        log.warn("这是一条 warn 级别的日志");
    }
}
```



### 使用jsp作为模板

```xml
<!-- jsp -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>
<dependency>
    <!--embed-jasper用于处理jsp-->
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
```

在application.properties中配置视图前后缀与Tomcat编码：

```properties
## jsp
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
# 配置Tomcat编码
server.tomcat.uri-encoding=UTF-8
```

对应的，application.properties的默认相对路径为src/main/resoucres/，创建对应的webapp/WEB-INF/jsp目录，启动页为index.jsp，写对应的controller跳转即可

**找不到jsp**

404问题，找不到Jsp,springboot默认不会启用某块目录，因此不会根据模块路径找webapp，找不到，则需要手动在IDEA中进行配置，打包发布的时候要注意打模块war包，参看:

 https://blog.csdn.net/slg1988/article/details/88980390 

**jsp不加载静态资源文件**

将静态资源放在resources/static/下，引用时的默认路径为/static/，因此引用时不加/static/

 https://blog.csdn.net/f2315895270/article/details/83380845 

#### c:forEach

用于循环迭代，语法为：

```jsp
<c:forEach var="每个变量名字"   items="要迭代的list"   varStatus="每个对象的状态"   begin="循环从哪儿开始"    end="循环到哪儿结束"    step="循环的步长">
    循环要输出的东西
</c:forEach>
```

举例：

```jsp
<c:forEach var="item" items="${list}">
   <span>${item.id}</span>
   <span>${item.name}</span><br/>
</c:forEach>
```

```jsp
<!--固定迭代，指定大小-->
<c:forEach var="x" begin="1"end="9" step="1"> 
    ${x*x} 
</c:forEach>
```

### 测试

测试时一般使用junit，如果让整个SpringBoot项目启动的话需要在测试类上加上注解以读取信息

```java
@RunWith(SpringRunner.class)
@SpringBootTest
```

### 文件上传

#### 前端

文件上传时，仍以表单提交

```html
<form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="file" />
    <input type="submit" value="文件上传" />
</form>
```

并且必须选择post与enctype="multipart/form-data，这样才会把文件数据传到请求body中，否则只是单纯的传递name字段，即 只有使用`enctype="multipart/form-data"`,表单才会把文件的内容编码到HTML请求中。可以看做是一种标准 

#### controller

```java
@Controller
public class UploadRequestTest {

    private final static String FILE_UPLOAD_PATH="";//文件上传路径，最好是绝对路径

    @PostMapping(value = "/upload")
    @ResponseBody
    public Result upload(@RequestParam("file")MultipartFile file){
        if(file.isEmpty()){
            return new Result(Constants.RESULT_CODE_BAD_REQUEST,"文件上传失败");
        }

        String fileName=file.getOriginalFilename();//获取原始文件名
        String suffixName=fileName.substring(fileName.lastIndexOf("."));//截取文件后缀
        //对于上传的文件生成文件名称，其实应该使用文件唯一标识符进行标识，这样做还是可能导致文件覆盖
        SimpleDateFormat simpleDateFormat=new SimpleDateFormat("yyyyMMdd_HHmmss");
        Random random=new Random();
        String tmpFileName=simpleDateFormat.format(new Date())+random.nextInt(1000)+suffixName;//生成文件名称

        //保存文件到本地
        try {
            byte[] bytes=file.getBytes();//获取文件数据
            Path path= Paths.get(FILE_UPLOAD_PATH+tmpFileName);//生成文件路径
            Files.write(path,bytes);
        }catch (IOException e){
            e.printStackTrace();
        }
        return new Result(Constants.RESULT_CODE_SUCCESS,"文件上传成功");
    }
}
```

MultipartFile是一个接口，表示一个前端文件实体，上传的文件最好以绝对路径保存

#### WebMvcConfigurer

接口，整个web控制的抽象功能定义，提供一系列方法以供重写

##### 配置视图解析器

```java
@Override
public void configureViewResolvers(ViewResolverRegistry registry) {
    registry.jsp("/WEB-INF/jsp/", ".jsp");
    registry.enableContentNegotiation(new MappingJackson2JsonView());
}
```

## 文件传输

### FormData-MultipartFile

前端使用formdata传递数据，后台使用MultipartFile映射，可以直接映射

```java
    public ReturnType postAudio(@PathVariable long courseId,
                                @PathVariable long startTime,
                                MultipartFile multipartFile)
```

注意名称必须是文件名称才可以映射

### 使用拦截器

#### HandlerInterceptor

通过实现HandlerInterceptor接口或继承实现该接口的类进行拦截方法处理，之后要向Spring框架注册拦截器。

Spring MVC的拦截器（Interceptor）和Filter不同，但是也可以实现对请求进行预处理，后处理。先介绍它的使用，只需要两步：
1.1 实现拦截器
实现拦截器可以通过继承HandlerInterceptorAdapter类。如果preHandle方法return true，则继续后续处理。

```java
public class InterceptorDemo extends HandlerInterceptorAdapter {


    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        StringBuffer requestURL = httpServletRequest.getRequestURL();
        System.out.println("前置拦截器1 preHandle： 请求的uri为："+requestURL.toString());
        return true;
    }
    
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器1 postHandle： ");
    }
    
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("拦截器1 afterCompletion： ");
    }

}
```



1.2 注册拦截器
实现拦截器后还需要将拦截器注册到spring容器中，可以通过implements WebMvcConfigurer，覆盖其addInterceptors(InterceptorRegistry registry)方法。记得把Bean注册到Spring容器中，可以选择@Component 或者 @Configuration。

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer{

	 /**
	 * 注册自定义拦截器
	 */
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
	    
	   	registry.addInterceptor(new InterceptorDemo2()).addPathPatterns("/**");
	    registry.addInterceptor(new InterceptorDemo()).addPathPatterns("/**");
	}

 }


```



注意这里注册了两个拦截器。这两个拦截器的执行顺序和配置顺序有关系，即先配置顺序就在前（感觉这样不太方便，但没有找到设置类似order的API）。

发起一个请求，在控制台可以看到拦截器生效：

前置拦截器2 preHandle： 用户名：null
前置拦截器1 preHandle： 请求的uri为：http://localhost:8010/user/353434
拦截器1 postHandle： 
拦截器2 postHandle： 
拦截器1 afterCompletion： 
拦截器2 afterCompletion：

    1
    2
    3
    4
    5
    6

1.3 拦截器的总结
1.3.1 工作原理
一个拦截器，只有preHandle方法返回true，postHandle、afterCompletion才有可能被执行；如果preHandle方法返回false，则该拦截器的postHandle、afterCompletion必然不会被执行。拦截器不是Filter，却实现了Filter的功能，其原理在于：

    所有的拦截器(Interceptor)和处理器(Handler)都注册在HandlerMapping中。
    Spring MVC中所有的请求都是由DispatcherServlet分发的。
    当请求进入DispatcherServlet.doDispatch()时候，首先会得到处理该请求的Handler（即Controller中对应的方法）以及所有拦截该请求的拦截器。拦截器就是在这里被调用开始工作的。

1.3.2 拦截器工作流程

    正常流程：
    
      	1.Interceptor2.preHandle
      	2.Interceptor1.preHandle
      	3.Controller处理请求
      	4.Interceptor1.postHandle
      	5.Interceptor2.postHandle
      	6.渲染视图view
      	2.Interceptor1.afterCompletion
      	2.Interceptor2.afterCompletion
        1
        2
        3
        4
        5
        6
        7
        8
    
    中断流程
    如果在Interceptor1.preHandle中报错或返回false ,那么接下来的流程就会被中断，但注意被执行过的拦截器的afterCompletion仍然会执行。下图为Interceptor1.preHandle返回false的情况：
    
    前置拦截器2 preHandle： 用户名：null
    前置拦截器1 preHandle： 请求的uri为：http://localhost:8010/user/353434
    拦截器2 afterCompletion：
        1
        2
        3

1.3.3 和Filter共存时的执行顺序
拦截器是在DispatcherServlet这个servlet中执行的，因此所有的请求最先进入Filter，最后离开Filter。其顺序如下。

Filter->Interceptor.preHandle->Handler->Interceptor.postHandle->Interceptor.afterCompletion->Filter

1.3.4 应用场景
拦截器本质上是面向切面编程（AOP），符合横切关注点的功能都可以放在拦截器中来实现，主要的应用场景包括：

    登录验证，判断用户是否登录。
    权限验证，判断用户是否有权限访问资源，如校验token
    日志记录，记录请求操作日志（用户ip，访问时间等），以便统计请求访问量。
    处理cookie、本地化、国际化、主题等。
    性能监控，监控请求处理时长等。

### 事务管理

#### @Transactional

 使用 `@Transactional` 注解来声明一个函数需要被事务管理 ，调用的仍是JDBC等平台提供的事务管理方法，当事务执行出现错误时执行回滚。

SpringBoot 使用事务非常简单，首先使用注解 @EnableTransactionManagement 开启事务支持后，然后在访问[数据库](http://lib.csdn.net/base/mysql)的Service方法上添加注解 @Transactional 便可 

### SpringBoot项目部署（以jar包方式)

https://www.cnblogs.com/joeblackzqq/p/10788684.html

1.pom中进行配置

 最下面的build这块一定要配置否则打jar的时候会说找不到主类 

 ![image.png](https://upload-images.jianshu.io/upload_images/5548226-3249288d08ff1638.png?imageMogr2/auto-orient/strip) 

在启动类当中加上extends SpringBootServletInitializer并重写configure方法，这是为了打包springboot项目用的。

2.主应用中配置

```
package com.weixin;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

@SpringBootApplication
public class SmallsystemApplication extends SpringBootServletInitializer{

    public static void main(String[] args) {
        SpringApplication.run(SmallsystemApplication.class, args);
    }

    @Override//为了打包springboot项目
    protected SpringApplicationBuilder configure(
            SpringApplicationBuilder builder) {
        return builder.sources(this.getClass());
    }
}
```

3.Maven生成

然后按照顺序运行mvn clean再mvn install,用idea执行

 

![image.png](https://upload-images.jianshu.io/upload_images/5548226-516f058780d17553.png?imageMogr2/auto-orient/strip)

然后就会出来我们需要的jar

![image.png](https://upload-images.jianshu.io/upload_images/5548226-d91c19cfebeee328.png?imageMogr2/auto-orient/strip)

4.服务器端运行：

Linux 运行jar包命令如下：

方式一

```
java -jar shareniu.jar
```

特点：当前ssh窗口被锁定，可按CTRL + C打断程序运行，或直接关闭窗口，程序退出

那如何让窗口不锁定？

方式二

```
java -jar shareniu.jar &
```

&代表在后台运行。

特定：当前ssh窗口不被锁定，但是当窗口关闭时，程序中止运行。

继续改进，如何让窗口关闭时，程序仍然运行？

方式三

```
nohup java -jar shareniu.jar &
```

nohup 意思是不挂断运行命令,当账户退出或终端关闭时,程序仍然运行

当用 nohup 命令执行作业时，缺省情况下该作业的所有输出被重定向到nohup.out的文件中，除非另外指定了输出文件。

方式四

 grant  ALL PRIVILEGES ON  *.*  to root@127.0.0.1 indentified by "123456"; 

nohup java -jar web-0.0.1-SNAPSHOT.jar >web.txt  &  

```
nohup java -jar shareniu.jar >/dev/null  &  
```

解释下 >temp.txt

command >out.file

command >out.file是将command的输出重定向到out.file文件，即输出内容不打印到屏幕上，而是输出到out.file文件中。

可通过jobs命令查看后台运行任务

```
jobs
```

那么就会列出所有后台执行的作业，并且每个作业前面都有个编号。
如果想将某个作业调回前台控制，只需要 fg + 编号即可。

```
fg 23
```

查看某端口占用的线程的pid

```
netstat -nlp |grep :9181
```

### 常见错误

#### Failed to configure a DataSource: 'url' attribute is not specified and no embedded datas

因为创建项目是勾选导入了mysql,jdbc等jar依赖，但最初没有在application.properties中配置数据源，因此报数据集绑定错误，可以现在主应用下使用

```
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
```

忽视数据集配置，之后真正使用数据库时删掉就行

#### org.springframework.context.ApplicationContextException:Unable to start Embedded

因为有一些类实现了WebMvcConfigurer接口，但是并没有配置文件配这个类，导致spring工厂绑定不了这个配置，在根应用上加上@SpringBootApplication即可，进行自动配置发现

```java
@EnableAutoConfiguration
@MapperScan("lsl.java.web.mapper")
public class WebApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebApplication.class, args);
    }

}
```

@SpringBootApplication包括@EnableAutoConfiguratio





```
package org.phoenix.aladdin.app.business.controller;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.phoenix.aladdin.app.business.viewobject.PackVO;
import org.phoenix.aladdin.constant.ExpressInfo;
import org.phoenix.aladdin.controller.BaseController;
import org.phoenix.aladdin.error.BusinessException;
import org.phoenix.aladdin.error.EmBusinessError;
import org.phoenix.aladdin.model.entity.Express;
import org.phoenix.aladdin.response.CommonReturnType;
import org.phoenix.aladdin.service.ExpressService;
import org.phoenix.aladdin.service.PackageService;
import org.phoenix.aladdin.util.JSONUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

@RestController
@RequestMapping("/business")
public class BusinessExpressController extends BaseController {

    @Autowired
    private ExpressService expressService;

    @Autowired
    private PackageService packageService;
    /**
     * 前端发送请求表示预约已经上门，快件生命周期变更为 预约上门
     */
    @RequestMapping("/order/mailingDeal")
    public CommonReturnType updateMailingDeal(@RequestBody String id,
                                              HttpServletRequest request,
                                              HttpSession session) throws BusinessException {
        if(id==null)throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
        //获取订单id
        int orderId=Integer.parseInt(JSON.parseObject(id).getString("id"));
        //更新状态成功
        if(expressService.updateExpressStatus(orderId, ExpressInfo.EXPRESS_MAILING_DEAL)){
            return CommonReturnType.create(JSONUtil.oneMessageData("更新状态成功"));
        }
        //更新失败
        return CommonReturnType.create(
                JSONUtil.oneErrorData(EmBusinessError.SERVER_EXCEPTION_ERROR.getErrMsg()),
                EmBusinessError.SERVER_EXCEPTION_ERROR.getErrCode());
    }

    /**
     * 订单图片上传，使用第三方图床接口sm.ms后上传给服务器一个url
     **/
    @RequestMapping("/order/{orderId}/addImg")
    public CommonReturnType addOrderImg(@PathVariable("orderId")String orderId,
                                        @RequestBody String imgUrl,
                                        HttpServletRequest request,
                                        HttpSession session)throws BusinessException{
        if(orderId==null||imgUrl==null)throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
        imgUrl=JSON.parseObject(imgUrl).getString("imgUrl");

        if(expressService.insertExpressSendImg(Long.parseLong(orderId),imgUrl)){
            return CommonReturnType.create(JSONUtil.oneMessageData("上传成功"));
        }

        return CommonReturnType.create(JSONUtil.oneErrorData("上传失败"),10000);
    }


    /**
     * 包裹打包，快件/包裹的生命周期变更为 等待出库
     * 记录历史数据
     */
    //TODO mainId并没有进行约束
    @RequestMapping("/pack")
    public CommonReturnType packToPackage(@RequestBody PackVO packVO,
                                 HttpServletRequest request,
                                 HttpSession session)throws BusinessException{
        if(packVO==null||
                (packVO.getPackId()!=null&&packVO.getPassId()!=null))throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
        //将包裹打进最外层包裹，更新状态，记录历史
        boolean suc;
        if(packVO.getPackId()!=null){
            suc=packageService.packToPackage(Long.parseLong(packVO.getPackId())
                    ,1,Integer.parseInt(packVO.getMainId()));
            packageService.updatePackageStatusByPackageId(Long.parseLong(packVO.getPackId()),ExpressInfo.PACKAGE_WAIT_START);
        }else {//将快件打进最外层包裹，更新状态，记录历史
            suc=packageService.packToPackage(Integer.parseInt(packVO.getPassId())
                    ,0,Integer.parseInt(packVO.getMainId()));
            expressService.updateExpressStatus(Integer.parseInt(packVO.getPackId()),ExpressInfo.EXPRESS_WAIT_START);
        }
        if(suc)return CommonReturnType.create(JSONUtil.oneMessageData("打包成功"));

        return CommonReturnType.create(JSONUtil.oneErrorData("打包失败，请稍后重试"),
                10000);
    }

    /**
     * 包裹拆包，拆包则表明该包裹中的所有
     */
    public CommonReturnType unpackFromPackage(){
        return null;
    }


}
```