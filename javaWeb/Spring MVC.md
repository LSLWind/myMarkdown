## Spring MVC

### 处理流程

SpringMVC基于ServletAPI构建，来源于Spring项目模块spring-webmvc，Spring MVC的基本处理流程为

<img src="https://s2.ax1x.com/2020/02/17/3Ct8FH.png" alt="3Ct8FH.png" style="zoom:67%;" />

在请求离开浏览器时，会带有用户所请求内容的信息，至少会包含请求的URL。但是还可能带有其他的信息，例如用户提交的表单信息。请求旅程的第一站是Spring的`DispatcherServlet`。与大多数基于Java的Web框架一样，Spring MVC所有的请求都会通过一个前端控制器（front controller）Servlet。前端控制器是常用的Web应用程序模式，在这里一个单实例的Servlet将请求委托给应用程序的其他组件来执行实际的处理。在Spring MVC中，`DispatcherServlet`就是前端控制器。

`DispatcherServlet`的任务是将请求发送给Spring MVC控制器（controller）。控制器是一个用于处理请求的Spring组件。在典型的应用程序中可能会有多个控制器，`DispatcherServlet`需要知道应该将请求发送给哪个控制器。所以`DispatcherServlet`以会查询一个或多个处理器映射（handler mapping）来确定请求的下一站在哪里。处理器映射会根据请求所携带的URL信息来进行决策。

一旦选择了合适的控制器，`DispatcherServlet`会将请求发送给选中的控制器。到了控制器，请求会卸下其负载（用户提交的信息）并耐心等待控制器处理这些信息。（实际上，设计良好的控制器本身只处理很少甚至不处理工作，而是将业务逻辑委托给一个或多个服务对象进行处理。）

控制器在完成逻辑处理后，通常会产生一些信息，这些信息需要返回给用户并在浏览器上显示。这些信息被称为模型（model）。不过仅仅给用户返回原始的信息是不够的——这些信息需要以用户友好的方式进行格式化，一般会是HTML。所以，信息需要发送给一个视图（view），通常会是JSP。

控制器所做的最后一件事就是将模型数据打包，并且标示出用于渲染输出的视图名。它接下来会将请求连同模型和视图名发送回`DispatcherServlet`。

这样，控制器就不会与特定的视图相耦合，传递给`DispatcherServlet`的视图名并不直接表示某个特定的JSP。实际上，它甚至并不能确定视图就是JSP。相反，它仅仅传递了一个逻辑名称，这个名字将会用来查找产生结果的真正视图。`DispatcherServlet`将会使用视图解析器（view resolver）来将逻辑视图名匹配为一个特定的视图实现，它可能是也可能不是JSP。

既然`DispatcherServlet`已经知道由哪个视图渲染结果，那请求的任务基本上也就完成了。它的最后一站是视图的实现（可能是JSP），在这里它交付模型数据。请求的任务就完成了。视图将使用模型数据渲染输出，这个输出会通过响应对象传递给客户端（不会像听上去那样硬编码）。

### 搭建Spring MVC

**1.配置DispatcherServlet**

`DispatcherServlet`是Spring MVC的核心。在这里请求会第一次接触到框架，它要负责将请求路由到其他的组件之中。

按照传统的方式，像`DispatcherServlet`这样的Servlet会配置在web.xml文件中，这个文件会放到应用的WAR包里面。当然，这是配置`DispatcherServlet`的方法之一，在这里，我们会使用Java将`DispatcherServlet`配置在Servlet容器中，而不会再使用web.xml文件。

<img src="https://s2.ax1x.com/2020/02/17/3CNwg1.png" alt="3CNwg1.png" style="zoom:67%;" />

扩展`AbstractAnnotation-ConfigDispatcherServletInitializer`的任意类都会自动地配置`Dispatcher-Servlet`和Spring应用上下文，Spring的应用上下文会位于应用程序的Servlet上下文之中。

**AbstractAnnotationConfigDispatcherServletInitializer剖析**

如果你坚持要了解更多细节的话，那就看这里吧。在Servlet 3.0环境中，容器会在类路径中查找实现`javax.servlet.ServletContainerInitializer`接口的类，如果能发现的话，就会用它来配置Servlet容器。

