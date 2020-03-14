### Servlet基础概念
Java Servlet 是运行在 Web 服务器或应用服务器上的程序

Servlet 可以使用 javax.servlet 和 javax.servlet.http 包创建，它是 Java 企业版的标准组成部分，Java 企业版是支持大型开发项目的 Java 类库的扩展版本，这些类实现 Java Servlet 和 JSP 规范

#### Servlet 任务
读取客户端（浏览器）发送的显式的数据。这包括网页上的 HTML 表单，或者也可以是来自 applet 或自定义的 HTTP 客户端程序的表单。
读取客户端（浏览器）发送的隐式的 HTTP 请求数据。这包括 cookies、媒体类型和浏览器能理解的压缩格式等等。
处理数据并生成结果。这个过程可能需要访问数据库，执行 RMI 或 CORBA 调用，调用 Web 服务，或者直接计算得出对应的响应。
发送显式的数据（即文档）到客户端（浏览器）。该文档的格式可以是多种多样的，包括文本文件（HTML 或 XML）、二进制文件（GIF 图像）、Excel 等。
发送隐式的 HTTP 响应到客户端（浏览器）。这包括告诉浏览器或其他客户端被返回的文档类型（例如 HTML），设置 cookies 和缓存参数，以及其他类似的任务。

#### Tomcat
Apache Tomcat 是一款实现 Java Servlet 和 JavaServer Pages 技术的开源软件，可以作为测试 Servlet 的独立服务器，而且可以集成到 Apache Web 服务器。

下载完Tomcat之后，进入目录

\bin\startup.bat   执行脚本启动服务器
\bin\shutdown.bat     关闭服务器
导入jar包
通常需要手动在项目中导入apache-tomcat\lib\servlet-api.jar以使用javax.servlet.*

