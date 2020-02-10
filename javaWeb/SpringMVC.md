### 基础概念

#### 媒体格式

即前端传输到后端的数据格式

常见媒体格式如下:

- text/html ： HTML格式
- text/plain ：纯文本格式
- text/xml ：  XML格式
- image/gif ：gif图片格式
- image/jpeg ：jpg图片格式
- image/png：png图片格式

以application开头的媒体格式类型：

- application/xhtml+xml ：XHTML格式
- application/xml     ： XML数据格式
- application/atom+xml  ：Atom XML聚合格式
- application/json    ： JSON数据格式
- application/pdf       ：pdf格式
- application/msword  ： Word文档格式
- application/octet-stream ： 二进制流数据（如常见的文件下载）
- application/x-www-form-urlencoded ： 中默认的encType，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）

#### 模板引擎

#### DipatcherServlet

SpringMVC基于ServletAPI构建，来源于Spring项目模块spring-webmvc

和其它WEB框架相同，SpringMVC也围绕着一个中心Servlet-DispatcherServlet，提供request的分发处理服务，该Servlet需在web.xml中进行配置或者直接在代码中进行初始化配置

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletCxt) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(AppConfig.class);
        ac.refresh();

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

上述代码等价于:

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>
```

即使用DispatcherServlet作为统一处理的Servlet，接收request，分配Handler交给对应的Controller处理并返回View，返回response。

![Spring MVC æ¡æ¶å¾](https://img-blog.csdn.net/20170207154527170?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzUyNDY2MjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

对于DispatcherServlet来说，光声明肯定不够，DispatcherServlet为完成其功能必须指定其处理器，视图解析器等信息，在上述xml中由下述声明指定

```xml
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>指定DispatcherServlet配置文件路径</param-value>
        </init-param>
```

#### @Controller/@RequestMapping与方法参数

DispatcherServlet需读取配置文件信息，配置文件可以是一个类，使用注解声明，如下

```java
@Configuration//声明这是一个配置class
@ComponentScan("org.example.web")//自动扫描指定package下注解
public class WebConfig {

    // ...
}
```

也可以使用xml文件，如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
<!--开启注解扫描-->
    <context:component-scan base-package="org.example.web"/>

    <!-- ... -->

</beans>
```

##### @RequestMapping 

用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径；用于方法上，表示在类的父路径下追加方法上注解中的地址将会访问到该方法。 @RequestMapping 默认响应所有HTTP方法，也可以指定HTTP方法如下，作用相同

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

##### @RequestBody

@requestBody注解常用来处理content-type不是默认的application/x-www-form-urlcoded编码的内容，比如说：application/json或者是application/xml等。一般情况下来说常用其来处理application/json类型。

 `@RequestBody` 这个注解的使用，使得REST接口接收的不再content-type为`application/x-www-form-urlencoded`的请求, 反而需要显示指定为`application/json` 

请求方法一般设置为POST，用在Controller方法参数内声明参数，使用@RequestBody声明参数变量获取json中的数据，json是k-v，@RequestBody注入的属性也是k-v，可以是map，也可以是基于setter的对象

 https://juejin.im/post/5b5efff0e51d45198469acea 

##### @Responsebody 

表示该方法的返回的结果直接写入 HTTP 响应正文（ResponseBody）中，一般在异步获取数据时使用； 注意要写在方法返回类型的前面，对于异步Ajax，注意要引入jquery

示例:

```js
    <!--ajax异步提交表单数据，注意要引入jquery-->
    function submitFormData(){
        var datas={//获取表单数据
            "email":$("#email").val(),
            "username":$("#username").val(),
            "password":$("#password").val()
        };
        $.ajax({
            type:"POST",
            async:true,
            url:"/home/register",
            data:JSON.stringify(datas),
            success:function (data) {
                alert(data);
            }
        });
    }
```

对于请求，有DispatcherServlet分发，由@RequestMapping 映射，返回响应体为

```java
    @RequestMapping(value = "/home/register",method = RequestMethod.POST)
    public  @ResponseBody String signUp(){
        return "success";//直接返回到response，一般要转换成json格式进行传输
    }
```

##### @RestController

Spring4之后新加入的注解，原来返回json需要@ResponseBody和@Controller配合。

即@RestController是@ResponseBody和@Controller的组合注解。需要声明class，即该控制器的所有返回都是直接返回response

##### @RequestParam
用于从request中接收请求，接收表单参数数据，适用于 application/x-www-form-urlcoded编码的内容 

http://localhost:8080/springmvc/hello/101?param1=10&param2=20

根据上面的这个URL，你可以用这样的方式来进行获取

```java
public String getDetails(
    @RequestParam(value="param1", required=true) String param1,
        @RequestParam(value="param2", required=false) String param2){
...
}
```

@RequestParam 支持下面四种参数

* defaultValue 如果本次请求没有携带这个参数，或者参数为空，那么就会启用默认值
* name 绑定本次参数的名称，要跟URL上面的一样
* required 这个参数是不是必须的
* value 跟name一样的作用，是name属性的一个别名

