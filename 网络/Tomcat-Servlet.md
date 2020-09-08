## Servlet 接口

Servlet编程是通过javax.servlet和 javax.servlet.http 这两个包的类和接口来实现的。其中一个至关重要的就是 javax.servlet.Servlet 接口了。所有的 servlet 必须实现实现或者继承实现该接口的类。Servlet 接口有五个方法

```java
public void init(ServletConfig config) throws ServletException
public void service(ServletRequest request, ServletResponse response)throws ServletException, java.io.IOException
public void destroy()
public ServletConfig getServletConfig()
public java.lang.String getServletInfo()
```

在Servlet的五个方法中，init，service和destroy是servlet的生命周期方法。在servlet类已经初始化之后，init 方法将会被 servlet 容器所调用。servlet 容器只调用一次，以此表明servlet 已经被加载进服务中。init 方法必须在 servlet 可以接受任何请求之前成功运行完毕。一个 servlet 程序员可以通过覆盖这个方法来写那些仅仅只要运行一次的初始化代码，例如加载数据库驱动，值初始化等等。在其他情况下，这个方法通常是留空的。

servlet 容器 为 servlet 请求调用它的 service 方法 。servlet 容器传递一个javax.servlet.ServletRequest对象和javax.servlet.ServletResponse对象。ServletRequest对象包括客户端的 HTTP 请求信息，而 ServletResponse 对象封装 servlet 的响应。在 servlet的生命周期中，service 方法将会给调用多次。

当从服务中移除一个 servlet 实例的时候，servlet 容器调用 destroy 方法。这通常发生在servlet 容器正在被关闭或者 servlet 容器需要一些空闲内存的时候。仅仅在所有 servlet 线程的 service 方法已经退出或者超时淘汰的时候，这个方法才被调用。在 servlet 容器已经调用完destroy 方法之后，在同一个 servlet 里边将不会再调用 service 方法。destroy 方法提供了一个机会来清理任何已经被占用的资源，例如内存，文件句柄和线程，并确保任何持久化状态和servlet 的内存当前状态是同步的。

PrimitiveServlet 类实现了javax.servlet.Servlet(所有的 servlet 都必须这样做)，并为 Servlet 的这五个方法都提供了实现。PrimitiveServlet 做的事情非常简单。在 init，service 或者 destroy 中的任何一个方法每次被调用的时候，servlet 把方法名写到标准控制台上面去。另外，service 方法从ServletResponse 对象获得java.io.PrintWriter 实例，并发送字符串到浏览器去。

### 简单Servlet

```java
import javax.servlet.*;
import java.io.IOException;
import java.io.PrintWriter;

public class PrimitiveServlet implements Servlet {
    public void init(ServletConfig config) throws ServletException {
        System.out.println("init");
    }
    public void service(ServletRequest request, ServletResponse response)
            throws ServletException, IOException {
        System.out.println("from service");
        PrintWriter out = response.getWriter();
        out.println("Hello. Roses are red.");
        out.print("Violets are blue.");
    }
    public void destroy() {
        System.out.println("destroy");
    }
    public String getServletInfo() {
        return null;
    }
    public ServletConfig getServletConfig() {
        return null;
    }
}
```

## Servlet容器

现在，让我们从一个 servlet 容器的角度来研究一下 servlet 编程。总的来说，一个全功能的 servlet 容器会为servlet 的每个 HTTP 请求做下面一些工作：

1. 当第一次调用 servlet 的时候，加载该 servlet 类并调用 servlet 的 init 方法(仅仅一次)。
2.  对每次请求，构造一个 javax.servlet.ServletRequest 实例和一个javax.servlet.ServletResponse 实例。
3. 调用 servlet 的 service 方法，同时传递 ServletRequest 和 ServletResponse 对象。
4. 当 servlet 类被关闭的时候，调用 servlet 的 destroy 方法并卸载 servlet 类。

也就是说Servlet提供对请求的应用服务能力，并返回响应。Servlet容器则解析请求，交给对应的Servlet去处理。

### 简单Servlet容器

<img src="https://i.loli.net/2020/03/23/xKnfZPNEC5Ui4eR.png" alt="image.png" style="zoom: 67%;" />

浏览器与服务器进行socket通信，服务器从请求中构造输入与输出流，传递给Request与Response构造性相应的HTTP请求与响应对象，根据Url进行解析交给相应的Servlet进行处理。

**Request**

servlet 的 service 方法从 servlet 容器中接收一个 javax.servlet.ServletRequest 实例和一个javax.servlet.ServletResponse 实例。这就是说对于每一个 HTTP 请求，servlet 容器必须构造一个ServletRequest 对象和一个 ServletResponse 对象并把它们传递给正在服务的servlet 的 service 方法。