如果使用Maven等工具创建项目，需要在pom.xml(配置文件)中手动添加依赖

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
</dependency>
```

#### Servlet 生命周期
Servlet 生命周期可被定义为从创建直到毁灭的整个过程。

Servlet 通过调用 init () 方法进行初始化。
Servlet 调用 service() 方法来处理客户端的请求。
Servlet 通过调用 destroy() 方法终止（结束）。
最后，由 JVM 的垃圾回收器进行垃圾回收。
init() 方法被设计成只调用一次。它在第一次创建 Servlet 时被调用，在后续每次用户请求时不再调用。

service() 方法是执行实际任务的主要方法。Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。

service() 方法由容器调用，service 方法在适当的时候调用 doGet、doPost、doPut、doDelete 等方法。所以不用对 service() 方法做任何动作，只需要根据来自客户端的请求类型来重写 doGet() 或 doPost() 即可。

destroy() 方法只会被调用一次，在 Servlet 生命周期结束时被调用。destroy() 方法让的Servlet 关闭数据库连接、停止后台线程、把 Cookie 列表或点击计数器写入到磁盘，并执行其他类似的清理活动。在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收。

#### Servlet部署/IDEA创建Web项目
默认情况下，Servlet 应用程序位于路径 <Tomcat-installation-directory\>/webapps/ROOT 下，且类文件放在 <Tomcat-installation-directory\>/webapps/ROOT/WEB-INF/classes 中。

并要对<Tomcat-installation-directory\>/webapps/ROOT/WEB-INF/ 的 web.xml 文件进行相应的配置。这种修改很麻烦，通常创建Web项目更方便。

使用IDEA+maven创建Web项目，IDEA要使用企业版，否则无法开发web项目

### Servlet编写

#### Servlet与Get/Post
前端页面与服务器交互响应通过一些定义好的方法进行说明，如get/post等。

GET 方法向页面请求发送已编码的用户信息。页面和已编码的信息中间用 ? 字符分隔，如下所示：

http://www.test.com/hello?key1=value1&key2=value2
GET 方法是默认的从浏览器向 Web 服务器传递信息的方法，它会产生一个很长的字符串，出现在浏览器的地址栏中。

POST方法也是向服务器传输数据，但是是作为单独的消息发送，与Get功能一样

servlet通过doGet()方法处理get请求，doPost()方法处理post()请求

可以调用 request.getParameter() 方法来获取表单参数的值。


可以看到,get方法是默认的request的方法，通过get发送数据，可以直接通过url?key=value&..这样的形式，但前提是你必须知道相应的key到底是什么，也可以前端自己设置，进行表单提交，Servlet进行处理后返回对应的response。

#### 读取表单数据
web的核心是访问资源与数据通信，访问资源包括浏览与下载，浏览html页面，下载相应资源。数据通信就是与服务器进行交互，给予服务器数据返回响应，后台如何读取数据呢,Servlet提供三个最基本方法。

在这之前，前端的表单的展现形式是标签<form\>，标签包含多个字标签,<name\>就是表单选项的name。

Servlet读取表单数据有三个方法

```java
getParameter()：如果知道具体的name，传入即返回前端对应的value
getParameterNames()：返回一个枚举类型的迭代器Enumeration，通过迭代获取表单的所有name。
Enumeration paraNames=request.getParameterNames();//所有表单数据参数
while(paraNames.hasMoreElements()){
    String paraName=(String)paraNames.nextElement();
}
getParameterValues()：如果参数出现一次以上，则调用该方法，并返回多个值，例如复选框。
String[] paramValues=request.getParameterValues(paraName);//由name获取数据值
即前端表单与后台相互连通对应。
```

#### Http获取请求头部信息
请求request的头部包含许多重要信息

下面的方法可用在 Servlet 程序中读取 HTTP 头。这些方法通过 HttpServletRequest 对象可用。

* Cookie[] getCookies()：返回一个数组，包含客户端发送该请求的所有的 Cookie 对象。
* Enumeration getAttributeNames()：返回一个枚举，包含提供给该请求可用的属性名称。
* Enumeration getHeaderNames()：返回一个枚举，包含在该请求中包含的所有的头名。
* Enumeration getParameterNames()：返回一个 String 对象的枚举，包含在该请求中包含的参数的名称。
  5	HttpSession getSession()
  返回与该请求关联的当前 session 会话，或者如果请求没有 session 会话，则创建一个。
  6	HttpSession getSession(boolean create)
  返回与该请求关联的当前 HttpSession，或者如果没有当前会话，且创建是真的，则返回一个新的 session 会话。
  7	Locale getLocale()
  基于 Accept-Language 头，返回客户端接受内容的首选的区域设置。
  8	Object getAttribute(String name)
  以对象形式返回已命名属性的值，如果没有给定名称的属性存在，则返回 null。
  9	ServletInputStream getInputStream()
  使用 ServletInputStream，以二进制数据形式检索请求的主体。
  10	String getAuthType()
  返回用于保护 Servlet 的身份验证方案的名称，例如，"BASIC" 或 "SSL"，如果JSP没有受到保护则返回 null。
  11	String getCharacterEncoding()
  返回请求主体中使用的字符编码的名称。
  12	String getContentType()
  返回请求主体的 MIME 类型，如果不知道类型则返回 null。
  13	String getContextPath()
  返回指示请求上下文的请求 URI 部分。
  14	String getHeader(String name)
  以字符串形式返回指定的请求头的值。
  15	String getMethod()
  返回请求的 HTTP 方法的名称，例如，GET、POST 或 PUT。
  16	String getParameter(String name)
  以字符串形式返回请求参数的值，或者如果参数不存在则返回 null。
  17	String getPathInfo()
  当请求发出时，返回与客户端发送的 URL 相关的任何额外的路径信息。
  18	String getProtocol()
  返回请求协议的名称和版本。
  19	String getQueryString()
  返回包含在路径后的请求 URL 中的查询字符串。
  20	String getRemoteAddr()
  返回发送请求的客户端的互联网协议（IP）地址。
  21	String getRemoteHost()
  返回发送请求的客户端的完全限定名称。
  22	String getRemoteUser()
  如果用户已通过身份验证，则返回发出请求的登录用户，或者如果用户未通过身份验证，则返回 null。
  23	String getRequestURI()
  从协议名称直到 HTTP 请求的第一行的查询字符串中，返回该请求的 URL 的一部分。
  24	String getRequestedSessionId()
  返回由客户端指定的 session 会话 ID。
  25	String getServletPath()
  返回调用 JSP 的请求的 URL 的一部分。
  26	String[] getParameterValues(String name)
  返回一个字符串对象的数组，包含所有给定的请求参数的值，如果参数不存在则返回 null。
  27	boolean isSecure()
  返回一个布尔值，指示请求是否使用安全通道，如 HTTPS。
  28	int getContentLength()
  以字节为单位返回请求主体的长度，并提供输入流，或者如果长度未知则返回 -1。
  29	int getIntHeader(String name)
  返回指定的请求头的值为一个 int 值。
  30	int getServerPort()
  返回接收到这个请求的端口号。
  31	int getParameterMap()
  将参数封装成 Map 类型。

```java
Enumeration headerNames = request.getHeaderNames();//获取http请求头部信息name，返回一个枚举
while(headerNames.hasMoreElements()) {
    String paramName = (String)headerNames.nextElement();//http头部所定义的参数name
    String paramValue = request.getHeader(paramName);//由name获取头部信息value,即请求的当前状态
}
```

#### Http设置响应头部信息
Servlet也可以对返回的响应也可以进行设置

响应通常包括一个状态行、一些响应报头、一个空行和文档。
Servlet提供一系列setter()方法对其响应进行相应的设置。

序号	方法 & 描述
1	String encodeRedirectURL(String url)
为 sendRedirect 方法中使用的指定的 URL 进行编码，或者如果编码不是必需的，则返回 URL 未改变。
2	String encodeURL(String url)
对包含 session 会话 ID 的指定 URL 进行编码，或者如果编码不是必需的，则返回 URL 未改变。
3	boolean containsHeader(String name)
返回一个布尔值，指示是否已经设置已命名的响应报头。
4	boolean isCommitted()
返回一个布尔值，指示响应是否已经提交。
5	void addCookie(Cookie cookie)
把指定的 cookie 添加到响应。
6	void addDateHeader(String name, long date)
添加一个带有给定的名称和日期值的响应报头。
7	void addHeader(String name, String value)
添加一个带有给定的名称和值的响应报头。
8	void addIntHeader(String name, int value)
添加一个带有给定的名称和整数值的响应报头。
9	void flushBuffer()
强制任何在缓冲区中的内容被写入到客户端。
10	void reset()
清除缓冲区中存在的任何数据，包括状态码和头。
11	void resetBuffer()
清除响应中基础缓冲区的内容，不清除状态码和头。
12	void sendError(int sc)
使用指定的状态码发送错误响应到客户端，并清除缓冲区。
13	void sendError(int sc, String msg)
使用指定的状态发送错误响应到客户端。
14	void sendRedirect(String location)
使用指定的重定向位置 URL 发送临时重定向响应到客户端。
15	void setBufferSize(int size)
为响应主体设置首选的缓冲区大小。
16	void setCharacterEncoding(String charset)
设置被发送到客户端的响应的字符编码（MIME 字符集）例如，UTF-8。
17	void setContentLength(int len)
设置在 HTTP Servlet 响应中的内容主体的长度，该方法设置 HTTP Content-Length 头。
18	void setContentType(String type)
如果响应还未被提交，设置被发送到客户端的响应的内容类型。
19	void setDateHeader(String name, long date)
设置一个带有给定的名称和日期值的响应报头。
20	void setHeader(String name, String value)
设置一个带有给定的名称和值的响应报头。
21	void setIntHeader(String name, int value)
设置一个带有给定的名称和整数值的响应报头。
22	void setLocale(Locale loc)
如果响应还未被提交，设置响应的区域。
23	void setStatus(int sc)
为该响应设置状态码。
response.setIntHeader("Refresh",1);//设置响应头部信息，5秒自动刷新
response.setContentType("text/html;charset=UTF-8");//设置响应头部
Http设置头部状态码

一般只需要响应头部设置状态码，返回信息即可 

这些方法通过 HttpServletResponse 对象可用。

序号	方法 & 描述
1	public void setStatus ( int statusCode )
该方法设置一个任意的状态码。setStatus 方法接受一个 int（状态码）作为参数。如果您的响应包含了一个特殊的状态码和文档，请确保在使用 PrintWriter 实际返回任何内容之前调用 setStatus。
2	public void sendRedirect(String url)
该方法生成一个 302 响应，连同一个带有新文档 URL 的 Location 头。
3	public void sendError(int code, String message)
该方法发送一个状态码（通常为 404），连同一个在 HTML 文档内部自动格式化并发送到客户端的短消息。
response.sendError(777, "响应状态码，可自行设置");
#### 过滤器Filter
过滤器用于在请求与响应之前进行拦截并处理，类似于面向切面的概念，请求不能一味的直接交给Servlet,先要进行拦截，拦截时对一系列信息进行检查或者包装等等操作，最终再返回，通过继承Filter接口实现。过滤器当然可以有多个，因此就形成了过滤链，用类 FilterChain表示，其只有一个doFilter()方法，表示将当前请求或响应交给下一个过滤器。

过滤器是一个实现了 javax.servlet.Filter 接口的 Java 类。javax.servlet.Filter 接口定义了三个方法：

序号	方法 & 描述
1	public void doFilter (ServletRequest, ServletResponse, FilterChain)
该方法完成实际的过滤操作，当客户端请求方法与过滤器设置匹配的URL时，Servlet容器将先调用过滤器的doFilter方法。FilterChain用户访问后续过滤器。
2	public void init(FilterConfig filterConfig)
web 应用程序启动时，web 服务器将创建Filter 的实例对象，并调用其init方法，读取web.xml配置，完成对象的初始化功能，从而为后续的用户请求作好拦截的准备工作（filter对象只会创建一次，init方法也只会执行一次）。开发人员通过init方法的参数，可获得代表当前filter配置信息的FilterConfig对象。
3	public void destroy()
Servlet容器在销毁过滤器实例前调用该方法，在该方法中释放Servlet过滤器占用的资源。
Filter的生命周期存在于整个服务器的运行周期，当服务器启动时，创建Filter对象并调用init(),filter对象只会创建一次，init方法也只会执行一次,之后会根据web.xml或注解配置的url启动对应的过滤器，每有一次访问即调用一次doFilter()方法，最后服务器关闭时调用destory()。

```java
package study;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

