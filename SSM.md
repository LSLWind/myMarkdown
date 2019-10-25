### 前后端分离

##### 前端未分离时期开发模式

一开始，网站的数据规模不大的时候，前端负责页面，后端负责数据，前端写几个页面，准确的说是几个模板，交给后端，后端负责填充页面；比较典型的就是jsp；这搞得前端得懂后端，后端也得懂页面；更重要的一点是，如果工程大了，大家都没法单独测试，必须要等整个工程前后端都结束了才能测试，于是大家就都觉得，这不行啊，这开发效率跟不上啊；那就前后端分离吧，

前端使用MVC模式

 ![image](https://images.cnblogs.com/cnblogs_com/rjzheng/1236364/o_front1.png) 

开发方式：

 ![image](https://images.cnblogs.com/cnblogs_com/rjzheng/1236364/o_front2.png) 

##### 半分离时期

那么怎么分离呢?

既然后端是提供数据的，那就专职提供数据吧，后端只提供json这样的数据，前端通过ajax到后端去请求数据，然后js填充页面；大家一看，哎呀，这不错，这样就分离了，大家各干各的，后端想知道接口对不对，curl一下就知道了，前端想知道页面对不对，mock一下；只要前后端约定好数据的格式，各干各的，效率上去了，而且各司其职，多好；这时候，页面是js生成或者填充的，主要逻辑都在js里面，页面可能就一个，这有个极大的好处，就是只需要加载一次页面和js脚本，页面就展现出来了，而且后面点击页面的时候，页面不会像以前一样，浏览器在那不停地转圈，给用户的体验也更好，这就是SPA，单页面应用；

这很好吧，其实问题挺大的，最大的问题在哪呢，最大的问题在用户浏览网页时，实际上大部分情况下都是从搜索引擎中搜索的，像百度Google啥的都是，坑爹的问题是，js动态生成页面，除了Google这种逆天的搜索引擎能解析搜索外，其余的似乎都搞不定，那不行啊，我的网站在百度上搜不到，我屮艸芔茻，那搞毛啊，这就是SPA单页面应用的SEO问题，即搜索引擎优化。为了解决这个问题，还得使用模板，就是后端返回渲染好的页面，那不又历史的倒退了。

 前后端半分离，前端负责开发页面，通过接口（Ajax）获取数据，采用dom操作对页面进行数据绑定，最终是由前端把页面渲染出来。 

 ![image](https://images.cnblogs.com/cnblogs_com/rjzheng/1236364/o_front4.png) 

首先，这种方式的优点是很明显的。前端不会嵌入任何后台代码，前端专注于html、css、js的开发，不依赖于后端。自己还能够模拟json数据来渲染页面。发现bug，也能迅速定位出是谁的问题，不会出现互相推脱的现象。
然而，在这种架构下，还是存在明显的弊端的。最明显的有如下几点：
(1)js存在大量冗余，在业务复杂的情况下，页面的渲染部分的代码，非常复杂。
(2)在json返回的数据比较大的情况下，渲染的十分缓慢，会出现页面卡顿的情况
(3)seo非常不方便，由于搜索引擎的爬虫无法爬下js异步渲染的数据，导致这样的页面，SEO会存在一定的问题。
(4)资源消耗严重，在业务复杂的情况下，一个页面可能要发起多次http请求才能将页面渲染完毕。可能有人不服，觉得pc端建立多次http请求也没啥。那你考虑过移动端么，知道移动端建立一次http请求需要消耗多少资源么？
正是因为如上缺点，真正的前后端分离架构诞生了 

##### 分离时期

 在这一时期，扩展了前端的范围。认为controller层也属于前端的一部分。在这一时期
前端：负责View和Controller层。
后端：只负责Model层，业务处理/数据等。 

 ![image](https://images.cnblogs.com/cnblogs_com/rjzheng/1236364/o_front5.png) 

使用node，js也能做服务端，node拿数据渲染返回即可

当然，天无绝人之路啊，node.js这种神器出现了；js也能做服务端了，这对前端来说，绝对是个好消息，一下子就把所有的问题都解决了，前端可以用node.js做渲染引擎，具体实施就是，前端写好页面，后端提供数据接口，然后前端用node.js从后端请求数据，然后用数据填充渲染页面，最后node.js直接返回渲染后的页面，看吧，前后端又分离了，这时候的前端又叫大前端了。其实这个时候，前端还能做不少事情，node.js甚至可以搭个数据库或者缓存啥的，做一些权限管理过滤啥的；

那就有人说了，前端既然这么牛逼了，还要后端搞毛啊，是啊，如果单纯的只是写个网站，搞个页面，还真没后端什么事；但是现在的网站已经复杂到没办法控制了，你看到一个页面，实际上后面还有一堆东西，比如风险控制，实时数据分析，推荐系统，另外，海量的数据还需要做分库分表，海量的数据如何做快速查询，提升用户体验，采用什么样的缓存技术等等都是要考虑的，为保证高可靠性，还得引入各种冗余备份，熔断，限流，削峰等等技术；还有像现在要么618，要么双十一这种瞬时流量大的，平时流量小的业务，机器资源怎么实现动态分配扩容等等，这就涉及到云计算了；但凡种种，现在的后端已经不是当年搞个随便搞个mysql存个几万几十万条数据，然后随便搞搞增删改查就能搞定的东西了，背后涉及了太多的技术了；

所以，到了这个时候，前后端不是为了分离而分离，而是不得不分离了……

#### SSM分层结构

表现层(jsp/前端页面)&rightarrow;MVC控制器层(多个控制器)&rightarrow;业务逻辑层(使用Spring实现业务逻辑)&rightarrow;DAO(MyBatis,包含多个DAO，实现CRUD原子操作)&rightarrow;POJO(Plain Old java Object 普通的java对象)，一些简单的Bean&rightarrow;数据库(数据库表,索引...)

springMVC表现层→Service业务层→mybatis持久层(DAO)

### Spring MVC

#### Spring Web MVC framework架构

围绕DispatcherServlet调度请求request，将request分配给句柄handlers处理，默认handler基于注解**@Controller**与**@RequestMapping**

![mvc](https://docs.spring.io/spring/docs/4.2.9.RELEASE/spring-framework-reference/htmlsingle/images/mvc.png)

DispatcherServlet继承于HttpServlet,在web.xml中声明,是前端控制器，(将所有request分发给对应的HandlerMapping将URL映射之后传给HandlerAdapter)，再传给Controller处理完之后返回一个model给DispatcherServlet,Dispatcher将model传给ViewResolver解析成View(视图)再返回Dispatcher返回响应

![img](https://img-my.csdn.net/uploads/201211/16/1353059506_5137.jpg)

控制器相当于根据request返回ViewModel,然后View根据Model返回对应的响应View

- 在接收到HTTP请求后，`DispatcherServlet`会查询`HandlerMapping`以调用相应的`Controller`。
- `Controller`接受请求并根据使用的`GET`或`POST`方法调用相应的服务方法。 服务方法将基于定义的业务逻辑设置模型数据，并将视图名称返回给`DispatcherServlet`。
- `DispatcherServlet`将从`ViewResolver`获取请求的定义视图。
- 当视图完成，`DispatcherServlet`将模型数据传递到最终的视图，并在浏览器上呈现。

所有上述组件，即: `HandlerMapping`，`Controller`和`ViewResolver`是`WebApplicationContext`的一部分，它是普通`ApplicationContext`的扩展

![img](http://www.yiibai.com/uploads/images/201701/18/451110124_66519.png)

#### 配置使用

##### maven引入SpringMVC

```xml
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>4.3.1.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.3.1.RELEASE</version>
    </dependency>
```

##### web.xml中配置DispatcherServlet与映射

DispatcherServlet相当于Servlet的总控制器，需要在web.xml中进行配置并且设置映射

```xml
<web-app>
    <servlet>
        <servlet-name>example</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup><!--启动时立即加载Servlet-->
    </servlet>

    <servlet-mapping>
        <servlet-name>example</servlet-name>
        <url-pattern>/example/*</url-pattern>
    </servlet-mapping>

</web-app>
```

映射servlet-mapping配置由DispatcherServlet调度，上例即表明所有/example/*的URL请求都被映射给example,该example表示具体的请求映射，默认在WEB-INF下以example-servlet.xml（即`[servlet-name]-servlet.xml`)形式呈现，由DispatcherServlet调度。SpringMVC的配置文件使用init-param指定加载路径，例

```xml
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <!-- contextConfigLocation 是参数名称，该参数的值包含 Spring MVC 的配置文件路径 -->
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/springmvc-config.xml</param-value>
        </init-param>
        <!-- 在 Web 应用启动时立即加载 Servlet -->
        <load-on-startup>1</load-on-startup>
    </servlet>
```

`[servlet-name]-servlet.xml`文件将用于创建定义的`bean`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd">

    <!-- 开启注解 -->
    <mvc:annotation-driven />
    <!-- 配置自动扫描的包，完成 Bean 的创建和自动依赖注入的功能 -->
    <context:component-scan base-package="com.controller" />
    <!-- 默认静态资源处理 -->
    <mvc:default-servlet-handler/>
    <!-- 配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```

`<context：component-scan base-package="xxx">`标签将用于激活`Spring MVC`注释扫描功能，允许使用`@Controller`和`@RequestMapping`

视图解析器 InternalResourceViewResolver 用来解析视图，将 View 呈现给用户。视图解析器中配置的 `prefix`表示视图的前缀， `suffix`表示视图的后缀。即将所有视图处理委托给views下的对应的.jsp显示

**即在web.xml下配置DispatcherServlet，声明某些url由DispatcherServlet进行调度处理（如上述配置，所有wind/下的url请求都由DispatcherServlet进行调度），对应的url路径的配置默认在WEB-INF下的wind-servlet.xml下，在该xml中配置相关信息，主要是使用<context:component-scan base-package="com.controller" />指定注解扫描包并配置视图解析器，然后在包中使用注解@Controller声明控制器，使用@RequestMapping声明对应url处理方法，返回一个视图解析器中的视图名称，DispatcherServlet接收请求，交给控制器，控制器中使用@RequestMapping声明的方法处理请求，参数为model，返回一个视图名称，DispatcherServlet返回View**

#### 初始示例

##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>lsl</groupId>
  <artifactId>winds</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>winds Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>

    <!--springMVC-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>4.3.1.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.3.1.RELEASE</version>
    </dependency>

  </dependencies>

  <build>
    <finalName>winds</finalName>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-war-plugin</artifactId>
          <version>3.2.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
    </plugins>
    </pluginManagement>
  </build>
</project>

```

##### web.xml

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <servlet>
    <servlet-name>wind</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup><!--启动时立即加载Servlet-->
  </servlet>

  <servlet-mapping>
    <servlet-name>wind</servlet-name>
    <url-pattern>/wind/*</url-pattern>
  </servlet-mapping>
</web-app>

```

##### wind-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd">

    <!-- 开启注解 -->
    <mvc:annotation-driven />
    <!-- 配置自动扫描的包，完成 Bean 的创建和自动依赖注入的功能 -->
    <context:component-scan base-package="wind" />
    <!-- 默认静态资源处理 -->
    <mvc:default-servlet-handler/>
    <!-- 配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

</beans>
```

视图解析器规定视图前缀为/WEB-INF/views，后缀为.jsp，模式映射返回一个String，必须为视图

##### 控制器HelloWorld.java

```java
package wind;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

//控制器
@Controller
public class HelloWorld {
    //控制器当中的请求映射，传递model
    @RequestMapping(value = "/helloWorld",method = RequestMethod.GET)
    public String Register(ModelMap modelMap){
        //model进行数据处理
        modelMap.addAttribute("messageKey","messageValue");
        //返回视图名称，返回给DispatcherServlet
        return "helloWorld";//返回一个视图名称
    }
}
```

##### 视图helloWorld.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
响应视图------------helloWorld.jsp<br>
<h2>${messageKey}</h2>
</body>
</html>

```

##### 效果

访问url:127.0.0.1:8080/wind/helloWorld

### 详细使用

View为服务器上的某个文件容器，可以为JSP,FTL等动态页面文件，甚至是媒体文件等等，单单是一个文件。Model的作用是存储动态页面属性，动态页面文件即View可以在Model中获取动态数据，这样就实现了View和Model分离的目的。

一、View核心方法：

　　1、getContentType()获取当前view的ContentType()，同http请求中的ContenType()是一样的作用。

　　2、render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response)。

　　根据参数就可以知道，这个是为视图绑定Request，Response和Model的方法。

　　4、view的子接口SmartView，唯一一个方法isRedirectView，用于标记是否是重定向View。

 二、model相对简单，就是一些属性的键值对而已。

　　Model addAttribute(String attributeName, Object attributeValue);　　

　　Model addAttribute(Object attributeValue);

　　Model addAllAttributes(Collection<?> attributeValues);

　　Model addAllAttributes(Map<String, ?> attributes);

　　Model mergeAttributes(Map<String, ?> attributes);

　　boolean containsAttribute(String attributeName);

　　Map<String, Object> asMap();

三、ModelAndView，顾名思义，就是整合了Model和View，常用于动态页面也是，一个后台网页就包含这两部分，前台就是基本的html代码。

　　内部包含一个Object类型的view，一个ModelMap类型的model，还有一个标记是否被clear()方法清除的cleared变量。

　　构造方法：ModelAndView()

　　ModelAndView(String viewName)

　　ModelAndView(View view)

　　ModelAndView(String viewName, Map<String, ?> model)

　　ModelAndView(View view, Map<String, ?> model)

　　ModelAndView(String viewName, String modelName, Object modelObject)

　　ModelAndView(View view, String modelName, Object modelObject)

　　(s&g)ViewName

　　(s&g)View

　　boolean hasView

　　isReference:如果view instanceof String则为true

　　Map<String, Object> getModelInternal()

　　ModelMap getModelMap()

　　Map<String, Object> getModel() 

　　ModelAndView addObject(String attributeName, Object attributeValue)

　　ModelAndView addObject(Object attributeValue)

　　ModelAndView addAllObjects(Map<String, ?> modelMap)

　　void clear()：清除其中的view和model

　　由此可以看出，构造View可以直接使用字符串，相当于路径引用View

 四、最后，这几个实际中到底是在哪使用，如何使用的呢？

　　Spring架构中，他是使用在@Controller层中的@RequestMapping注解方法中的，该注解方法有以下几种：

　　返回值可以有：ModelAndView, Model, ModelMap, Map,View, String, void

　　参数可以任意多，参数可以为POJO对象，可以为PO，可以为Array，List，String等等，只要能和Request中参数名一致，Spring就会自动绑定到特定的数据类型中，非常方便。其中有三个特殊类型的参数：HttpServletRequest request, HttpServletResponse response, Model model，其中第一个为当前网页发过来的请求，第二个是要返回的响应，第三个则是当前网页发过来的model参数集以及要设置给返回视图的参数集Model。

　　注：参数中的Model也可以变为同样的Map，或者ModelMap，三者等价。

　　来看看不同类型的返回值代表什么吧。

　　1、ModelAndView

```
@RequestMapping("/show1") 
public ModelAndView show1(HttpServletRequest request, 
           HttpServletResponse response) throws Exception { 
       ModelAndView mav = new ModelAndView("/demo2/show"); 
       mav.addObject("account", "account -1"); 
       return mav; 
   } 
```

　　这种是直接把View和Model封装好返回给Spring，Spring再交给动态网页架构去返回静态html给客户端。

　　通过ModelAndView构造方法可以指定返回的页面名称，也可以通过setViewName()方法跳转到指定的页面 ,使用addObject()设置需要返回的值，addObject()有几个不同参数的方法，可以默认和指定返回对象的名字。
　　2、Model=ModelMap=Map，是一样的

```
@RequestMapping("/demo2/show") 
    public Map<String, String> getMap() { 
        Map<String, String> map = new HashMap<String, String>(); 
        map.put("key1", "value-1"); 
        map.put("key2", "value-2"); 
        return map; 
    } 
```

　　相当于把Map返回给当前网页。在jsp页面中可直通过${key1}获得到值, map.put()相当于request.setAttribute方法。key值包括 - . 时会有问题.

　　3、View 可以返回pdf excel等各种View，此时该View的Model可以在方法传递的参数中设置。只需在方法中加上参数Model model，对该model设置属性，都会反馈到该View中。

　　4、String，其实也是View，该View是urlView，指定返回的视图页面名称，结合设置的返回地址路径加上页面名称后缀即可访问到。

```
@RequestMapping(value = "/something", method = RequestMethod.GET) 
@ResponseBody 
public String helloWorld()  { 
return "Hello World"; 
} 
```

　　注意：如果方法声明了注解@ResponseBody ，则会直接将返回值输出到页面。上面的结果会将文本"Hello World "直接写到http响应流。

　　若没有注解@ResponseBody，则对应的逻辑视图名为“center”，URL= prefix前缀+视图名称 +suffix后缀组成。如果以绝对路径书写，则在applicationContext路径为根的目录下寻找View。

```
@RequestMapping("/welcome") 
public String welcomeHandler() { 
  return "center"; 
} 
```

　　最后一个特殊的为void，响应的视图页面对应为访问地址。

```
@RequestMapping("/welcome") 
public void welcomeHandler() {} 
```

　　此例对应的逻辑视图名为"welcome"。

　　PS：1.使用 String 作为请求处理方法的返回值类型是比较通用的方法，这样返回的逻辑视图名不会和请求 URL 绑定，具有很大的灵活性，而模型数据又可以通过 ModelMap 控制。
　　2.使用void,map,Model 时，返回对应的逻辑视图名称真实url为：prefix前缀+视图名称 +suffix后缀组成。
　　3.使用String,ModelAndView返回视图名称可以不受请求的url绑定，ModelAndView可以设置返回的视图名称。

　　补充：

　　Model model,HttpServletRequest request, ModelMap map声明变量，都可以赋属性和属性值，具体先引用谁呢？

　　　　request.getSession().setAttribute("test", "haiwei2Session");
　　　　request.setAttribute("test", "haiwei1request"); 
　　　　map.addAttribute("test", "haiweiModelMap");
　　　　model.addAttribute("test", "haiweiModel");

　　通过${test}取值，优先取Model和ModelMap的，Model和ModelMap是同一个东西，谁最后赋值的就取谁的，然后是request，最后是从session中获取





#### 基本注解

##### 使用@Controller声明控制器

注解@Controller表示这是一个控制器

##### 使用@RequestMapping映射url

请求映射，@RequestMapping 注解用来处理请求地址映射，指示 Spring 用哪个类或方法来处理请求动作。可用于类或方法上。

当 @RequestMapping 用于类上时，表示类中的所有响应方法都被映射到 value 属性所指示的路径下

```
@Controller
@RequestMapping(value="/user")//即映射wind/user
```

当 @RequestMapping 用于方法上时，指示该方法用于处理哪个url

##### 使用@RequestParam绑定参数

@RequestParam声明字段，将url传递过来的信息绑定到参数上

```java
@RequestMapping(value="/login")
public ModelAndView login(
    @RequestParam("username") String username, @RequestParam("password") String password) { 

    return ...
}
```

如果url为http://localhost:8080/login?username=tom&password=123456则会将对应信息绑定到参数上