Request 类代表一个 request 对象并被传递给 servlet 的 service 方法。就本身而言，它必须实现 javax.servlet.ServletRequest 接口。

```java
public class Request implements ServletRequest {
    private InputStream input;
    private String uri;
    public Request(InputStream input) {
        this.input = input;
    }
    public String getUri() {
        return uri;
    }
    public void parse() {
        //从socket中读取数据
        StringBuffer request = new StringBuffer(2048);
        int i;
        byte[] buffer = new byte[2048];
        try {
            i = input.read(buffer);
        } catch (IOException e) {
            e.printStackTrace();
            i = -1;
        }
        for (int j = 0; j < i; j++) {
            request.append((char) buffer[j]);
        }
        System.out.print(request.toString());
        uri = parseUri(request.toString());
    }
    private String parseUri(String requestString) {
        int index1, index2;
        index1 = requestString.indexOf(' ');
        if (index1 != -1) {
            index2 = requestString.indexOf(' ', index1 + 1);
            if (index2 > index1)
                return requestString.substring(index1 + 1, index2);
        }
        return null;
    }

    /**
     * 一大段接口方法，此处省略
     */
}
```

**Response**

Response 类代表一个 response 对象并被传递给 servlet 的 service 方法。就本身而言，它必须实现 javax.servlet.ServletResponse 接口。

```java
/**
 * 简单Servlet响应对象
 */
public class Response implements ServletResponse {
    private static final int BUFFER_SIZE = 1024;
    Request request;
    OutputStream output;
    PrintWriter writer;

    public Response(OutputStream output) {
        this.output = output;
    }

    public void setRequest(Request request) {
        this.request = request;
    }

    //返回静态资源
    public void sendStaticResource() throws IOException {
        byte[] bytes = new byte[BUFFER_SIZE];
        FileInputStream fis = null;
        try {
            /*从根目录中读取指定资源 */
            File file = new File(Constants.WEB_ROOT, request.getUri());
            fis = new FileInputStream(file);

            int ch = fis.read(bytes, 0, BUFFER_SIZE);
            while (ch != -1) {
                output.write(bytes, 0, ch);
                ch = fis.read(bytes, 0, BUFFER_SIZE);
            }
        } catch (FileNotFoundException e) {
            String errorMessage = "HTTP/1.1 404 File Not Found\r\n" +
                    "Content-Type: text/html\r\n" +
                    "Content-Length: 23\r\n" +
                    "\r\n" +
                    "<h1>File Not Found</h1>";
            output.write(errorMessage.getBytes());
        } finally {
            if (fis != null)
                fis.close();
        }
    }
    @Override
    public PrintWriter getWriter() throws IOException {
        writer = new PrintWriter(output, true);
        return writer;
    }
    /**
     * 一大段接口方法，此处省略
    */
}
```

**HttpServer**

应用服务器的主体，建立Socket通信，接收HTTP请求

```java
import java.net.Socket;
import java.net.ServerSocket;
import java.net.InetAddress;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.IOException;

/**
 * 简单服务器，Servlet容器
 */
public class HttpServer {
    private static final String SHUTDOWN_COMMAND = "/SHUTDOWN";
    // the shutdown command received
    private boolean shutdown = false;
    public static void main(String[] args) {
        HttpServer server = new HttpServer();
        server.await();
    }
    public void await() {
        ServerSocket serverSocket = null;
        int port = 8080;
        try {
            serverSocket = new ServerSocket(port, 1,
                    InetAddress.getByName("127.0.0.1"));
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(1);
        }
        //循环等待连接
        while (!shutdown) {
            Socket socket = null;
            InputStream input = null;
            OutputStream output = null;
            try {
                socket = serverSocket.accept();
                input = socket.getInputStream();
                output = socket.getOutputStream();
                //创建请求对象并序列化
                Request request = new Request(input);
                request.parse();
                //创建响应对象并序列化
                Response response = new Response(output);
                response.setRequest(request);
                //检查uri，进行解析，判断是返回静态资源还是交给Servlet处理
                if (request.getUri().startsWith("/servlet/")) {
                    ServletProcessor processor = new ServletProcessor();
                    processor.process(request, response);
                } else {
                    StaticResourceProcessor processor =
                            new StaticResourceProcessor();
                    processor.process(request, response);
                }
                //socket关闭
                socket.close();
                //检查是否关闭服务器
                shutdown = request.getUri().equals(SHUTDOWN_COMMAND);
            } catch (Exception e) {
                e.printStackTrace();
                System.exit(1);
            }
        }
    }
}
```