//Filter接口实现过滤，用于在请求与响应之前进行拦截并处理然后再交给Servlet
//Filter接口主要要实现三个方法

/**

 * public void init(FilterConfig filterConfig)

 * web 应用程序启动时，web 服务器将创建Filter 的实例对象，并调用其init方法，读取web.xml配置，完成对象的初始化功能，从而为后续的用户请求作好拦截的准备工作（filter对象只会创建一次，init方法也只会执行一次）。开发人员通过init方法的参数，可获得代表当前filter配置信息的FilterConfig对象。

 * public void doFilter (ServletRequest, ServletResponse, FilterChain)

 * 该方法完成实际的过滤操作，当客户端请求方法与过滤器设置匹配的URL时，Servlet容器将先调用过滤器的doFilter方法。FilterChain用户访问后续过滤器。

 * public void destroy()

 * Servlet容器在销毁过滤器实例前调用该方法，在该方法中释放Servlet过滤器占用的资源。
   */
   //使用web.xml与注解类似,<filter>标签初始化一个FilterConfig对象，传给init
   @WebFilter("/*")
   public class LogFilter implements Filter {
   @Override
   public void init(FilterConfig config){
       System.out.println("过滤初始化-LogFilter-init");
   }
   @Override
   public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)throws ServletException,IOException{
       System.out.println("开始过滤-LogFilter-doFilter");
       chain.doFilter(request,response);//将请求与响应传入过滤链
   }

   @Override
   public void destroy() {
       System.out.println("过滤销毁-LogFilter-destroy");
   }
   }
```

同Servlet一样，Filter也要在web.xml下进行相应配置或使用注解。

```xml
<filter>
  <filter-name>LogFilter</filter-name>
  <filter-class>com.runoob.test.LogFilter</filter-class>
  <init-param>
    <param-name>filterName</param-name>
    <param-value>filterValue</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>LogFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
</filter>
```

其中<init-param\>标签在init()中有用,init()传入的FilterConfig即代表过滤器初始实例,可以调用

getInitParameter("filterName");
方法获取配置的对应的filterValue

通过使用注解@WebFilter("/XXX")效果相同

当然可以使用多个过滤器，调用过滤链方法即可

#### 异常处理
如果请求抛出异常或包含错误状态码(404等),想要返回对应的错误页面，也应该对应一个Servlet,即发生错误时跳转到对应的Servlet。同时在web.xml中设置标签<error-page\>

<error-page\>包含子标签<error-code\>对应的错误状态码,<exception-type\>抛出异常类型,<location\>跳转到对应的Servlet

```xml
<error-page>
    <error-code>404</error-code>
    <location>/ErrorHandler</location>
</error-page>
<error-page>
    <exception-type>java.io.IOException</exception-type >
    <location>/ErrorHandler</location>