Spring提供了这个接口的实现，名为`SpringServletContainerInitializer`，这个类反过来又会查找实现`WebApplicationInitializer`的类并将配置的任务交给它们来完成。Spring 3.2引入了一个便利的`WebApplicationInitializer`基础实现，也就是`AbstractAnnotationConfigDispatcherServletInitializer`。因为我们的`Spittr-WebAppInitializer`扩展了`AbstractAnnotationConfig DispatcherServlet-Initializer`（同时也就实现了`WebApplicationInitializer）`，因此当部署到Servlet 3.0容器中的时候，容器会自动发现它，并用它来配置Servlet上下文。

尽管它的名字很长，但是`AbstractAnnotationConfigDispatcherServlet-Initializer`使用起来很简便。

在上述例子中，第一个方法是`getServletMappings()`，它会将一个或多个路径映射到`DispatcherServlet`上。在本例中，它映射的是“/”，这表示它会是应用的默认Servlet。它会处理进入应用的所有请求。

为了理解其他的两个方法，我们首先要理解`DispatcherServlet`和一个Servlet监听器（也就是`ContextLoaderListener`）的关系。

**两个应用上下文之间的故事**

当`DispatcherServlet`启动的时候，它会创建Spring应用上下文，并加载配置文件或配置类中所声明的bean。在上例的`getServletConfigClasses()`方法中，我们要求`DispatcherServlet`加载应用上下文时，使用定义在`WebConfig`配置类（使用Java配置）中的bean。

但是在Spring Web应用中，通常还会有另外一个应用上下文。另外的这个应用上下文是由`ContextLoaderListener`创建的。

我们希望`DispatcherServlet`加载包含Web组件的bean，如控制器、视图解析器以及处理器映射，而C`ontextLoaderListener`要加载应用中的其他bean。这些bean通常是驱动应用后端的中间层和数据层组件。

实际上，`AbstractAnnotationConfigDispatcherServletInitializer`会同时创建`DispatcherServlet`和`ContextLoaderListener`。`GetServlet-ConfigClasses()`方法返回的带有`@Configuration`注解的类将会用来定义`DispatcherServlet`应用上下文中的bean。`getRootConfigClasses()`方法返回的带有`@Configuration`注解的类将会用来配置`ContextLoaderListener`创建的应用上下文中的bean。

在本例中，根配置定义在`RootConfig`中，`DispatcherServlet`的配置声明在`WebConfig`中，这两个配置类都需要自己进行配置来自定义Web服务。

如果按照这种方式配置`DispatcherServlet`，而不是使用web.xml的话，那唯一问题在于它只能部署到支持Servlet 3.0的服务器中才能正常工作，如Tomcat 7或更高版本。

**2.启用Spring MVC**

我们有多种方式来配置`DispatcherServlet`，与之类似，启用Spring MVC组件的方法也不仅一种。以前，Spring是使用XML进行配置的，可以使用`<mvc:annotation-driven>`启用注解驱动的Spring MVC。

我们所能创建的最简单的Spring MVC配置就是一个带有`@EnableWebMvc`注解的类：

```java
@Configuration
@EnableWebMvc
public class WebConfig {
}
```

这可以运行起来，它的确能够启用Spring MVC，但还有不少问题要解决：

- 没有配置视图解析器。如果这样的话，Spring默认会使用`BeanNameView-Resolver`，这个视图解析器会查找ID与视图名称匹配的bean，并且查找的bean要实现`View`接口，它以这样的方式来解析视图。
- 没有启用组件扫描。这样的结果就是，Spring只能找到显式声明在配置类中的控制器。
- 这样配置的话，`DispatcherServlet`会映射为应用的默认Servlet，所以它会处理所有的请求，包括对静态资源的请求，如图片和样式表（在大多数情况下，这可能并不是你想要的效果）。

<img src="https://s2.ax1x.com/2020/02/17/3COCjJ.png" alt="3COCjJ.png" style="zoom:67%;" />

`WebConfig`现在添加了`@Component-Scan`注解，因此将会扫描spitter.web包来查找组件。稍后你就会看到，我们所编写的控制器将会带有`@Controller`注解，这会使其成为组件扫描时的候选bean。因此，我们不需要在配置类中显式声明任何的控制器。

接下来，我们添加了一个`ViewResolver` bean。更具体来讲，是`Internal-ResourceViewResolver`。我们将会在第6章更为详细地讨论视图解析器。我们只需要知道它会查找JSP文件，在查找的时候，它会在视图名称上加一个特定的前缀和后缀（例如，名为`home`的视图将会解析为/WEB-INF/views/home.jsp）。

