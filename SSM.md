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