</error-page>
```

如果只有<location\>则默认所有错误都进行跳转

```java
@WebServlet("/ErrorHandler")
public class ErrorHandler extends HttpServlet {
    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response)throws ServletException, IOException {
        Throwable throwable=(Throwable)request.getAttribute("javax.servlet.error.exception");//获取异常属性
        Integer statusCode=(Integer)request.getAttribute("javax.servlet.error.status_code");//获取状态码
        String servletName=(String)request.getAttribute("javax.servlet.error.servlet_name");//获取servlet
        servletName=servletName==null?"未知Servlet":servletName;

        String requestUri=(String)request.getAttribute("javax.servlet.error.request_uri");//获取请求uri
        requestUri=requestUri==null?"未知URI":requestUri;
     
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        String title = " Error/Exception 信息处理";
     
        String docType = "<!DOCTYPE html>\n";
        out.println(docType +
                "<html>\n" +
                "<head><title>" + title + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n");
        out.println("<h1>抛出错误时的显示页面</h1>");
        if (throwable == null && statusCode == null){
            out.println("<h2>错误信息丢失</h2>");
            out.println("请返回 <a href=\"" +
                    response.encodeURL("http://localhost:8080/") +
                    "\">主页</a>。");
        }else if (statusCode != null) {
            out.println("错误代码 : " + statusCode);//如果错误状态码则显示状态码
        }else{//如果抛出异常则显示异常信息
            out.println("<h2>错误信息</h2>");
            out.println("Servlet Name : " + servletName +
                    "</br></br>");
            out.println("异常类型 : " +
                    throwable.getClass( ).getName( ) +
                    "</br></br>");
            out.println("请求 URI: " + requestUri +
                    "<br><br>");
            out.println("异常信息: " +
                    throwable.getMessage( ));
        }
        out.println("</body></html>");
    }
    // 处理 POST 方法请求的方法
    public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }

}
```

其中通过getAttribute()获取request的相关属性信息，返回一个Object,，需要向下转型，如

Throwable throwable=(Throwable)request.getAttribute("javax.servlet.error.exception");//获取异常属性

Integer statusCode=(Integer)request.getAttribute("javax.servlet.error.status_code");//获取状态码

String servletName=(String)request.getAttribute("javax.servlet.error.servlet_name");//获取servlet

#### Cookie处理
如果第一次访问服务器，服务器读取表单数据，然后返回的响应中设置相应cookie,下一次发送服务器的cookie中就有相应的信息，主要目的就是方便用户。

第一次读取:Browser--->Server(读取Cookie,没有信息，response设置Cookie)--->response返回给Browse，Browse设置Cookie

已经有Cookie:Browser---->Server(读取Cookie获取信息)<->DataBase                                                                                                             

Servlet中的Cookie处理主要使用类Cookie,代表一个Cookie信息

Servlet Cookie 处理需要对中文进行编码与解码，方法如下：

String   str   =   java.net.URLEncoder.encode("中文"，"UTF-8");            //编码
String   str   =   java.net.URLDecoder.decode("编码后的字符串","UTF-8");   // 解码
Cookie 通常设置在 HTTP 头信息中,，Set-Cookie 头包含了一个名称值对、一个 GMT 日期、一个路径和一个域。名称和值会被 URL 编码。Cookie以键值对的形式展现。

以下是在 Servlet 中操作 Cookie 时可使用的有用的方法列表。

序号	方法 & 描述
1	public void setDomain(String pattern)
该方法设置 cookie 适用的域，例如 runoob.com。
2	public String getDomain()
该方法获取 cookie 适用的域，例如 runoob.com。
3	public void setMaxAge(int expiry)
该方法设置 cookie 过期的时间（以秒为单位）。如果不这样设置，cookie 只会在当前 session 会话中持续有效。
4	public int getMaxAge()
该方法返回 cookie 的最大生存周期（以秒为单位），默认情况下，-1 表示 cookie 将持续下去，直到浏览器关闭。
5	public String getName()
该方法返回 cookie 的名称。名称在创建后不能改变。
6	public void setValue(String newValue)
该方法设置与 cookie 关联的值。
7	public String getValue()
该方法获取与 cookie 关联的值。
8	public void setPath(String uri)
该方法设置 cookie 适用的路径。如果您不指定路径，与当前页面相同目录下的（包括子目录下的）所有 URL 都会返回 cookie。
9	public String getPath()
该方法获取 cookie 适用的路径。
10	public void setSecure(boolean flag)
该方法设置布尔值，表示 cookie 是否应该只在加密的（即 SSL）连接上发送。
11	public void setComment(String purpose)
设置cookie的注释。该注释在浏览器向用户呈现 cookie 时非常有用。
12	public String getComment()
获取 cookie 的注释，如果 cookie 没有注释则返回 null。
通过 Servlet 设置 Cookie 包括三个步骤：

(1) 创建一个 Cookie 对象：调用带有 cookie 名称和 cookie 值的 Cookie 构造函数，cookie 名称和 cookie 值都是字符串。

Cookie cookie = new Cookie("key","value");
(2) 设置最大生存周期：使用 setMaxAge 方法来指定 cookie 能够保持有效的时间（以秒为单位）。下面将设置一个最长有效期为 24 小时的 cookie。

cookie.setMaxAge(60*60*24); 
(3) 发送 Cookie 到 HTTP 响应头：使用 response.addCookie 来添加 HTTP 响应头中的 Cookie，如下所示：

response.addCookie(cookie);
读取Cookie也很简单:

Cookie[] cookies=request.getCookies();//获取cookies
删除Cookie:

调用Cookie的setMaxAge()方法将最大生存周期设置为0就相当于删除Cookie

```java
@WebServlet("/SetAndReadCookie")
public class SetAndReadCookie extends HttpServlet {
    private static final long serialVersionUID=1L;

    public SetAndReadCookie(){
        super();
    }
     