最后，新的`WebConfig`类还扩展了`WebMvcConfigurerAdapter`并重写了其`configureDefaultServletHandling()`方法。通过调用`DefaultServlet-HandlerConfigurer`的`enable()`方法，我们要求`DispatcherServlet`将对静态资源的请求转发到Servlet容器中默认的Servlet上，而不是使用`DispatcherServlet`本身来处理此类请求。

`WebConfig`已经就绪，那`RootConfig`呢？因为本章聚焦于Web开发，而Web相关的配置通过`DispatcherServlet`创建的应用上下文都已经配置好了，因此现在的`RootConfig`相对很简单：

```java
@Configuration
@ComponentScan(basePackages={"spitter"},
    excludeFilters={
        @Filter(type=FilterType.ANNOTATION, value=EnableWebMvc.class)
    })
public class RootConfig {
}
```

### 编写控制器

```java
@Controller
public class Test {
    @RequestMapping(value = "/",method = GET)
    public String home(){
        return "home";
    }
}
```

#### @Conttroller

这个注解是用来声明控制器的，但实际上这个注解对Spring MVC本身的影响并不大。

#### @RequestMapping

`@RequestMapping`注解的`value`属性指定了这个方法所要处理的请求路径，`method`属性细化了它所处理的HTTP方法。在本例中，当收到对“/”的HTTP `GET`请求时，就会调用`home()`方法。

`home()`方法其实并没有做太多的事情：它返回了一个`String`类型的“home”。这个`String`将会被Spring MVC解读为要渲染的视图名称。`DispatcherServlet`会要求视图解析器将这个逻辑名称解析为实际的视图。

鉴于我们配置`InternalResourceViewResolver`的方式，视图名“home”将会解析为“/WEB-INF/views/home.jsp”路径的JSP。

`@RequestMapping`注解。它的`value`属性指定了这个方法所要处理的请求路径，`method`属性细化了它所处理的HTTP方法。在本例中，当收到对“/”的HTTP `GET`请求时，就会调用`home()`方法。

你可以看到，`home()`方法其实并没有做太多的事情：它返回了一个`String`类型的“home”。这个`String`将会被Spring MVC解读为要渲染的视图名称。`DispatcherServlet`会要求视图解析器将这个逻辑名称解析为实际的视图。

鉴于我们配置`InternalResourceViewResolver`的方式，视图名“home”将会解析为“/WEB-INF/views/home.jsp”路径的JSP。

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<html>
  <head>
    <title>Spittr</title>
    <link rel="stylesheet"
          type="text/css"
          href="<c:url value="/resources/style.css" />" >
  </head>
  <body>
    <h1>Welcome to Spittr</h1>
    <a href="<c:url value="/spittles" />">Spittles</a> |
    <a href="<c:url value="/spitter/register" />">Register</a>
  </body>