可以看到HttpServer根据URl进行解析，既可以访问静态资源也可以交给Servlet去处理

**ServletProcessor**

```java
/**
 * 简单Servlet处理器
 */
public class ServletProcessor {
    public void process(Request request, Response response) throws ServletException, IOException {
        String uri = request.getUri();
        String servletName = uri.substring(uri.lastIndexOf("/") + 1);
        System.out.println(servletName);
        //使用类加载器加载Servlet并进行处理
        Servlet servlet = new PrimitiveServlet();
        servlet.service((ServletRequest) request,
                (ServletResponse) response);
    }
}
```

**StaticResourceProcessor**

```java
/**
 * 简单静态资源处理器
 */
public class StaticResourceProcessor {
    public void process(Request request, Response response) {
        try {
            response.sendStaticResource();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**Constants**

```java
public class Constants {
    public static final String WEB_ROOT = new File("").getAbsolutePath() + "\\";
}
```

对于url要指定的Servlet来说，可以使用类加载器加载所需要的Servlet实例并调用service进行服务

## 连接器

一个符合 Servlet 2.3 和 2.4规 范 的 连 接 器 必 须 创 建 javax.servlet.http.HttpServletRequest 和
javax.servlet.http.HttpServletResponse，并传递给被调用的 servlet 的 service 方法。连接器解析 HTTP 请求头部并让 servlet 可以获得头部, cookies, 参数名/值等等。

### StringManager 类

一个像 Tomcat 这样的大型应用需要仔细的处理错误信息。在 Tomcat 中，错误信息对于系统管理员和 servlet 程序员都是有用的。例 如，Tomcat 记录错误信息，让系统管理员可以定位发生 的 任 何 异 常 。 对 servlet 程 序 员 来 说 ， Tomcat 会 在 抛 出 的 任 何 一 个javax.servlet.ServletException 中发送一个错误信息，这样程序员可以知道他/她的 servlet究竟发送什么错误了。

Tomcat 所采用的方法是在一个属性文件里边存储错误信息，这样，可以容易的修改这些信息。不过，Tomcat 中有数以百计的类。把所有类使用的错误信 息存储到一个大的属性文件里边将会容易产生维护的噩梦。为了避免这一情况，Tomcat 为每个包都分配一个属性文件。例如，在包 org.apache.catalina.connector 里边的属性文件包含了该包所有的类抛出的所有错误信息。每个属性文件都会被一个 org.apache.catalina.util.StringManager 类的实例所处理。当Tomcat 运行时，将会有许多 StringManager 实例，每个实例会读取包对应的一个属性文件。此
外，由于 Tomcat 的受欢迎程度，提供多种语言的错误信息也是有意义的。

当包里边的一个类需要查找放在该包属性文件的一个错误信息时，它首先会获得一个StringManager 实例。不过，相同包里边的许多类可能也需要 StringManager，为每个对象创建一个 StringManager 实例是一种资源浪费。因此，StringManager 类被设计成一个 StringManager实例可以被包里边的所有类共享（单例模式）。通过传递一个包名来调用它的公共静态方法 getManager 来获得一个实例。每个实例存储在一个以包名为键(key)的 Hashtable 中。

```java
private static Hashtable managers = new Hashtable();
public synchronized static StringManager getManager(String packageName) {
	StringManager mgr = (StringManager)managers.get(packageName);
	if (mgr == null) {
		mgr = new StringManager(packageName);
		managers.put(packageName, mgr);
	}
	return mgr;
}
```

### 架构



HttpServer 类被分离为两个类：HttpConnector和 HttpProcessor，Request 被 HttpRequest 所取代，而 Response 被 HttpResponse 所取代。

等待 HTTP 请求的工作交给 HttpConnector 实例，而创建请求和响应对象的工作交给了HttpProcessor 实例。

HTTP 请求对象由实现了 javax.servlet.http.HttpServletRequest 的 HttpRequest类来代表。一个 HttpRequest 对象将会给转换为一个 HttpServletRequest 实例并传递给被调用的 servlet 的 service 方法。因此，每个 HttpRequest 实例必须适当增加字段，以便 servlet可以使用它们。值需要赋给 HttpRequest 对象，包括 URI，查询字符串，参数，cookies 和其他的头部等等。因为连接器并不知道被调用的 servlet 需要哪个值，所以连接器必须从 HTTP 请求中解析所有可获得的值。不过，解析一个 HTTP 请求牵涉昂贵的字符串和其他操作，假如只是解
析 servlet 需要的值的话，连接器就能节省许多 CPU 周期。例如，假如 servlet 不 解析任何一个 请 求 参 数 ( 例 如 不 调 用 javax.servlet.http.HttpServletRequest 的 getParameter,getParameterMap,getParameterNames 或者 getParameterValues 方法)，连接器就不需要从查询字符串或者 HTTP 请求内容中解析这些参数。

Tomcat 的默认连接器(和本章应用程序的连接器)试图不解析参数直到 servlet 真正需要它的时候，通过这样来获得更高效率。

Tomcat的默认连接器和我们的连接器使用SocketInputStream类来从套接字的InputStream中读取字节流。一个 SocketInputStream 实例对从套接字的 getInputStream 方法中返回的java.io.InputStream 实例进行包装。 SocketInputStream 类提供了两个重要的方法：readRequestLine 和 readHeader。readRequestLine 返回一个 HTTP 请求的第一行。例如，这行包括了 URI，方法和 HTTP 版本。因为从套接字的输入流中处理字节流意味着只读取一次，从第一个字节到最后一个字节(并且不回退)，因此 readHeader 被调用之前，readRequestLine 必须
只被调用一次。readHeader 每次被调用来获得一个头部的名/值对，并且应该被重复的调用直到所有的头部被读取到。readRequestLine 的返回值是一个 HttpRequestLine 的实例，而readHeader 的返回值是一个HttpHeader 对象。

HttpProcessor 对象创建了 HttpRequest 的实例，因此必须在它们当中增加字段。HttpProcessor 类使用它的 parse 方法 来解析一个 HTTP 请求中的请求行和头部。解析出来并把值赋给 HttpProcessor 对象的这些字段。不过，parse 方法并不解析请求内容或者请求字符串里边的参数。这个任务留给了 HttpRequest 对象它们。只是当 servlet 需要一个参数时，查询字符串或者请求内容才会被解析。

### startup模块

**Bootstrap**

用于启动整个应用程序



**HttpConnector**

指代一个连接器，职责是创建一个服务器套接字用来等待前来的 HTTP 请求。HttpConnector 类实现了 java.lang.Runnable，所以它能被它自己的线程专用。当你启动应用程序，一个 HttpConnector 的实例被创建，并且它的 run 方法被执行。

run 方法包括一个 while 循环，用来做下面的事情：

* 等待 HTTP 请求
*  为每个请求创建个 HttpProcessor 实例
* 调用 HttpProcessor 的 process 方法



```
public void process(Socket socket) {
SocketInputStream input = null;
OutputStream output = null;
try {
input = new SocketInputStream(socket.getInputStream(), 2048);
output = socket.getOutputStream();
// create HttpRequest object and parse
request = new HttpRequest(input);
// create HttpResponse object
response = new HttpResponse(output);
response.setRequest(request);
response.setHeader("Server", "Pyrmont Servlet Container");
parseRequest(input, output);
parseHeaders(input);
//check if this is a request for a servlet or a static resource
//a request for a servlet begins with "/servlet/"
if (request.getRequestURI().startsWith("/servlet/")) {
ServletProcessor processor = new ServletProcessor();
processor.process(request, response);
}
else {
StaticResourceProcessor processor = new
StaticResourceProcessor();
processor.process(request, response);
}
// Close the socket
socket.close();
// no shutdown for this application
}
catch (Exception e) {
 e.printStackTrace ();
}
}
```

#### HttpRequest 

HttpRequest 类实现了 javax.servlet.http.HttpServletRequest。跟随它的是一个叫做HttpRequestFacade 的 facade 类。

HttpRequest 类的很多方法都留空，但是 servlet 程序员已经可以从到来的 HTTP 请求中获得头部，cookies 和参数。这三种类型的值被存储在下面几个引用变量中：

* protected HashMap headers = new HashMap();
* protected ArrayList cookies = new ArrayList();
* protected ParameterMap parameters = null;

因此，一个 servlet 程序员可以从 javax.servlet.http.HttpServletRequest 中的下列方法中取得正确的返回值：getCookies,getDateHeader,getHeader, getHeaderNames, getHeaders,getParameter, getPrameterMap,getParameterNames 和 getParameterValues 。 

不用说，这里主要的挑战是解析 HTTP 请求和填充 HttpRequest 类。对于头部和 cookies，HttpRequest 类提供了 addHeader 和 addCookie 方法用于 HttpProcessor 的 parseHeaders 方法调用。当需要的时候，会使用 HttpRequest 类的 parseParameters 方法来解析参数。

**读取套接字的输入流**
在上面的程序中仅仅只是读取缓冲区的字符，并没有对报文进行解析

```java
byte[] buffer = new byte [2048];
try {
	// input is the InputStream from the socket.
	i = input.read(buffer);
}
```

org.apache.catalina.connector.http.SocketInputStream这个类提供了方法不仅用来获取请求行，还有请求头部。
你通过传递一个 InputStream 和一个指代实例使用的缓冲区大小的整数，来构建一个SocketInputStream 实例。

拥有一个 SocketInputStream 是为了两个重要方法：readRequestLine和 readHeader。

**解析请求行**

HttpProcessor的process方法调用私有方法parseRequest用来解析请求行，例如一个HTTP请求的第一行。这里是一个请求行的例子：GET /myApp/ModernServlet?userName=tarzan&password=pwd HTTP/1.1

请求行的第二部分是 URI 加上一个查询字符串。在上面的例子中，URI 是这样的：/myApp/ModernServlet
另外，在问号后面的任何东西都是查询字符串。因此，查询字符串是这样的：userName=tarzan&password=pwd

在 servlet/JSP 编程中，参数名 jsessionid 是用来携带一个会话标识符。会话标识符经常被作为 cookie 来嵌入，但是程序员可以选择把它嵌入到查询字符串去，例如，当浏览器的 cookie 被禁用的时候。

当 parseRequest 方法被 HttpProcessor 类的 process 方法调用的时候，request 变量指向一个 HttpRequest 实例。parseRequest 方法解析请求行用来获得几个值并把这些值赋给HttpRequest 对象。

```java
private void parseRequest(SocketInputStream input, OutputStream output)
throws IOException, ServletException {
// Parse the incoming request line
input.readRequestLine(requestLine);
String method =
new String(requestLine.method, 0, requestLine.methodEnd);
String uri = null;
String protocol = new String(requestLine.protocol, 0,
requestLine.protocolEnd);
// Validate the incoming request line
if (method, length () < 1) {
throw new ServletException("Missing HTTP request method");
}
else if (requestLine.uriEnd < 1) {
throw new ServletException("Missing HTTP request URI");
}
// Parse any query parameters out of the request URI
int question = requestLine.indexOf("?");
if (question >= 0) {
request.setQueryString(new String(requestLine.uri, question + 1,
requestLine.uriEnd - question - 1));
uri = new String(requestLine.uri, 0, question);
}
else {
request.setQueryString(null);
uri = new String(requestLine.uri, 0, requestLine.uriEnd);
}
// Checking for an absolute URI (with the HTTP protocol)
if (!uri.startsWith("/")) {
int pos = uri.indexOf("://");
// Parsing out protocol and host name
if (pos != -1) {
pos = uri.indexOf('/', pos + 3);
if (pos == -1) {
uri = "";
}
else {
uri = uri.substring(pos);
}
 }
}
// Parse any requested session ID out of the request URI
String match = ";jsessionid=";
int semicolon = uri.indexOf(match);
if (semicolon >= 0) {
String rest = uri.substring(semicolon + match,length());
int semicolon2 = rest.indexOf(';');
if (semicolon2 >= 0) {
request.setRequestedSessionId(rest.substring(0, semicolon2));
rest = rest.substring(semicolon2);
}
else {
request.setRequestedSessionId(rest);
rest = "";
}
request.setRequestedSessionURL(true);
uri = uri.substring(0, semicolon) + rest;
}
else {
request.setRequestedSessionId(null);
request.setRequestedSessionURL(false);
}
// Normalize URI (using String operations at the moment)
String normalizedUri = normalize(uri);
// Set the corresponding request properties
((HttpRequest) request).setMethod(method);
request.setProtocol(protocol);
if (normalizedUri != null) {
((HttpRequest) request).setRequestURI(normalizedUri);
}
else {
((HttpRequest) request).setRequestURI(uri);
}
if (normalizedUri == null) {
throw new ServletException("Invalid URI: " + uri + "'");
}
}
```



parseRequest 方法首先调用 SocketInputStream 类的 readRequestLine 方法：input.readRequestLine(requestLine);
在这里 requestLine 是 HttpProcessor 里边的 HttpRequestLine 的一个实例：
private HttpRequestLine requestLine = new HttpRequestLine();
调用它的 readRequestLine 方法来告诉 SocketInputStream 去填入 HttpRequestLine 实例。
接下去，parseRequest 方法获得请求行的方法，URI 和协议：
String method =
new String(requestLine.method, 0, requestLine.methodEnd);
String uri = null;
String protocol = new String(requestLine.protocol, 0, requestLine.protocolEnd);
不过，在 URI 后面可以有查询字符串，假如存在的话，查询字符串会被一个问好分隔开来。
因此，parseRequest 方法试图首先获取查询字符串。并调用 setQueryString 方法来填充
HttpRequest 对象：
// Parse any query parameters out of the request URI
int question = requestLine.indexOf("?");
if (question >= 0) { // there is a query string.
request.setQueryString(new String(requestLine.uri, question + 1,
requestLine.uriEnd - question - 1));
uri = new String(requestLine.uri, 0, question);
}
else {
request.setQueryString (null);
uri = new String(requestLine.uri, 0, requestLine.uriEnd);
}
不过，大多数情况下，URI 指向一个相对资源，URI 还可以是一个绝对值，就像下面所示：
http://www.brainysoftware.com/index.html?name=Tarzan
parseRequest 方法同样也检查这种情况：
// Checking for an absolute URI (with the HTTP protocol)
if (!uri.startsWith("/")) {
// not starting with /, this is an absolute URI
int pos = uri.indexOf("://");
// Parsing out protocol and host name
if (pos != -1) {
pos = uri.indexOf('/', pos + 3);
if (pos == -1) {
uri = "";
}
else {
uri = uri.substring(pos);
}
}
}
然后，查询字符串也可以包含一个会话标识符，用 jsessionid 参数名来指代。因此，
parseRequest 方法也检查一个会话标识符。假如在查询字符串里边找到 jessionid，方法就取得
会话标识符，并通过调用 setRequestedSessionId 方法把值交给 HttpRequest 实例：
// Parse any requested session ID out of the request URI
String match = ";jsessionid=";
int semicolon = uri.indexOf(match);
if (semicolon >= 0) {
String rest = uri.substring(semicolon + match.length());
int semicolon2 = rest.indexOf(';');
if (semicolon2 >= 0) {
request.setRequestedSessionId(rest.substring(0, semicolon2));
rest = rest.substring(semicolon2);
}
else {
request.setRequestedSessionId(rest);
rest = "";
}
request.setRequestedSessionURL (true);
uri = uri.substring(0, semicolon) + rest;
}
else {
request.setRequestedSessionId(null);
request.setRequestedSessionURL(false);
}
当 jsessionid 被找到，也意味着会话标识符是携带在查询字符串里边，而不是在 cookie里边。因此，传递 true 给 request 的 setRequestSessionURL 方法。否则，传递 false 给setRequestSessionURL 方法并传递 null 给 setRequestedSessionURL 方法。到这个时候，uri 的值已经被去掉了 jsessionid。

接下去，parseRequest 方法传递 uri 给 normalize 方法，用于纠正“异常”的 URI。例如，任何\的出现都会给/替代。假如 uri 是正确的格式或者异常可以给纠正的话，normalize 将会返回相同的或者被纠正后的 URI。假如 URI 不能纠正的话，它将会给认为是非法的并且通常会返回null。在这种情况下(通常返回 null)，parseRequest 将会在方法的最后抛出一个异常。最后，parseRequest 方法设置了 HttpRequest 的一些属性：
((HttpRequest) request).setMethod(method);
request.setProtocol(protocol);
if (normalizedUri != null) {
((HttpRequest) request).setRequestURI(normalizedUri);
}
else {
((HttpRequest) request).setRequestURI(uri);
}
还有，假如 normalize 方法的返回值是 null 的话，方法将会抛出一个异常：
if (normalizedUri == null) {
 throw new ServletException("Invalid URI: " + uri + "'");
}
**解析头部**
一个 HTTP 头部是用类 HttpHeader 来代表的。这个类将会在第 4 章详细解释，而现在知道下面的内容就足够了：
你可以通过使用类的无参数构造方法构造一个 HttpHeader 实例。
 一旦你拥有一个HttpHeader实例，你可以把它传递给SocketInputStream的readHeader
方法。假如这里有头部需要读取，readHeader 方法将会相应的填充 HttpHeader 对象。
假如再也没有头部需要读取了，HttpHeader实例的nameEnd和valueEnd字段将会置零。
 为了获取头部的名称和值，使用下面的方法：
 String name = new String(header.name, 0, header.nameEnd);
 String value = new String(header.value, 0, header.valueEnd);
parseHeaders 方法包括一个 while 循环用于持续的从 SocketInputStream 中读取头部，直
到再也没有头部出现为止。循环从构建一个 HttpHeader 对象开始，并把它传递给类
SocketInputStream 的 readHeader 方法：
HttpHeader header = new HttpHeader();
// Read the next header
input.readHeader(header);
然后，你可以通过检测 HttpHeader 实例的 nameEnd 和 valueEnd 字段来测试是否可以从输入
流中读取下一个头部信息：
if (header.nameEnd == 0) {
if (header.valueEnd == 0) {
return;
}
else {
throw new
ServletException (sm.getString("httpProcessor.parseHeaders.colon"));
}
}
假如存在下一个头部，那么头部的名称和值可以通过下面方法进行检索：
String name = new String(header.name, 0, header.nameEnd);
String value = new String(header.value, 0, header.valueEnd);
一旦你获取到头部的名称和值，你通过调用 HttpRequest 对象的 addHeader 方法来把它加入
headers 这个 HashMap 中：
request.addHeader(name, value);
一些头部也需要某些属性的设置。例如，当 servlet 调用 javax.servlet.ServletRequest
的getContentLength方法的时候，content-length头部的值将被返回。而包含cookies的cookie
头部将会给添加到 cookie 集合中。就这样，下面是其中一些过程：
if (name.equals("cookie")) {
... // process cookies here
}
else if (name.equals("content-length")) {
int n = -1;
try {
n = Integer.parseInt (value);
}
catch (Exception e) {
throw new ServletException(sm.getString(
"httpProcessor.parseHeaders.contentLength"));
}
request.setContentLength(n);
}
else if (name.equals("content-type")) {
request.setContentType(value);
}
Cookie 的解析将会在下一节“解析 Cookies”中讨论。
解析 Cookies
Cookies 是作为一个 Http 请求头部通过浏览器来发送的。这样一个头部名为"cookie"并且
它的值是一些 cookie 名/值对。这里是一个包括两个 cookie:username 和 password 的 cookie
头部的例子。
Cookie: userName=budi; password=pwd;
Cookie 的解析是通过类 org.apache.catalina.util.RequestUtil的 parseCookieHeader 方
法来处理的。这个方法接受 cookie 头部并返回一个 javax.servlet.http.Cookie 数组。数组内
的元素数量和头部里边的cookie名/值对个数是一样的。parseCookieHeader方法在Listing 3.5
中列出。
Listing 3.5: The org.apache.catalina.util.RequestUtil class's parseCookieHeader method
public static Cookie[] parseCookieHeader(String header) {
if ((header == null) || (header.length 0 < 1) )
return (new Cookie[0]);
ArrayList cookies = new ArrayList();
while (header.length() > 0) {
int semicolon = header.indexOf(';');
if (semicolon < 0)
semicolon = header.length();
if (semicolon == 0)
break;
String token = header.substring(0, semicolon);
 if (semicolon < header.length())
header = header.substring(semicolon + 1);
else
header = "";
try {
int equals = token.indexOf('=');
if (equals > 0) {
String name = token.substring(0, equals).trim();
String value = token.substring(equals+1).trim();
cookies.add(new Cookie(name, value));
}
}
catch (Throwable e) {
;
}
}
return ((Cookie[]) cookies.toArray (new Cookie [cookies.size ()]));
}
还有，这里是 HttpProcessor 类的 parseHeader 方法中用于处理 cookie 的部分代码:
else if (header.equals(DefaultHeaders.COOKIE_NAME)) {
Cookie cookies[] = RequestUtil.ParseCookieHeader (value);
for (int i = 0; i < cookies.length; i++) {
if (cookies[i].getName().equals("jsessionid")) {
// Override anything requested in the URL
if (!request.isRequestedSessionIdFromCookie()) {
// Accept only the first session id cookie
request.setRequestedSessionId(cookies[i].getValue());
request.setRequestedSessionCookie(true);
request.setRequestedSessionURL(false);
}
}
request.addCookie(cookies[i]);
}
}
**获取参数**
你不需要马上解析查询字符串或者 HTTP 请求内容，直到 servlet 需要通过调用javax.servlet.http.HttpServletRequest 的 getParameter,getParameterMap, getParameterNames 或者 getParameterValues 方法来读取参数。因此，HttpRequest 的这四个方法开头调用了 parseParameter 方法。
这些参数只需要解析一次就够了，因为假如参数在请求内容里边被找到的话，参数解析将会
使得 SocketInputStream 到达字节流的尾部。类 HttpRequest 使用一个布尔变量 parsed 来指示是否已经解析过了。
参数可以在查询字符串或者请求内容里边找到。假如用户使用GET方法来请求servlet的话，
所有的参数将在查询字符串里边出现。假如使用 POST 方法的话，你也可以在请求内容中找到一
些。所有的名/值对将会存储在一个 HashMap 里边。Servlet 程序员可以以 Map 的形式获得参数(通
过调用 HttpServletRequest 的 getParameterMap 方法)和参数名/值。There is a catch, though.
Servlet 程 序 员 不 被 允 许 修 改 参 数 值 。 因 此 ， 将 使 用 一 个 特 殊 的 HashMap ：
org.apache.catalina.util.ParameterMap。
类 ParameterMap 继承 java.util.HashMap，并使用了一个布尔变量 locked。当 locked 是
false 的时候，名/值对仅仅可以添加，更新或者移除。否则，异常 IllegalStateException 会
抛出。而随时都可以读取参数值。
类 ParameterMap 将会在 Listing 3.6 中列出。它覆盖了方法用于增加，更新和移除值。那些方
法仅仅在 locked 为 false 的时候可以调用。
Listing 3.6: The org.apache.Catalina.util.ParameterMap class.
package org.apache.catalina.util;
import java.util.HashMap;
import java.util.Map;
public final class ParameterMap extends HashMap {
public ParameterMap() {
super ();
}
public ParameterMap(int initialCapacity) {
super(initialCapacity);
}
public ParameterMap(int initialCapacity, float loadFactor) {
super(initialCapacity, loadFactor);
}
public ParameterMap(Map map) {
super(map);
}
private boolean locked = false;
public boolean isLocked() {
return (this.locked);
}
public void setLocked(boolean locked) {
this.locked = locked;
}
private static final StringManager sm =
StringManager.getManager("org.apache.catalina.util");
public void clear() {
if (locked)
throw new IllegalStateException
(sm.getString("parameterMap.locked"));
super.clear();
}
 public Object put(Object key, Object value) {
if (locked)
throw new IllegalStateException
(sm.getString("parameterMap.locked"));
return (super.put(key, value));
}
public void putAll(Map map) {
if (locked)
throw new IllegalStateException
(sm.getString("parameterMap.locked"));
super.putAll(map);
}
public Object remove(Object key) {
if (locked)
throw new IllegalStateException
(sm.getString("parameterMap.locked"));
return (super.remove(key));
}
}
现在，让我们来看 parseParameters 方法是怎么工作的。
因为参数可以存在于查询字符串或者 HTTP 请求内容中，所以 parseParameters 方法会检查
查询字符串和请求内容。一旦解析过后，参数将会在对象变量 parameters 中找到，所以方法的
开头会检查 parsed 布尔变量，假如已经解析过的话，parsed 将会返回 true。
if (parsed)
return;
然后，parseParameters 方法创建一个名为 results 的 ParameterMap 变量，并指向
parameters。假如
parameters 为 null 的话，它将创建一个新的 ParameterMap。
ParameterMap results = parameters;
if (results == null)
results = new ParameterMap();
然后，parseParameters 方法打开 parameterMap 的锁以便写值。
results.setLocked(false);
下一步，parseParameters 方法检查字符编码，并在字符编码为 null 的时候赋予默认字符
编码。
String encoding = getCharacterEncoding();
if (encoding == null)
encoding = "ISO-8859-1";
然 后 ， parseParameters 方 法 尝 试 解 析 查 询 字 符 串 。 解 析 参 数 是 使 用
org.apache.Catalina.util.RequestUtil 的 parseParameters 方法来处理的。
// Parse any parameters specified in the query string
String queryString = getQueryString();
try {
RequestUtil.parseParameters(results, queryString, encoding);
}
catch (UnsupportedEncodingException e) {
;
}
接下来，方法尝试查看 HTTP 请求内容是否包含参数。这种情况发生在当用户使用 POST 方法
发送请求的时候，内容长度大于零，并且内容类型是 application/x-www-form-urlencoded 的时
候。所以，这里是解析请求内容的代码：
// Parse any parameters specified in the input stream
String contentType = getContentType();
if (contentType == null)
contentType = "";
int semicolon = contentType.indexOf(';');
if (semicolon >= 0) {
contentType = contentType.substring (0, semicolon).trim();
}
else {
contentType = contentType.trim();
}
if ("POST".equals(getMethod()) && (getContentLength() > 0)
&& "application/x-www-form-urlencoded".equals(contentType)) {
try {
int max = getContentLength();
int len = 0;
byte buf[] = new byte[getContentLength()];
ServletInputStream is = getInputStream();
while (len < max) {
int next = is.read(buf, len, max - len);
if (next < 0 ) {
break;
}
len += next;
}
is.close();
if (len < max) {
throw new RuntimeException("Content length mismatch");
}
RequestUtil.parseParameters(results, buf, encoding);
}
 catch (UnsupportedEncodingException ue) {
;
}
catch (IOException e) {
throw new RuntimeException("Content read fail");
}
}
最后，parseParameters 方法锁定 ParameterMap，设置 parsed 为 true，并把 results 赋予
parameters。
// Store the final results
results.setLocked(true);
parsed = true;
parameters = results;