    @Override
    public void doGet(HttpServletRequest request, HttpServletResponse response)throws ServletException,IOException{
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");//都设置字符集
     
        Cookie[] cookies=request.getCookies();//获取cookies
        PrintWriter out=response.getWriter();
     
        out.println("<!DOCTYPE html>\n" +
                "<html>\n" +
                "<head><title>" + "读取/设置Cookie" + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n" );
        //如果访问时没有cookie则设置cookie
        if(cookies.length==3){
            Cookie id=new Cookie("id",request.getParameter("id"));//由表单获取数据并设置cookie
            Cookie name=new Cookie("name", URLEncoder.encode(request.getParameter("name"),"UTF-8"));
            id.setMaxAge(24*60*60);//设置服务器的cookie的存在时间
            name.setMaxAge(24*60*60);
            response.addCookie(id);//在响应头部中增加cookie即可，服务器会保存，下一次自动发给服务器
            response.addCookie(name);
     
            out.print("第一次访问网站或者cookie过期，已重置cookie");
        }else{
            out.println("<h2>已获取到Cookie,Cookie 名称和值</h2>");
            for(Cookie cookie:cookies){
                out.print("名称：" + cookie.getName( ) + "，");//读取cookie
                out.print("值：" +  URLDecoder.decode(cookie.getValue(), "utf-8") +" <br/>");
            }
        }
        out.println("</body></html>");
    }
     
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }

}
```

#### Session会话
session会话是一个抽象概念,session存储于服务器内存中，相当于服务器存储连接用户信息的临时容器,session可以用cookie,重写url等方式实现，每一次连接会在服务器产生一个session,使用session-id进行标识，每刷新一次或者开一个子窗口浏览器会自动偷偷发送session-id给服务器
对于Servlet来说就是cookie中的JSESSIONID，通过识别JSESSIONID服务器知道仍然是这个链接所以直接可以抽取出信息并进行处理，如果不是这样，因为http是无状态连接，所以不会保存访问信息，不能浏览网页的时候刷新一下就重新登录一次吧?因此就出现了session这个抽象概念，访问静态html页面没必要创建session,
访问动态页面，如用户登录时就需要，当然浏览器请求或者Servlet自动创建，用户并不知道，这还是为了提供更好的用户体验，可自行设置session的存在时间。
session的实现方式

1.cookie

一个 Web 服务器可以分配一个唯一的 session 会话 ID 作为每个 Web 客户端的 cookie，对于客户端的后续请求可以使用接收到的 cookie 来识别。

2.隐藏的表单字段

一个 Web 服务器可以发送一个隐藏的 HTML 表单字段，以及一个唯一的 session 会话 ID，如下所示：

<input type="hidden" name="sessionid" value="12345"\>
该条目意味着，当表单被提交时，指定的名称和值会被自动包含在 GET 或 POST 数据中。每次当 Web 浏览器发送回请求时，session_id 值可以用于保持不同的 Web 浏览器的跟踪。

这可能是一种保持 session 会话跟踪的有效方式，但是点击常规的超文本链接（<A HREF...>）不会导致表单提交，因此隐藏的表单字段也不支持常规的 session 会话跟踪。

3.URL 重写

您可以在每个 URL 末尾追加一些额外的数据来标识 session 会话，服务器会把该 session 会话标识符与已存储的有关 session 会话的数据相关联。

例如，http://w3cschool.cc/file.htm;sessionid=12345，session 会话标识符被附加为 sessionid=12345，标识符可被 Web 服务器访问以识别客户端。

Servlet中的session

Servlet 提供 HttpSession 接口，该接口提供了一种跨多个页面请求或访问网站时识别用户以及存储有关用户信息的方式。

Servlet 容器使用这个接口来创建一个 HTTP 客户端和 HTTP 服务器之间的 session 会话。会话持续一个指定的时间段，跨多个连接或页面请求。

通过调用 HttpServletRequest 的公共方法 getSession() 来获取 HttpSession 对象，如下所示：

HttpSession session = request.getSession();
你需要在向客户端发送任何文档内容之前调用 request.getSession()。下面总结了 HttpSession 对象中可用的几个重要的方法：

序号	方法 & 描述
1	public Object getAttribute(String name)
该方法返回在该 session 会话中具有指定名称的对象，如果没有指定名称的对象，则返回 null。
2	public Enumeration getAttributeNames()
该方法返回 String 对象的枚举，String 对象包含所有绑定到该 session 会话的对象的名称。
3	public long getCreationTime()
该方法返回该 session 会话被创建的时间，自格林尼治标准时间 1970 年 1 月 1 日午夜算起，以毫秒为单位。
4	public String getId()
该方法返回一个包含分配给该 session 会话的唯一标识符的字符串。
5	public long getLastAccessedTime()
该方法返回客户端最后一次发送与该 session 会话相关的请求的时间自格林尼治标准时间 1970 年 1 月 1 日午夜算起，以毫秒为单位。
6	public int getMaxInactiveInterval()
该方法返回 Servlet 容器在客户端访问时保持 session 会话打开的最大时间间隔，以秒为单位。
7	public void invalidate()
该方法指示该 session 会话无效，并解除绑定到它上面的任何对象。
8	public boolean isNew()
如果客户端还不知道该 session 会话，或者如果客户选择不参入该 session 会话，则该方法返回 true。
9	public void removeAttribute(String name)
该方法将从该 session 会话移除指定名称的对象。
10	public void setAttribute(String name, Object value) 
该方法使用指定的名称绑定一个对象到该 session 会话。
11	public void setMaxInactiveInterval(int interval)
该方法在 Servlet 容器指示该 session 会话无效之前，指定客户端请求之间的时间，以秒为单位。

```java
@WebServlet("/SessionTrack")
public class SessionTrack extends HttpServlet {
    private static final long serialVersionUID=1L;
    public void doGet(HttpServletRequest request, HttpServletResponse response)throws IOException {
        HttpSession session=request.getSession(true);//如果不存在session对象则创建

        Date createTime=new Date(session.getCreationTime());//获取session的创建时间
        Date lastAccessTime=new Date(session.getLastAccessedTime());//获取最后一次传递session的时间
        SimpleDateFormat df=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");//设置简单日期格式
     
        /*
        session会话是一个抽象概念,session存储于服务器内存中，相当于服务器存储连接用户信息的临时容器,session可以用cookie,重写url
        等方式实现，每一次连接会在服务器产生一个session,使用session-id进行标识，每刷新一次或者开一个子窗口浏览器会自动偷偷发送session-id给服务器
        对于Servlet来说就是cookie中的JSESSIONID，通过识别JSESSIONID服务器知道仍然是这个链接所以直接可以抽取出信息并进行处理，如果不是这样，因为http是无状态
        连接，所以不会保存访问信息，不能浏览网页的时候刷新一下就重新登录一次吧?因此就出现了session这个抽象概念，访问静态html页面没必要创建session,
        访问动态页面，如用户登录时就需要，当然浏览器请求或者Servlet自动创建，用户并不知道，这还是为了提供更好的用户体验
         */
        String visitCountKey ="visitCount";
        int visitCount = 0;
     
        String userKey ="userID";//session信息值存储于本地服务器
        String user ="LSL";
        //如果该session没有设置相应的key则直接在服务器设置需要的键值
        if(session.getAttribute(visitCountKey) == null) {
            session.setAttribute(visitCountKey, 0);
        }
     
        // 检查是是新的访问者(由session追踪)
        if (session.isNew()){
            session.setAttribute(userKey, user);
        } else {
            visitCount = (Integer)session.getAttribute(visitCountKey);
            visitCount++;
        }
        session.setAttribute(visitCountKey,  visitCount);//更新session
     
        // 设置响应内容类型
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
     
        out.println("<!DOCTYPE html>\n" +
                "<html>\n" +
                "<head><title>" + "Servlet Session 会话实例 "+ "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + "Servlet Session 会话实例 " + "</h1>\n" +
                "<h2 align=\"center\">Session 信息</h2>\n" +
                "<table border=\"1\" align=\"center\">\n" +
                "<tr bgcolor=\"#949494\">\n" +
                "  <th>Session 信息</th><th>值</th></tr>\n" +
                "<tr>\n" +
                "  <td>id</td>\n" +
                "  <td>" + session.getId() + "</td></tr>\n" +
                "<tr>\n" +
                "  <td>创建时间</td>\n" +
                "  <td>" +  df.format(createTime) +
                "  </td></tr>\n" +
                "<tr>\n" +
                "  <td>最后访问时间</td>\n" +
                "  <td>" + df.format(lastAccessTime) +
                "  </td></tr>\n" +
                "<tr>\n" +
                "  <td>用户 ID</td>\n" +
                "  <td>" + user +
                "  </td></tr>\n" +
                "<tr>\n" +
                "  <td>访问统计：</td>\n" +
                "  <td>" + visitCount + "</td></tr>\n" +
                "</table>\n" +
                "</body></html>");
    }

}
```


这样服务器自动创建session实例并保存session信息，在规定时间内由session-id标识每一个session,如果是处于同一个session则通过getAttribute()等方法获取原来的信息，如果是第一次连接则创建session并通过setAttribute()等方法设置信息。

#### 访问数据库
最原始的方式即通过JDBC访问数据库。

以mysql8.0.15为例

1.导入JDBC驱动

Class.forName(JDBC_DRIVER);//8.0.15中,JDBC_DRIVER="com.mysql.cj.jdbc.Driver";
2.连接数据库，返回连接对象

Connection connection= DriverManager.getConnection(DB_URL,USER,PASSWORD);
8.0.15中，DB_URL后要加上时区，否则抛出异常

DB_URL="jdbc:mysql://localhost:3306/lsl?serverTimezone=UTC";
3.进行实际连接，返回操作对象

Statement statement=connection.createStatement();
4.执行sql,返回结果集或者boolean值

ResultSet resultSet=statement.executeQuery(sql);

```java
@WebServlet("/DatabaseAccess")
public class DatabaseAccess extends HttpServlet {
    private static final long serialVersionUID=1L;