</html>
```

`@RequestMapping`可以应用于类上，表示所有控制器方法的映射的url的共同前缀

```java
@Controller
@RequestMapping("/")
public class HomeController {
    @RequestMapping(method = GET)
    public String home(){
        return "home";
    }
}
```

`@RequestMapping`也可以映射多个url，如`@RequestMapping({"/", "/homepage"})`

控制方法的返回类型为：

* String：返回的是View名称
* ModelAndView：返回的是ModelAndView中指定的跳转页面
* Map/Void：返回的页面就是请求的页面

#### 传递模型数据到视图中

对于控制器方法参数，可以传递一个Model对象作为参数，使用该参数可以附加k-v数据并传递到视图当中使用。使用addAttribute()方法可增加k-v

#### 接受url查询参数@RequestParam

对于URL中使用?与&附加的查询参数信息，可以使用注解`@RequestParam`进行参数映射

```java
@RequestMapping(method=RequestMethod.GET)
public List<Spittle> spittles(
    @RequestParam(value="max",
                  defaultValue=MAX_LONG_AS_STRING) long max,
    @RequestParam(value="count", defaultValue="20") int count) {
  return spittleRepository.findSpittles(max, count);
}
```

defaultValue表明没有值则使用默认值，value表示url传来的查询信息

#### 接收Rest风格查询参数@PathVariable

Spring MVC允许我们在`@RequestMapping`路径中添加占位符。占位符的名称要用大括号（“`{`”和“`}`”）括起来，可以实现Rest风格的url路径映射

```java
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle( @PathVariable("spittleId") long spittleId,Model model) {
  	model.addAttribute(spittleRepository.findOne(spittleId));
  	return "spittle";
}
```

#### 处理表单/使用API校验表单

接收前台的表单信息很简单，以Post形式发送的表单将直接放在request的报文中，对于表单来说，在后台要有一个与之对应的DAO，直接将DAO作为参数即可，该参数会使用报文中表单的同名的信息对DAO进行填充。

同时，对于表单数据来说，经常会遇到校验表单值的操作，虽说可以往里面插入校验代码，但是Spring MVC已经内置了一些校验API，都通过注解的方式进行校验，非常方便。

| 注　　解       | 描　　述                                                     |
| -------------- | ------------------------------------------------------------ |
| `@AssertFalse` | 所注解的元素必须是Boolean类型，并且值为`false`               |
| `@AssertTrue`  | 所注解的元素必须是Boolean类型，并且值为`true`                |
| `@DecimalMax`  | 所注解的元素必须是数字，并且它的值要小于或等于给定的`BigDecimalString`值 |
| `@DecimalMin`  | 所注解的元素必须是数字，并且它的值要大于或等于给定的`BigDecimalString`值 |
| `@Digits`      | 所注解的元素必须是数字，并且它的值必须有指定的位数           |
| `@Future`      | 所注解的元素的值必须是一个将来的日期                         |
| `@Max`         | 所注解的元素必须是数字，并且它的值要小于或等于给定的值       |
| `@Min`         | 所注解的元素必须是数字，并且它的值要大于或等于给定的值       |
| `@NotNull`     | 所注解元素的值必须不能为`null`                               |
| `@Null`        | 所注解元素的值必须为`null`                                   |
| `@Past`        | 所注解的元素的值必须是一个已过去的日期                       |
| `@Pattern`     | 所注解的元素的值必须匹配给定的正则表达式                     |
| `@Size`        | 所注解的元素的值必须是`String`、集合或数组，并且它的长度要符合给定的范围 |

[<img src="https://s2.ax1x.com/2020/02/18/3kTnud.png" alt="3kTnud.png" style="zoom:67%;" />](https://imgchr.com/i/3kTnud)

同时必须开启校验与传入错误参数才能进行校验，使用注解@Valid与类Errors

[<img src="https://s2.ax1x.com/2020/02/18/3kTrCT.png" alt="3kTrCT.png" style="zoom:67%;" />](https://imgchr.com/i/3kTrCT)

如果有校验出现错误的话，那么这些错误可以通过`Errors`对象进行访问，现在这个对象已作为`processRegistration()`方法的参数。（很重要的一点需要注意，`Errors`参数要紧跟在带有`@Valid`注解的参数后面，`@Valid`注解所标注的就是要检验的参数。）`processRegistration()`方法所做的第一件事就是调用`Errors.hasErrors()`来检查是否有错误。

## 渲染Web视图

### 视图解析

将控制器中请求处理的逻辑和视图中的渲染实现解耦是Spring MVC的一个重要特性。如果控制器中的方法直接负责产生HTML的话，就很难在不影响请求处理逻辑的前提下，维护和更新视图。控制器方法和视图的实现会在模型内容上达成一致，这是两者的最大关联，除此之外，两者应该保持足够的距离。

但是，如果控制器只通过逻辑视图名来了解视图的话，那Spring该如何确定使用哪一个视图实现来渲染模型呢？这就是Spring视图解析器的任务了。

Spring MVC定义了一个名为`ViewResolver`的接口，它大致如下所示：

```java
public interface ViewResolver {
  View resolveViewName(String viewName, Locale locale)throws Exception;
}
```

当给`resolveViewName()`方法传入一个视图名和`Locale`对象时，它会返回一个`View`实例。`View`是另外一个接口，如下所示：

```java
public interface View {
  String getContentType();
  void render(Map<String, ?> model,
              HttpServletRequest request,
              HttpServletResponse response) throws Exception;
}
```

`View`接口的任务就是接受模型以及Servlet的request和response对象，并将输出结果渲染到response中。

我们所需要做的就是编写`ViewResolver`和`View`的实现，将要渲染的内容放到response中，进而展现到用户的浏览器中。实际上，我们并不需要这么麻烦。尽管我们可以编写`ViewResolver`和`View`的实现，在有些特定的场景下，这样做也是有必要的，但是一般来讲，我们并不需要关心这些接口。Spring提供了多个内置的实现，它们能够适应大多数的场景，不同的实现能够解析不同的视图，如`InternalResourceViewResolver`一般会用于JSP，`FreeMarkerViewResolver`和`VelocityViewResolver`分别对应FreeMarker和Velocity模板视图。

### 配置JSP视图解析器

有一些视图解析器，如`ResourceBundleViewResolver`会直接将逻辑视图名映射为特定的`View`接口实现，而`InternalResourceViewResolver`所采取的方式并不那么直接。它遵循一种约定，会在视图名上添加前缀和后缀，进而确定一个Web应用中视图资源的物理路径。

作为样例，考虑一个简单的场景，假设逻辑视图名为`home`。通用的实践是将JSP文件放到Web应用的WEB-INF目录下，防止对它的直接访问。如果我们将所有的JSP文件都放在“/WEB-INF/views/”目录下，并且`home`页的JSP名为home.jsp，那么我们可以确定物理视图的路径就是逻辑视图名`home`再加上“/WEB-INF/views/”前缀和“.jsp”后缀。

<img src="https://s2.ax1x.com/2020/02/18/3kbNKe.png" alt="3kbNKe.png" style="zoom:33%;" />

当使用`@Bean`注解的时候，我们可以按照如下的方式配置`Internal-ResourceView Resolver`，使其在解析视图时，遵循上述的约定。

```java
@Bean
public ViewResolver viewResolver() {
  InternalResourceViewResolver resolver =new InternalResourceViewResolver();
  resolver.setPrefix("/WEB-INF/views/");
  resolver.setSuffix(".jsp");
  return resolver;
}
```

```xml
<bean id="viewResolver"
      class="org.springframework.web.servlet.view.InternalResourceViewResolver"
      p:prefix="/WEB-INF/views/"
      p:suffix=".jsp" />