##### @PathVariable
这个注解能够识别URL里面的一个模板，也是用于从request中接收请求的,接收的是URI中携带的数据，使用{}在URL上引用，在参数中使用@PathVariable获取，我们看下面的一个URL

http://localhost:8080/springmvc/hello/101?param1=10&param2=20

```java
@RequestMapping("/hello/{id}")
    public String getDetails(@PathVariable(value="id") String id,
    @RequestParam(value="param1", required=true) String param1,
    @RequestParam(value="param2", required=false) String param2){
.......
}
```

#### Model/ModelAndView

Model用于数据传输，传参时实例化并在返回view时为模板注入属性(key-value)，有两种主要的数据:request数据与session数据，Model通过返回String，跳转视图，通过 addAttribute(String key,Object value) 设置参数，由mvc自动创建，直接传参即可

ModelAndView通过viewName设置视图名称，addObject设置参数，需手动创建。 都是用于给模板填充数据，即向View填充数据可使用两种方式：

1.返回值为String，参数为Model

2.返回值为ModelAndView，根据设置的ViewName返回对应的View

ModelAndView是LinkedHashMap的一个子类，需要实例化对象

##### **@ModelAttribute**

表示对model注入属性或从model中读取参数属性，有三种使用方式：

1. 每次执行方法时都会先执行@ModelAttribute注解的方法，并将结果添加到model中。 

```java
@ModelAttribute("top")
public Map top(){
    return pageTop.getDataMap();
}

@RequestMapping({"", "/", "/home"})
public String home(@RequestBody(required = false) Map<String, Object> param, Model model) {
    model.addAttribute("model", dataAssembly.homePageData(param));
    return "home";
}
```

执行home()前先执行top()将顶部模块数据放到model,在home返回的视图中包含top值，可以直接在页面获取.

2.用在方法参数上：

```java
@RequestMapping("/test")
public String test(@ModelAttribute("top") Map top, Model model) {
    JSONArray ja  = JSONArray.fromObject(map);
    return "test";
}
```

3.用在@ControllerAdvice方法中

```java
@ControllerAdvice
public class GlobalModelData {
    @ModelAttribute
    public Object globalUser() {
        User user = new User();
        user.setUn("xxx");
        return user;
        *//*这里在controller执行前将返回值填充到model中,则可以在model中获取数据*//*
    } 
}
```

 每个Controller中的方法执行前都会先执行 @ModelAttribute注解标注的方法，并将返回值添加到model 

##### **@SessionAttributes**

 @sessionAttributes注解应用到Controller上面，可以将Model中的属性同步到session当中。 

```java
@Controller
@RequestMapping("/Demo.do")
@SessionAttributes(value={"attr1","attr2"})
public class Demo {
    
    @RequestMapping(params="method=index")
    public ModelAndView index() {
        ModelAndView mav = new ModelAndView("index.jsp");
        mav.addObject("attr1", "attr1Value");
        mav.addObject("attr2", "attr2Value");
        return mav;
    }
    
    @RequestMapping(params="method=index2")
    public ModelAndView index2(@ModelAttribute("attr1")String attr1, @ModelAttribute("attr2")String attr2) {
        ModelAndView mav = new ModelAndView("success.jsp");
        return mav;
    }
}
```

index方法返回一个ModelAndView 其中包括视图index.jsp 和 两个键值放入model当中，在没有加入@sessionattributes注解的时候，放入model当中的键值是request级别的。

现在因为在Controller上面标记了@SessionAttributes(value={"attr1","attr2"}) 那么model中的attr1,attr2会同步到session中，这样当你访问index 然后在去访问index2的时候也会获取这俩个属性的值。

当需要清除session当中的值的时候，我们只需要在controller的方法中传入一个SessionStatus的类型对象 通过调用setComplete方法就可以清除了。

```java
@RequestMapping(params="method=index3")
　　public ModelAndView index4(SessionStatus status) {
　　ModelAndView mav = new ModelAndView("success.jsp");
　　status.setComplete();
　　return mav;
}
```

##### @SessionAttribute

@SessionAttribute用于读取model中的值

```java
@RequestMapping(value="/checkA", method=RequestMethod.GET)
public String checkA(Model model, HttpSession session, @SessionAttribute("userId") String userId,
			@ModelAttribute("cityid") String cityid) {
    .......
   }
```

#### 返回类型

##### String

返回的是View名称

##### ModelAndView

返回的是ModelAndView中指定的跳转页面

##### Map/Void

返回的页面就是请求的页面

### JSP中使用Model数据

直接通过${modelAttribute}即可取值，取值优先级为map,mapModel,session

 



#### 处理方法-Handler Methods

##### 方法参数-Method Arguments

**获取请求request**

有三种方式

1. 直接传入参数 HttpServletRequest request 即可，线程安全
2. 使用注解，直接在控制器中声明一个request，线程安全

```java
 @Autowired
  private HttpServletRequest request; //自动注入request
```

3. 手动调用，线程安全

```java
  HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();
```



### 测试

maven引入依赖

```xml
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.1.9.RELEASE</version>
    </dependency>
```





### 常见错误

 #####   java.lang.NoSuchMethodError 

java.lang.NoSuchMethodError: org.springframework.util.Assert.notNull 

检查spring家族版本