    static final String JDBC_DRIVER="com.mysql.cj.jdbc.Driver";
    static final String DB_URL="jdbc:mysql://localhost:3306/lsl?serverTimezone=UTC";
    static final String USER="root";
    static final String PASSWORD="123456";
     
    public DatabaseAccess(){
        super();
    }
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException{
        Connection connection=null;
        Statement statement=null;
        String sql="select * from test";
     
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        out.println("<!DOCTYPE html>\n"+
                "<html>\n" +
                "<head><title>" +"Servlet数据库访问" + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + "Servlet数据库访问" + "</h1>\n");
     
        try {
            Class.forName(JDBC_DRIVER);
            connection= DriverManager.getConnection(DB_URL,USER,PASSWORD);
            statement=connection.createStatement();
            ResultSet resultSet=statement.executeQuery(sql);
            //迭代结果集
            while (resultSet.next()){
                out.println("ID: " + resultSet.getString("id"));
                out.println(",姓名:" + resultSet.getString("name"));
                out.println(", 性别: " + resultSet.getString("sex"));
                out.println(", 年龄: " + resultSet.getString("age"));
                out.println("<br />");
            }
            resultSet.close();//关闭
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                //迭代完之后记得关闭资源
                statement.close();
                connection.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
    @Override
    protected void doPost(HttpServletRequest request,HttpServletResponse response) throws IOException{
        doGet(request,response);
    }

}
```

前端与数据库通过中间servlet交互，如果想要提取表单值并插入数据库中，可以这样操作:

/编写预处理 SQL 语句
String sql= "INSERT INTO websites1 VALUES(?,?,?,?,?)";
//实例化 PreparedStatement
ps = conn.prepareStatement(sql);
//传入参数，这里的参数来自于一个带有表单的jsp文件，很容易实现
ps.setString(1, request.getParameter("id"));
ps.setString(2, request.getParameter("name"));
ps.setString(3, request.getParameter("url"));
ps.setString(4, request.getParameter("alexa"));
ps.setString(5, request.getParameter("country"));
//执行数据库更新操作，不需要SQL语句
ps.executeUpdate();
sql = "SELECT id, name, url FROM websites1";
ps = conn.prepareStatement(sql);
ResultSet rs = ps.executeQuery();//获取查询结果
#### 网页重定向
当因为某种原因需要进行页面的自动跳转时，即页面重定向

servlet中页面重定向有两种方法

1.response调用sendRedirect()方法

2.设置头部状态码及指定跳转页面

String site = "http://www.baidu.com" ;
response.setStatus(response.SC_MOVED_TEMPORARILY);
response.setHeader("Location", site); 

```java
@WebServlet("/PageRedirect")
public class PageRedirect extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response)throws IOException{
        response.setContentType("text/html;charset=UTF-8");
        response.sendRedirect("http://www.baidu.com");//调用sendRedirect()重定向
    }
    public void doPost(HttpServletRequest request,HttpServletResponse response)throws IOException{
        doGet(request,response);
    }
}
```

#### 网页访问量计数器
单个网页只需要在初始化servlet的init()方法中初始化计数器为0即可(或者如果服务器重启则从数据库中读取数据),由servlet的生命周期可知，每次访问都会调用doGet()等方法，对应计数器加1即可。或者使用过滤器Filter即可。

总点击量可通过设置过滤器filter完成。

Servlet 自动刷新页面
调用方法设置头部自动刷新即可

public void setIntHeader(String header, int headerValue)
如：setIntHeader("Refresh",1)即设置页面每隔一秒自动刷新一次。

#### Servlet请求方式

```
Servlet容器默认是采用单实例多线程的方式处理多个请求的：
　　1.当web服务器启动的时候（或客户端发送请求到服务器时），Servlet就被加载并实例化(只存在一个Servlet实例)；
　　2.容器初始化化Servlet主要就是读取配置文件（例如tomcat,可以通过servlet.xml的<Connector>设置线程池中线程数目，初始化线程池通过web.xml,初始化每个参数值等等。
　　3.当请求到达时，Servlet容器通过调度线程(Dispatchaer Thread) 调度它管理下线程池中等待执行的线程（Worker Thread）给请求者；
　　4.线程执行Servlet的service方法；
　　5.请求结束，放回线程池，等待被调用；
（注意：避免使用实例变量（成员变量），因为如果存在成员变量，可能发生多线程同时访问该资源时，都来操作它，照成数据的不一致，因此产生线程安全问题）
从上面可以看出（好处）：
　　第一：Servlet单实例，减少了产生servlet的开销；
　　第二：通过线程池来响应多个请求，提高了请求的响应时间；
　　第三：Servlet容器并不关心到达的Servlet请求访问的是否是同一个Servlet还是另一个Servlet，直接分配给它一个新的线程；　　　　　　如果是同一个Servlet的多个请求，那么Servlet的service方法将在多线程中并发的执行；
　　第四：每一个请求由ServletRequest对象来接受请求，由ServletResponse对象来响应该请求；
```

#### JSP概述

**参考： https://zhuanlan.zhihu.com/p/42343690**

前端请求request->应用层协议http/https封装info->网络层/传输层->服务器网络层->http/https->info，服务器接收info，Tomcat Server封装成request，数据拿来，处理后封装成response进行响应，如何返回响应,response中只有通过流的方式进行数据传输

1. PrintWriter out=response.getWriter();
2. out.println()

![preview](https://pic2.zhimg.com/v2-fc8bb977784e7b703bfd157192825899_r.jpg)  

可以看到，返回语句通过response静态打印输出，如果要返回定制页面，需要专门的处理类处理完数据后用对象存储，通过out接着不断加载打印信息

 **我们的主要目的就是希望在最终输出的html的代码中嵌入后台数据罢了。**除了把html语句拿出来在Servlet里拼接好再输出这种方式外，我们也可以直接在html语句中写入动态数据 

为了在响应中动态加载并显示信息，出现了各种后台语言/技术，可以将jsp看成这样一种后台技术

   JSP全称Java Server Page，直译就是“运行在服务器端的页面”。我们可以直接在JSP文件里写HTML代码，使用上把它**当做**HTML文件。而且JSP中HTML/CSS/JS等的写法和HTML文件中的写法是一模一样的。但它毕竟不是HTML，而且本质差了十万八千里。因为我们还可以把Java代码内嵌在JSP页面中，很方便地把动态数据渲染成静态页面。这一点，HTML打死都做不到。 浏览器只能解析静态页面 。

当有人请求JSP时，服务器内部会经历一次动态资源（JSP）到静态资源（HTML）的转化，服务器会自动帮我们把JSP中的HTML片段和数据拼接成静态资源响应给浏览器。也就是说JSP是运行在服务器端，但最终发给客户端的都已经是转换好的HTML静态页面（在响应体里）。

即：**JSP = HTML + Java片段**（各种标签本质上还是Java片段）![preview](https://pic3.zhimg.com/v2-092cb7fb1de6877f53751531d05675de_r.jpg)   

request->容器识别jsp->服务器端加载jsp，转换成servlet后进行数据处理并返回静态页面（数据可以实时嵌入页面中）

 ![preview](https://pic2.zhimg.com/v2-83c1a7e3ae1e9d236ce568347dd2c3b1_r.jpg) 

 ![preview](https://pic2.zhimg.com/v2-f936b7b3ddef8248879c7f414475d459_r.jpg) 

 原来，为了不让Java程序员一行行复制HTML代码到Servlet里，**SUN公司干脆让Java程序员直接把HTML写在了Servlet里！**但是毕竟SUN还没有那么明目张胆，好歹让这个Servlet伪装了一把，打扮成JSP，然后跟程序员说：看，我搞了个JSP，这家伙可牛逼了，你能在上面同时写HTML和Java代码哦。

得了吧，等你写完JSP，回头访问时，Tomcat就把这个JSP翻译成Servlet，原先写在JSP里的HTML代码就**自动**放在了out.println()里啦！**相当于程序帮我做了“逐行复制HTML代码到Servlet”这一步。** 

 ![preview](https://pic1.zhimg.com/v2-955ff75e5217fe00274a13ca880cb80c_r.jpg) 

作为后台动态语言，jsp自有其语法格式

### 基本语法

##### 内嵌java代码<% ... %>

```html
<% 代码片段 %>
```

##### 显示中文<%@ page  ... %>

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
```