```

`InternalResourceViewResolver`配置就绪之后，它就会将逻辑视图名解析为JSP文件，如下所示：

- `home`将会解析为“/WEB-INF/views/home.jsp”
- `productList`将会解析为“/WEB-INF/views/productList.jsp”
- `books/detail`将会解析为“/WEB-INF/views/books/detail.jsp”

## 处理异常

### Spring内置异常-状态码映射

| Spring异常                                | HTTP状态码                   |
| ----------------------------------------- | ---------------------------- |
| `BindException`                           | 400 - Bad Request            |
| `ConversionNotSupportedException`         | 500 - Internal Server Error  |
| `HttpMediaTypeNotAcceptableException`     | 406 - Not Acceptable         |
| `HttpMediaTypeNotSupportedException`      | 415 - Unsupported Media Type |
| `HttpMessageNotReadableException`         | 400 - Bad Request            |
| `HttpMessageNotWritableException`         | 500 - Internal Server Error  |
| `HttpRequestMethodNotSupportedException`  | 405 - Method Not Allowed     |
| `MethodArgumentNotValidException`         | 400 - Bad Request            |
| `MissingServletRequestParameterException` | 400 - Bad Request            |
| `MissingServletRequestPartException`      | 400 - Bad Request            |
| `NoSuchRequestHandlingMethodException`    | 404 - Not Found              |
| `TypeMismatchException`                   | 400 - Bad Request            |

### 自定义异常-状态码映射@ResponseStatus

使用**@ResponseStatus**注解，可以将异常映射为特定的状态码，就和Spring内置的异常状态码一样

```java
@ResponseStatus(value = HttpStatus.NOT_FOUND,reason = "Not Found")
class  SpittleNotFoundException extends RuntimeException{
}

RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(
    @PathVariable("spittleId") long spittleId,Model model) {
  Spittle spittle = spittleRepository.findOne(spittleId);
  if (spittle == null) {
    throw new SpittleNotFoundException();
  }
  model.addAttribute(spittle);
  return "spittle";
}
```

### 专门处理异常@ExceptionHandler

@ExceptionHandler用在一个控制器中的一个方法上，专门处理异常，只要该控制器中的方法抛出了指定异常，就会使用该方法进行处理，相当于替代了try...catch语句，用于简化代码

```java
@ExceptionHandler(DuplicateSpittleException.class)//传入一个触发的异常类
public String handleDuplicateSpittle() {
  return "error/duplicate";
}
@RequestMapping(method=RequestMethod.POST)
public String saveSpittle(SpittleForm form, Model model) {
  spittleRepository.save(
      new Spittle(null, form.getMessage(), new Date(),
          form.getLongitude(), form.getLatitude()));
  return "redirect:/spittles";
}
```

### 控制器通知@ControllerAdvice

控制器通知（controller advice）是任意带有`@ControllerAdvice`注解的类，这个类会包含一个或多个如下类型的方法：

- `@ExceptionHandler`注解标注的方法；
- `@InitBinder`注解标注的方法；
- `@ModelAttribute`注解标注的方法。

在带有`@ControllerAdvice`注解的类中，以上所述的这些方法会运用到整个应用程序所有控制器中带有`@RequestMapping`注解的方法上。

`@ControllerAdvice`注解本身已经使用了`@Component`，因此`@ControllerAdvice`注解所标注的类将会自动被组件扫描获取到，就像带有`@Component`注解的类一样。

`@ControllerAdvice`最为实用的一个场景就是将所有的`@ExceptionHandler`方法收集到一个类中，这样所有控制器的异常就能在一个地方进行一致的处理。例如，我们想将`DuplicateSpittleException`的处理方法用到整个应用程序的所有控制器上。

```java
@ControllerAdvice 
public class AppWideExceptionHandler{
    @ExceptionHandler(DuplicateSpittleException.class)//传入一个触发的异常类
    public String handleDuplicateSpittle() {
      return "error/duplicate";
    }
}
```

如上所示，如果任意的控制器方法抛出了`DuplicateSpittleException`，不管这个方法位于哪个控制器中，都会调用这个`duplicateSpittleHandler()`方法来处理异常。

## 进行Web测试

使用Spring提供的MockMvc可以很容易进行测试，添加依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```

一个简单的例子为

```java
class HomeControllerTest {
    @Test
    void home() {
        HomeController controller=new HomeController();
        MockMvc mockMvc= MockMvcBuilders.standaloneSetup(controller).build();

        try {
            mockMvc.perform(get("/")).andExpect(view().name("home"));
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

首先传递一个`HomeController`实例到`MockMvcBuilders.standaloneSetup()`并调用`build()`来构建MockMvc实例。然后它使用MockMvc实例来执行针对“/”的GET请求并设置期望得到的视图名称。

```text
@Test
public void testHello() throws Exception {
    /*
     * 1、mockMvc.perform执行一个请求。
     * 2、MockMvcRequestBuilders.get("XXX")构造一个请求。
     * 3、ResultActions.param添加请求传值
     * 4、ResultActions.accept(MediaType.TEXT_HTML_VALUE))设置返回类型
     * 5、ResultActions.andExpect添加执行完成后的断言。
     * 6、ResultActions.andDo添加一个结果处理器，表示要对结果做点什么事情
     *   比如此处使用MockMvcResultHandlers.print()输出整个响应结果信息。
     * 7、ResultActions.andReturn表示执行完成后返回相应的结果。
     */
    mockMvc.perform(MockMvcRequestBuilders
            .get("/hello")
            // 设置返回值类型为utf-8，否则默认为ISO-8859-1
            .accept(MediaType.APPLICATION_JSON_UTF8_VALUE)
            .param("name", "Tom"))
            .andExpect(MockMvcResultMatchers.status().isOk())
            .andExpect(MockMvcResultMatchers.content().string("Hello Tom!"))
            .andDo(MockMvcResultHandlers.print());
}
```

测试结果打印：

```text
FlashMap:
       Attributes = null
MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/json;charset=UTF-8", Content-Length:"10"]
     Content type = application/json;charset=UTF-8
             Body = Hello Tom!
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
2019-04-02 21:34:27.954  INFO 6937 --- [       Thread-2] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
```

整个过程如下：

1. 准备测试环境
2. 通过MockMvc执行请求
3. 添加验证断言
4. 添加结果处理器
5. 得到MvcResult进行自定义断言/进行下一步的异步请求
6. 卸载测试环境

注意事项：如果使用DefaultMockMvcBuilder进行MockMvc实例化时需在SpringBoot启动类上添加组件扫描的package的指定，否则会出现404。如：

```text
@ComponentScan(basePackages = "com.secbro2")
```



## 相关API

RequestBuilder提供了一个方法buildRequest(ServletContext servletContext)用于构建MockHttpServletRequest；其主要有两个子类MockHttpServletRequestBuilder和MockMultipartHttpServletRequestBuilder（如文件上传使用）。

MockMvcRequestBuilders提供get、post等多种方法用来实例化RequestBuilder。

ResultActions，MockMvc.perform(RequestBuilder requestBuilder)的返回值，提供三种能力：andExpect，添加断言判断结果是否达到预期；andDo，添加结果处理器，比如示例中的打印；andReturn，返回验证成功后的MvcResult，用于自定义验证/下一步的异步处理。



## 一些常用的测试

1.测试普通控制器

```text
mockMvc.perform(get("/user/{id}", 1)) //执行请求  
            .andExpect(model().attributeExists("user")) //验证存储模型数据  
            .andExpect(view().name("user/view")) //验证viewName  
            .andExpect(forwardedUrl("/WEB-INF/jsp/user/view.jsp"))//验证视图渲染时forward到的jsp  
            .andExpect(status().isOk())//验证状态码  
            .andDo(print()); //输出MvcResult到控制台