##### 声明变量<%! ... %>

```html
<%! int i = 0; %> 
<%! int a, b, c; %> 
<%! Circle a = new Circle(2.0); %> 
```

##### 表达式<%= ... %>

表达式的值会被转化成String，然后插入到表达式出现的地方。表达式不能使用分号来结束表达式。

```html
<%= 表达式 %>
例：
<p>
   今天的日期是: <%= (new java.util.Date()).toLocaleString()%>
</p>
```

##### 注释<%-- ... --%>

<%-- 注释 --%>

#### JSP指令<%@ ... %>

JSP指令用来设置与整个JSP页面相关的属性。

JSP指令语法格式：

```html
<%@ directive attribute="value" %>
```

这里有三种指令标签：

| **指令**           | **描述**                                                  |
| ------------------ | --------------------------------------------------------- |
| <%@ page ... %>    | 定义页面的依赖属性，比如脚本语言、error页面、缓存需求等等 |
| <%@ include ... %> | 包含其他文件                                              |
| <%@ taglib ... %>  | 引入标签库的定义，可以是自定义标签                        |

##### 指令@ page：定义整个页面的相关信息

| **属性**           | **描述**                                            |
| ------------------ | --------------------------------------------------- |
| buffer             | 指定out对象使用缓冲区的大小                         |
| autoFlush          | 控制out对象的 缓存区                                |
| contentType        | 指定当前JSP页面的MIME类型和字符编码                 |
| errorPage          | 指定当JSP页面发生异常时需要转向的错误处理页面       |
| isErrorPage        | 指定当前页面是否可以作为另一个JSP页面的错误处理页面 |
| extends            | 指定servlet从哪一个类继承                           |
| import             | 导入要使用的Java类                                  |
| info               | 定义JSP页面的描述信息                               |
| isThreadSafe       | 指定对JSP页面的访问是否为线程安全                   |
| language           | 定义JSP页面所用的脚本语言，默认是Java               |
| session            | 指定JSP页面是否使用session                          |
| isELIgnored        | 指定是否执行EL表达式                                |
| isScriptingEnabled | 确定脚本元素能否被使用                              |

例：

```html
<%@ page import="java.util.Date" %>
```

##### 指令@ include:包含其他文件，包含的文件就好像是该JSP文件的一部分，会被同时编译执行

```html
<%@ include file="文件相对 url 地址" %>
```

编译器默认在当前路径下寻找。

#### JSP行为<jsp: ... />

JSP行为标签使用XML语法结构来控制servlet引擎。它能够动态插入一个文件，重用JavaBean组件，引导用户去另一个页面，为Java插件产生相关的HTML等等。主要是可以用javaBean

```html
<jsp:action_name attribute="value" />
```

行为标签基本上是一些预先就定义好的函数，下表罗列出了一些可用的JSP行为标签：