```

2.得到MvcResult自定义验证

```text
MvcResult result = mockMvc.perform(get("/user/{id}", 1))//执行请求  
        .andReturn(); //返回MvcResult  
Assert.assertNotNull(result.getModelAndView().getModel().get("user")); //自定义断言
```

3.验证请求参数绑定到模型数据及Flash属性

```text
mockMvc.perform(post("/user").param("name", "zhang")) //执行传递参数的POST请求(也可以post("/user?name=zhang"))  
            .andExpect(handler().handlerType(UserController.class)) //验证执行的控制器类型  
            .andExpect(handler().methodName("create")) //验证执行的控制器方法名  
            .andExpect(model().hasNoErrors()) //验证页面没有错误  
            .andExpect(flash().attributeExists("success")) //验证存在flash属性  
            .andExpect(view().name("redirect:/user")); //验证视图
```

4.文件上传

```text
byte[] bytes = new byte[] {1, 2};  
mockMvc.perform(fileUpload("/user/{id}/icon", 1L).file("icon", bytes)) //执行文件上传  
        .andExpect(model().attribute("icon", bytes)) //验证属性相等性  
        .andExpect(view().name("success")); //验证视图
```

5.JSON请求/响应验证

```text
String requestBody = "{\"id\":1, \"name\":\"zhang\"}";  
    mockMvc.perform(post("/user")  
            .contentType(MediaType.APPLICATION_JSON).content(requestBody)  
            .accept(MediaType.APPLICATION_JSON)) //执行请求  
            .andExpect(content().contentType(MediaType.APPLICATION_JSON)) //验证响应contentType  
            .andExpect(jsonPath("$.id").value(1)); //使用Json path验证JSON 请参考http://goessner.net/articles/JsonPath/  
    String errorBody = "{id:1, name:zhang}";  
    MvcResult result = mockMvc.perform(post("/user")  
            .contentType(MediaType.APPLICATION_JSON).content(errorBody)  
            .accept(MediaType.APPLICATION_JSON)) //执行请求  
            .andExpect(status().isBadRequest()) //400错误请求  
            .andReturn();  
    Assert.assertTrue(HttpMessageNotReadableException.class.isAssignableFrom(result.getResolvedException().getClass()));//错误的请求内容体
```

6.异步测试

```text
//Callable  
    MvcResult result = mockMvc.perform(get("/user/async1?id=1&name=zhang")) //执行请求  
            .andExpect(request().asyncStarted())  
            .andExpect(request().asyncResult(CoreMatchers.instanceOf(User.class))) //默认会等10秒超时  
            .andReturn();  
    mockMvc.perform(asyncDispatch(result))  
            .andExpect(status().isOk())  
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))  
            .andExpect(jsonPath("$.id").value(1));
```

7.全局配置

```text
mockMvc = webAppContextSetup(wac)  
            .defaultRequest(get("/user/1").requestAttr("default", true)) //默认请求 如果其是Mergeable类型的，会自动合并的哦mockMvc.perform中的RequestBuilder  
            .alwaysDo(print())  //默认每次执行请求后都做的动作  
            .alwaysExpect(request().attribute("default", true)) //默认每次执行后进行验证的断言  
            .build();  
    mockMvc.perform(get("/user/1"))  
            .andExpect(model().attributeExists("user"));
```

## 基础概念

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

#### @RequestMapping 

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

### 常见错误

 #####   java.lang.NoSuchMethodError 

java.lang.NoSuchMethodError: org.springframework.util.Assert.notNull 

检查spring家族版本