| **语法**        | **描述**                                                   |
| --------------- | ---------------------------------------------------------- |
| jsp:include     | 用于在当前页面中包含静态或动态资源                         |
| jsp:useBean     | 寻找和初始化一个JavaBean组件                               |
| jsp:setProperty | 设置 JavaBean组件的值                                      |
| jsp:getProperty | 将 JavaBean组件的值插入到 output中                         |
| jsp:forward     | 从一个JSP文件向另一个文件传递一个包含用户请求的request对象 |
| jsp:plugin      | 用于在生成的HTML页面中包含Applet和JavaBean对象             |
| jsp:element     | 动态创建一个XML元素                                        |
| jsp:attribute   | 定义动态创建的XML元素的属性                                |
| jsp:body        | 定义动态创建的XML元素的主体                                |
| jsp:text        | 用于封装模板数据                                           |

**所有的动作要素都有两个属性：id属性和scope属性。**

- id属性：

  id属性是动作元素的唯一标识，可以在JSP页面中引用。动作元素创建的id值可以通过PageContext来调用。

- scope属性：

  该属性用于识别动作元素的生命周期。 id属性和scope属性有直接关系，scope属性定义了相关联id对象的寿命。 scope属性有四个可能的值： (a) page, (b)request, (c)session, 和 (d) application。

使用行为标签就可以动态加载页面，复用其它类

##### jsp:include把指定文件(jsp)插入正在生成的页面

```html
<jsp:include page="相对 URL 地址" flush="true" />
```

例：

date.jsp

```jsp
<%@ page import="java.util.Date" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<<p>
今天的日期是:<%=(new Date().toLocaleString())%>
</p>
```

动态引用date.jsp的hello.jsp

```jsp
<%@ page import="java.util.Date" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head><title>Hello World</title></head>
<body>
<jsp:include page="date.jsp" flush="true" />
</body>
</html>
```

##### jsp:useBean加载一个将在JSP页面中使用的JavaBean。

```html
<jsp:useBean id="name" class="package.class" />
```

在类载入后，可以通过 jsp:setProperty 和 jsp:getProperty 动作来修改和检索bean的属性。

##### jsp:setProperty设置已经实例化的Bean对象的属性

用法1：

```jsp
<jsp:useBean id="myName" ... />
...
<jsp:setProperty name="myName" property="someProperty" value="xxx".../>
```

用法2：

```jsp
<jsp:useBean id="myName" ... >
...
   <jsp:setProperty name="myName" property="someProperty" value="xxx".../>
</jsp:useBean>
```

即通过id检索javaBean，指定属性property（value可选，为属性设置值）

##### jsp:getProperty提取指定Bean属性的值，转换成字符串，然后输出

```jsp
<jsp:useBean id="myName" ... />
...
<jsp:getProperty name="myName" property="someProperty" .../>
```

即通过id检索javaBean，获取指定属性property

### JSP隐含对象

JSP支持九个自动定义的变量，江湖人称隐含对象，可以直接使用它们而不用显式声明

| **对象**    | **描述**                                                     |
| ----------- | ------------------------------------------------------------ |
| request     | **HttpServletRequest**类的实例                               |
| response    | **HttpServletResponse**类的实例                              |
| out         | **PrintWriter**类的实例，用于把结果输出至网页上              |
| session     | **HttpSession**类的实例                                      |
| application | **ServletContext**类的实例，与应用上下文有关                 |
| config      | **ServletConfig**类的实例                                    |
| pageContext | **PageContext**类的实例，提供对JSP页面所有对象以及命名空间的访问 |
| page        | 类似于Java类中的this关键字                                   |
| Exception   | **Exception**类的对象，代表发生错误的JSP页面中对应的异常对象 |

**request对象**

request对象是javax.servlet.http.HttpServletRequest 类的实例。每当客户端请求一个JSP页面时，JSP引擎就会制造一个新的request对象来代表这个请求。request对象提供了一系列方法来获取HTTP头信息，cookies，HTTP方法等等。

**response对象**

response对象是javax.servlet.http.HttpServletResponse类的实例。当服务器创建request对象时会同时创建用于响应这个客户端的response对象。response对象也定义了处理HTTP头模块的接口。通过这个对象，开发者们可以添加新的cookies，时间戳，HTTP状态码等等。

**out对象**

out对象是 javax.servlet.jsp.JspWriter 类的实例，用来在response对象中写入内容。

| **方法**                     | **描述**                 |
| ---------------------------- | ------------------------ |
| **out.print(dataType dt)**   | 输出Type类型的值         |
| **out.println(dataType dt)** | 输出Type类型的值然后换行 |
| **out.flush()**              | 刷新输出流               |

**session对象**

session对象是 javax.servlet.http.HttpSession 类的实例。和Java Servlets中的session对象有一样的行为。

session对象用来跟踪在各个客户端请求间的会话。

**application对象**

application对象直接包装了servlet的ServletContext类的对象，是javax.servlet.ServletContext 类的实例。

这个对象在JSP页面的整个生命周期中都代表着这个JSP页面。这个对象在JSP页面初始化时被创建，随着jspDestroy()方法的调用而被移除。通过向application中添加属性，则所有组成web应用的JSP文件都能访问到这些属性。

**config对象**

config对象是 javax.servlet.ServletConfig 类的实例，直接包装了servlet的ServletConfig类的对象。

这个对象允许开发者访问Servlet或者JSP引擎的初始化参数，比如文件路径等。

**pageContext 对象**

pageContext对象是javax.servlet.jsp.PageContext 类的实例，用来代表整个JSP页面。

这个对象主要用来访问页面信息，同时过滤掉大部分实现细节。

这个对象存储了request对象和response对象的引用。application对象，config对象，session对象，out对象可以通过访问这个对象的属性来导出。

pageContext对象也包含了传给JSP页面的指令信息，包括缓存信息，ErrorPage URL,页面scope等。

PageContext类定义了一些字段，包括PAGE_SCOPE，REQUEST_SCOPE，SESSION_SCOPE， APPLICATION_SCOPE。它也提供了40余种方法，有一半继承自javax.servlet.jsp.JspContext 类。

其中一个重要的方法就是removeArribute()，它可接受一个或两个参数。比如，pageContext.removeArribute("attrName")移除四个scope中相关属性，但是下面这种方法只移除特定scope中的相关属性：

```html
pageContext.removeAttribute("attrName", PAGE_SCOPE);
```

**page 对象**

这个对象就是页面实例的引用。它可以被看做是整个JSP页面的代表。

page 对象就是this对象的同义词。

**exception 对象**

exception 对象包装了从先前页面中抛出的异常信息。它通常被用来产生对出错条件的适当响应。