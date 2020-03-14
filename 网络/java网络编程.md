## net包

### Socket 类

在 Java 里边，套接字指的是java.net.Socket 类。
要创建一个套接字，可以使用 Socket 类众多构造方法中的一个。其中一个接收主机名称和端口号：

```java
public Socket (java.lang.String host, int port)
```

在这里主机是指远程机器名称或者 IP 地址，端口是指远程应用的端口号。例如，要连接yahoo.com 的 80 端口：

```java
new Socket ("yahoo.com", 80);
```

一旦成功创建了一个 Socket 类的实例，可以使用它来发送和接受字节流。要发送字节流，首先必须调用Socket类的getOutputStream方法来获取一个java.io.OutputStream对象。要 发 送 文 本 到 一 个 远 程 应 用 ，要 从 返 回 的 OutputStream 对 象 中 构 造 一 个java.io.PrintWriter 对象。

要从连接的另一端接受字节流，可以调用 Socket 类的getInputStream 方法用来返回一个 java.io.InputStream 对象。

```java
//1.创建Socket，建立远程TCP连接
Socket socket = new Socket("127.0.0.1", "8080");
OutputStream os = socket.getOutputStream();
boolean autoflush = true;
//2.从Socket中获取输出流-发送
PrintWriter out = new PrintWriter(socket.getOutputStream(), autoflush);
//3.从Socket中获取输入流-读取
BufferedReader in = new BufferedReader(new InputStreamReader( socket.getInputstream() ));
// 准备发送一个Http请求
out.println("GET /index.jsp HTTP/1.1");
out.println("Host: localhost:8080");
out.println("Connection: Close");
out.println();
// 读取响应
boolean loop = true;
StringBuffer sb = new StringBuffer(8096);
while (loop) {
	if ( in.ready() ) {//ready()表示返回响应
		int i=0;
		while (i!=-1) {
			i = in.read();
			sb.append((char) i);
		}
		loop = false;
	}
	Thread.currentThread().sleep(50);
}
// 输出响应
System.out.println(sb.toString());
socket.close();
```

### ServerSocket 类
Socket 类代表一个客户端套接字，即任何时候你想连接到一个远程服务器应用的时候你构造的套接字，现在，假如你想实施一个服务器应用，例如一个 HTTP 服务器或者 FTP 服务器，你需要一种不同的做法。这是因为你的服务器必须随时待命，因为它不知道一个客户端应用什么时候会尝试去连接它。为了让你的应用能随时待命，你需要使用 java.net.ServerSocket 类。这是**服务器套接字**的实现。
ServerSocket 和 Socket 不同，服务器套接字的角色是等待来自客户端的连接请求。一旦服务器套接字获得一个连接请求，它创建一个 Socket 实例来与客户端进行通信。
要创建一个服务器套接字，需要使用 ServerSocket 类提供的四个构造方法中的一个。需要指定 IP 地址和服务器套接字将要进行监听的端口号。通常，IP 地址将会是 127.0.0.1，也就是说，服务器套接字将会监听本地机器。服务器套接字正在监听的 IP 地址被称为是绑定地址。
服务器套接字的另一个重要的属性是 backlog，这是服务器套接字开始拒绝传入的请求之前，传入的连接请求的最大队列长度。
其中一个 ServerSocket 类的构造方法如下所示:

```java
public ServerSocket(int port, int backLog, InetAddress bindingAddress);
```

对于这个构造方法，绑定地址必须是 java.net.InetAddress 的一个实例。一种构造InetAddress 对象的简单的方法是调用它的静态方法 getByName，传入一个包含主机名称的字符串，就像下面的代码一样。

```java
InetAddress.getByName("127.0.0.1");
```

下面一行代码构造了一个监听的本地机器 8080 端口的 ServerSocket，它的 backlog 为 1。

```java
new ServerSocket(8080, 1, InetAddress.getByName("127.0.0.1"));
```

可以通过调用 ServerSocket 类的 accept 方法接收连接请求。这个方法只会在有连接请求时才会返回一个 Socket 类的实例。Socket 对象接下去可以发送字节流并从客户端应用中接受字节流。

### 简单应用实例-简单Web服务器

简单Web服务器HttpServer

```java
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * 简单的Http服务器实现
 */
public class HttpServer {

    public static final String WEB_ROOT = System.getProperty("user.dir") + File.separator + "webroot";
    // shutdown command
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
            serverSocket = new ServerSocket(port, 1, InetAddress.getByName("127.0.0.1"));
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(1);
        }
        // Loop waiting for a request
        while (!shutdown) {
            Socket socket = null;
            InputStream input = null;
            OutputStream output = null;
            try {
                socket = serverSocket.accept();
                input = socket.getInputStream();
                output = socket.getOutputStream();
                // create Request object and parse
                Request request = new Request(input);
                request.parse();
                // create Response object
                Response response = new Response(output);
                response.setRequest(request);
                response.sendStaticResource();
                // Close the socket
                socket.close();
                //check if the previous URI is a shutdown command
                shutdown = request.getUri().equals(SHUTDOWN_COMMAND);
            } catch (Exception e) {
                e.printStackTrace ();
            }
        }
    }
}

```

该web 服务器能提供公共静态 final 变量 WEB_ROOT 所在的目录和它下面所有的子目录下的静态资源。要停止服务器，使用URL：http://localhost:8080/SHUTDOWN

使用方法名 await 而不是 wait 是因为 wait 方法是与线程相关的 java.lang.Object 类的一个重要方法。这个方法很简单，接收一个Socket，自定义封装输入与输出流成为一个Request与Response。

Request为

```java
import java.io.InputStream;
import java.io.IOException;
/**
 * 简单的Http Request实现，底部使用Socket
 */
public class Request {
    private InputStream input;
    private String uri;
    public Request(InputStream input) {
        this.input = input;
    }
    public void parse() {
        // Read a set of characters from the socket
        StringBuffer request = new StringBuffer(2048);
        int i;
        byte[] buffer = new byte[2048];
        try {
            i = input.read(buffer);
        } catch (IOException e) {
            e.printStackTrace();
            i = -1;
        }
        for (int j=0; j<i; j++) {
            request.append((char) buffer[j]);
        }
        System.out.print(request.toString());
        uri = parseUri(request.toString());
    }

    private String parseUri(String requestString) {
        int index1, index2;
        index1 = requestString.indexOf(' ');//找到第一个空字符
        if (index1 != -1) {
            index2 = requestString.indexOf(' ', index1 + 1);//找到第二个空字符
            if (index2 > index1)
                return requestString.substring(index1 + 1, index2);//返回url
        }
        return null;
    }
    public String getUri() {
        return uri;
    }
}
```

该Request封装从客户端Socket得来的IutputStream并提供方法返回输入流（也就是一个请求报文，此处则只是指请求的url）内容以及uri

Response为

```java
import java.io.OutputStream;
import java.io.IOException;
import java.io.FileInputStream;
import java.io.File;
/**
 * 自定义封装Response
*/
public class Response {
    private static final int BUFFER_SIZE = 1024;
    Request request;
    OutputStream output;
    public Response(OutputStream output) {
        this.output = output;
    }
    public void setRequest(Request request) {
        this.request = request;
    }
    public void sendStaticResource() throws IOException {
        byte[] bytes = new byte[BUFFER_SIZE];
        FileInputStream fis = null;
        try {
            File file = new File(HttpServer.WEB_ROOT, request.getUri());
            if (file.exists()) {
                fis = new FileInputStream(file);
                int ch = fis.read(bytes, 0, BUFFER_SIZE);
                while (ch!=-1) {
                    output.write(bytes, 0, ch);
                    ch = fis.read(bytes, 0, BUFFER_SIZE);
                }
            } else {
                // file not found
                String errorMessage = "HTTP/1.1 404 File Not Found\r\n" +
                        "Content-Type: text/html\r\n" +
                        "Content-Length: 23\r\n" +
                        "\r\n" +
                        "<h1>File Not Found</h1>";
                output.write(errorMessage.getBytes());
            }
        } catch (Exception e) {
            // thrown if cannot instantiate a File object
            System.out.println(e.toString() );
        } finally {
            if (fis!=null) fis.close();
        }
    }
}
```



### HTTP

**URLConnection**

 URLConnection 是一个抽象类，它代表应用程序和 URL 之间的通信链接。此类的实例可用于读取和写入此 URL 引用的资源。

 其子类HttpUrlConnection与HttpsUrlConnection分别代表使用http或https读写引用资源(类似于，先配置请求方式，再访问资源页面。具体的：

1. 构造一个URL对象（相当于该对象代表一个网址）
2. 如果使用https，需要先配置安全访问证书，如ssl
3. 通过在 URL 上调用 openConnection 方法创建连接对象。
4. 配置连接对象（调用各种setter方法），即配置http/https 访问参数和请求属性
5. 调用连接对象的connect方法建立tcp连接，此步只建立了tcp连接，并未发送请求
6. 调用连接对象的getInputStream()方法发送http请求，并获取响应数据。

 **注意事项：**

* URLConnection的connect()函数，实际上只是建立了一个与服务器的tcp连接，并没有实际发送http请求。
  http/https请求实际上在调用HttpURLConnection的getInputStream()这个方法后才正式发送出去。
* 在用POST方式发送URL请求时，URL请求参数的设定顺序是重中之重， 对connection对象的处理设置参数和一般请求属性和写入提交数据都必须要在connect()函数执行之前完成。对outputStream的写提交数据操作，必须要在inputStream的读操作之前。这些顺序实际上是由http请求的格式决定的。
* http请求实际上由两部分组成，一个是http头，所有关于此次http请求的配置都在http头里面定义，一个是正文content。connect()函数会根据HttpURLConnection对象的配置值生成http头部信息，因此在调用connect函数之前，就必须把所有的配置准备好。
* 在http/https头后面紧跟着的是http/https请求的正文，正文的内容是通过outputStream流写入的， 实际上outputStream不是一个网络流，充其量是个字符串流，往里面写入的东西不会立即发送到网络，而是存在于内存缓冲区中(因此会有setUseCaches()这个方法)，待outputStream流关闭时，根据输入的内容生成http正文。至此，http请求的东西已经全部准备就绪。在getInputStream()函数调用的时候，就会把准备好的http请求正式发送到服务器了，然后返回一个输入流，用于读取服务器对于此次http请求的返回信息。

2.对于https如何配置安全证书

https协议对于开发者而言其实只是多了一步证书验证的过程。这个证书正常情况下被jdk/jre/security/cacerts所管理。里面证书包含两种情况：

1、机构所颁发的被认证的证书，这种证书的网站在浏览器访问时https头显示为绿色如百度

2、个人所设定的证书，这种证书的网站在浏览器里https头显示为红色×，且需要点击信任该网站才能继续访问。而点击信任这一步的操作就是我们在java代码访问https网站时区别于http请求需要做的事情。

因此分两种情况：

第一种情况：Https网站的证书为机构所颁发的被认证的证书，这种情况下和http请求一模一样，无需做任何改变，用HttpsURLConnection或者HttpURLConnection都可以。

第二种情况：个人所设定的证书，这种证书默认不被信任，需要我们自己选择信任

  解决方案：本地设定证书 

 1.导入

```java
import java.security.cert.X509Certificate;
import javax.net.ssl.SSLSocketFactory;
import javax.net.ssl.X509TrustManager;
```

2.重写证书管理

```java
//证书管理
class MyX509TrustManager implements X509TrustManager {

    @Override
    public void checkClientTrusted(X509Certificate[] chain, String authType)
            throws CertificateException {
    }

    @Override
    public void checkServerTrusted(X509Certificate[] chain, String authType)
            throws CertificateException {
    }

    @Override
    public X509Certificate[] getAcceptedIssuers() {
        return null;
    }
}
```

3.初始化配置

```java
SSLSocketFactory ssf=null;
try{
    //创建SSLContext
    SSLContext sslContext=SSLContext.getInstance("SSL");
    TrustManager[] tm={new MyX509TrustManager()};//信任管理
    //初始化
    sslContext.init(null, tm, new java.security.SecureRandom());
    //获取SSLSocketFactory对象
    ssf=sslContext.getSocketFactory();
}catch(Exception e){
    System.out.println("SSL配置出错");
    e.printStackTrace();
}
```

4.连接对象调用setSSLSocketFactory()方法

```java
csdnConnection.setSSLSocketFactory(ssf);
```

3.demo

1. ```java
   import java.io.InputStreamReader;
   import java.net.*;
   import java.security.cert.CertificateException;
   import java.security.cert.X509Certificate;
   import java.util.ArrayList;
   import java.util.List;
   import javax.net.ssl.SSLSocketFactory;
   import javax.net.ssl.X509TrustManager;
   import javax.net.ssl.*;
   import javax.net.ssl.HttpsURLConnection;
   //实现https请求需要自定义SSL证书(https对链接使用安全证书SSL)
   public class Main {
       private static int COUNT=1000;//循环访问次数
       private static List<String> urls=new ArrayList<>();//网址
       private static int ERROR_COUNT=0;//访问出错总数
       //初始化信息
       static {
           urls.add("https://blog.csdn.net/qq_40918961/article/details/95912630");
           urls.add("https://blog.csdn.net/qq_40918961/article/details/99873716");
           urls.add("https://blog.csdn.net/qq_40918961/article/details/96607888");
           urls.add("https://blog.csdn.net/qq_40918961/article/details/96438282");
           urls.add("https://blog.csdn.net/qq_40918961/article/details/100144972");
       }
       public static void main(String[] args){
           //配置ssl管理
           SSLSocketFactory ssf=null;
           try{
               //创建SSLContext
               SSLContext sslContext=SSLContext.getInstance("SSL");
               TrustManager[] tm={new MyX509TrustManager()};//信任管理
               //初始化
               sslContext.init(null, tm, new java.security.SecureRandom());
               //获取SSLSocketFactory对象
               ssf=sslContext.getSocketFactory();
           }catch(Exception e){
               System.out.println("SSL配置出错");
               e.printStackTrace();
           }
   
           for(int i=0;i<COUNT;i++){
               for (String s:urls){
                   try {
                       if(s!=null){
                           URL csdn=new URL(s);//创建URl对象
                           HttpsURLConnection csdnConnection=(HttpsURLConnection) csdn.openConnection();//打开地址URL连接,使用https，设置请求对象格式与参数
                           //配置方法参数和请求属性
                           csdnConnection.setDoInput(true);
                           csdnConnection.setDoOutput(true);
                           csdnConnection.setUseCaches(false);
                           csdnConnection.setRequestMethod("GET");
                           csdnConnection.setSSLSocketFactory(ssf);
                           csdnConnection.setRequestProperty("User-Agent","Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:61.0) Gecko/20100101 Firefox/61.0");
                           csdnConnection.connect();//建立tcp连接
                           //发送请求并获得响应报文，返回流对象
                           InputStreamReader in;
                           if(csdnConnection.getResponseCode()==-1){
                               in=new InputStreamReader(csdnConnection.getErrorStream());
                           }else{
                               in=new InputStreamReader(csdnConnection.getInputStream());
                           }
                           in.close();//关闭数据流
                           csdnConnection.disconnect();//断开连接
                       }
                   }catch (Exception e){
                       ERROR_COUNT++;
                       e.printStackTrace();
                   }
                }
               System.out.println("已执行"+(i+1)+"次");
               //防止频繁执行，睡眠30秒
               try{
                   Thread.currentThread().sleep(30000);
               }catch (Exception e){
                   System.out.println("睡眠失败");
               }
           }
           System.out.println("over,失败次数:"+ERROR_COUNT+"成功访问次数:"+(urls.size()*COUNT-ERROR_COUNT));
       }
   }
   
   //证书管理
   class MyX509TrustManager implements X509TrustManager {
   
       @Override
       public void checkClientTrusted(X509Certificate[] chain, String authType)
               throws CertificateException {
       }
   
       @Override
       public void checkServerTrusted(X509Certificate[] chain, String authType)
               throws CertificateException {
       }
   
       @Override
       public X509Certificate[] getAcceptedIssuers() {
           return null;
       }
   }
   ```

### 阻塞I/O

```java
ServerSocket serverSocket = new ServerSocket(portNumber);//创建一个新的ServerSocket，用以监听指定端口上的连接请求
Socket clientSocket = serverSocket.accept();//❶ 对accept()方法的调用将被阻塞，直到一个连接建立
BufferedReader in = new BufferedReader(   
    new InputStreamReader(clientSocket.getInputStream()));
PrintWriter out =
    new PrintWriter(clientSocket.getOutputStream(), true);//❷这些流对象都派生于该套接字的流对象
String request, response;
while ((request = in.readLine()) != null) {//❸ 处理循环开始
    if ("Done".equals(request)) {
        break;   //如果客户端发送了“Done”，则退出处理循环
    }
    response = processRequest(request); //❹ 请求被传递给服务器的处理方法
    out.println(response);//服务器的响应被发送给了客户端
}  //继续执行处理循环
```

- `ServerSocket`上的`accept()`方法将会一直阻塞到一个连接建立❶，随后返回一个新的`Socket`用于客户端和服务器之间的通信。该`ServerSocket`将继续监听传入的连接。
- `BufferedReader`和`PrintWriter`都衍生自`Socket`的输入输出流❷。前者从一个字符输入流中读取文本，后者打印对象的格式化的表示到文本输出流。
- `readLine()`方法将会阻塞，直到在❸处一个由换行符或者回车符结尾的字符串被读取。
- 客户端的请求已经被处理❹。

这段代码片段将只能同时处理一个连接，要管理多个并发客户端，需要为每个新的客户端`Socket`创建一个新的`Thread`

[<img src="https://s2.ax1x.com/2020/02/15/1zRPN6.png" alt="1zRPN6.png" style="zoom:50%;" />](https://imgchr.com/i/1zRPN6)

阻塞式调用很耗资源，效率不高

### NIO

`java.nio.channels.Selector`是Java的非阻塞I/O实现的关键。它使用了事件通知API以确定在一组非阻塞套接字中有哪些已经就绪能够进行I/O相关的操作。因为可以在任何的时间检查任意的读操作或者写操作的完成状态，使一个单一的线程便可以处理多个并发的连接。

## HttpClient

### 基本使用

HttpClient是Apache Jakarta Common下的子项目，用来提供高效的、最新的、功能丰富的支持HTTP协议的客户端编程工具包。

使用HttpClient发送请求、接收响应很简单，一般需要如下几步即可。

1. 创建HttpClient对象。
2. 创建请求方法的实例，并指定请求URL。如果需要发送GET请求，创建HttpGet对象；如果需要发送POST请求，创建HttpPost对象。
3. 如果需要发送请求参数，可调用HttpGet、HttpPost共同的setParams(HetpParams params)方法来添加请求参数；对于HttpPost对象而言，也可调用setEntity(HttpEntity entity)方法来设置请求参数。
4. 调用HttpClient对象的execute(HttpUriRequest request)发送请求，该方法返回一个HttpResponse。
5. 调用HttpResponse的getAllHeaders()、getHeaders(String name)等方法可获取服务器的响应头；调用HttpResponse的getEntity()方法可获取HttpEntity对象，该对象包装了服务器的响应内容。程序可通过该对象获取服务器的响应内容。
6. 释放连接。无论执行方法是否成功，都必须释放连接

**引入依赖**

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.11</version>
</dependency>
```



```java
/**
 *普通的GET请求
 */
public class DoGET {
    public static void main(String[] args) throws Exception {
        // 创建Httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
        // 创建http GET请求
        HttpGet httpGet = new HttpGet("http://www.baidu.com");
        CloseableHttpResponse response = null;
        try {s
            // 执行请求
            response = httpclient.execute(httpGet);
            // 判断返回状态是否为200
            if (response.getStatusLine().getStatusCode() == 200) {
                //请求体内容
                String content = EntityUtils.toString(response.getEntity(), "UTF-8");
                //内容写入文件
                FileUtils.writeStringToFile(new File("E:\\devtest\\baidu.html"), content, "UTF-8");
                System.out.println("内容长度："+content.length());
            }
        } finally {
            if (response != null) {
                response.close();
            }
            //相当于关闭浏览器
            httpclient.close();
        }
    }
}
```



```java
import java.io.File;
import java.net.URI;
import org.apache.commons.io.FileUtils;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
/**
 * 带参数的GET请求
 * 两种方式：
 *       1.直接将参数拼接到url后面 如：?wd=java
 *       2.使用URI的方法设置参数 setParameter("wd", "java")
 */
public class DoGETParam {
    public static void main(String[] args) throws Exception {
        // 创建Httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
        // 定义请求的参数
        URI uri = new URIBuilder("http://www.baidu.com/s").setParameter("wd", "java").build();
        // 创建http GET请求
        HttpGet httpGet = new HttpGet(uri);
        //response 对象
        CloseableHttpResponse response = null;
        try {
            // 执行http get请求
            response = httpclient.execute(httpGet);
            // 判断返回状态是否为200
            if (response.getStatusLine().getStatusCode() == 200) {
                String content = EntityUtils.toString(response.getEntity(), "UTF-8");
                //内容写入文件
                FileUtils.writeStringToFile(new File("E:\\devtest\\baidu-param.html"), content, "UTF-8");
                System.out.println("内容长度："+content.length());
            }
        } finally {
            if (response != null) {
                response.close();
            }
            httpclient.close();
        }
    }
}
```



```java
import org.apache.commons.io.FileUtils;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.io.File;

/**
 * 常规post请求
 *    可以设置Header来伪装浏览器请求
 */
public class DoPOST {
    public static void main(String[] args) throws Exception {
        // 创建Httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
        // 创建http POST请求
        HttpPost httpPost = new HttpPost("http://www.oschina.net/");
        //伪装浏览器请求
        httpPost.setHeader("User-Agent", "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36");
        CloseableHttpResponse response = null;
        try {
            // 执行请求
            response = httpclient.execute(httpPost);
            // 判断返回状态是否为200
            if (response.getStatusLine().getStatusCode() == 200) {
                String content = EntityUtils.toString(response.getEntity(), "UTF-8");
                //内容写入文件
                FileUtils.writeStringToFile(new File("E:\\devtest\\oschina.html"), content, "UTF-8");
                System.out.println("内容长度："+content.length());
            }
        } finally {
            if (response != null) {
                response.close();
            }
            httpclient.close();
        }
    }
}
```



```java
import java.io.File;
import java.util.ArrayList;
import java.util.List;

import org.apache.commons.io.FileUtils;
import org.apache.http.NameValuePair;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;

/**
 * 带有参数的Post请求
 * NameValuePair
 */
public class DoPOSTParam {
    public static void main(String[] args) throws Exception {
        // 创建Httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
        // 创建http POST请求
        HttpPost httpPost = new HttpPost("http://www.oschina.net/search");
        // 设置2个post参数，一个是scope、一个是q
        List<NameValuePair> parameters = new ArrayList<NameValuePair>(0);
        parameters.add(new BasicNameValuePair("scope", "project"));
        parameters.add(new BasicNameValuePair("q", "java"));
        // 构造一个form表单式的实体
        UrlEncodedFormEntity formEntity = new UrlEncodedFormEntity(parameters);
        // 将请求实体设置到httpPost对象中
        httpPost.setEntity(formEntity);
        //伪装浏览器
        httpPost.setHeader("User-Agent",
                "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36");
        CloseableHttpResponse response = null;
        try {
            // 执行请求
            response = httpclient.execute(httpPost);
            // 判断返回状态是否为200
            if (response.getStatusLine().getStatusCode() == 200) {
                String content = EntityUtils.toString(response.getEntity(), "UTF-8");
                //内容写入文件
                FileUtils.writeStringToFile(new File("E:\\devtest\\oschina-param.html"), content, "UTF-8");
                System.out.println("内容长度："+content.length());
            }
        } finally {
            if (response != null) {
                response.close();
            }
            httpclient.close();
        }
    }
}
```



### 详解

**HttpClient 的实现是线程安全的。因此建议将此类的同一个实例重用于多个请求执行。**

基本格式为

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
  <...>
} finally {
  response.close();
}
```

### Http请求

HttpClient 支持了在 HTTP / 1.1 规范中定义的所有 HTTP 方法：`GET`，`HEAD`，`POST`，`PUT`，`DELETE`，`TRACE` 和 `OPTIONS`。

对于每个方法类型，都有一个特定的类来支持：`HttpGet`， `HttpHead`，`HttpPost`， `HttpPut`，`HttpDelete`， `HttpTrace`，和 `HttpOptions`。

HttpClient 提供 `URIBuilder` 实用工具类来简化请求 URI 的创建和修改。

```bash
stdout>
http://www.google.com/search?q=httpclient&btnG=Google+Search&aq=f&oq=
```

```java
URI uri = new URIBuilder()
        .setScheme("http")
        .setHost("www.google.com")
        .setPath("/search")
        .setParameter("q", "httpclient")
        .setParameter("btnG", "Google Search")
        .setParameter("aq", "f")
        .setParameter("oq", "")
        .build();
HttpGet httpget = new HttpGet(uri);
System.out.println(httpget.getURI());
```

### HTTP 响应

HTTP 响应是在接收和解释请求消息之后由服务器发送回客户端的消息。该消息的第一行包括协议版本，后跟数字状态代码及其关联的文本短语。

```bash
 stdout >
 HTTP/1.1
 200
 OK
 HTTP/1.1 200 OK
```

```java
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1, HttpStatus.SC_OK, "OK");
System.out.println(response.getProtocolVersion());
System.out.println(response.getStatusLine().getStatusCode());
System.out.println(response.getStatusLine().getReasonPhrase());
System.out.println(response.getStatusLine().toString());
```

### 处理消息头

HTTP 消息可以包含描述消息的属性的多个标题，例如内容长度，内容类型等。HttpClient 提供了检索，添加，删除和枚举消息头的方法。

```bash
 stdout >
 Set-Cookie: c1=a; path=/; domain=localhost
 Set-Cookie: c2=b; path="/", c3=c; domain="localhost"
 2
```

```java
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,
        HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie",
        "c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie",
        "c2=b; path=\"/\", c3=c; domain=\"localhost\"");
Header h1 = response.getFirstHeader("Set-Cookie");
System.out.println(h1);
Header h2 = response.getLastHeader("Set-Cookie");
System.out.println(h2);
Header[] hs = response.getHeaders("Set-Cookie");
System.out.println(hs.length);
```

获取给定类型的所有标头的最有效的方法是使用 HeaderIterator 接口。

```bash
 stdout >
 Set-Cookie: c1=a; path=/; domain=localhost
 Set-Cookie: c2=b; path="/", c3=c; domain="localhost"
```

```java
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,
        HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie",
        "c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie",
        "c2=b; path=\"/\", c3=c; domain=\"localhost\"");

HeaderIterator it = response.headerIterator("Set-Cookie");

while (it.hasNext()) {
    System.out.println(it.next());
}
```

它还提供了便捷的方法来将 HTTP 消息解析为单独的头元素。

```bash
 stdout >
 c1 = a
 path=/
 domain=localhost
 c2 = b
 path=/
 c3 = c
 domain=localhost
```

```java
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,
        HttpStatus.SC_OK, "OK");
response.addHeader("Set-Cookie","c1=a;path=/;domain=localhost");
response.addHeader("Set-Cookie","c2=b;path=\"/\",c3=c;domain=\"localhost\"");

HeaderElementIterator heit = new BasicHeaderElementIterator(
        response.headerIterator("Set-Cookie"));

while (heit.hasNext()) {
    HeaderElement element = heit.nextElement();
    System.out.println(element.getName() + "=" + element.getValue());
    NameValuePair[] params = element.getParameters();
    for (int i = 0; i < params.length; i++) {
        System.out.println("" + params[i]);
    }
}
```

### HTTP 实体

HTTP 消息可以携带与请求或响应相关联的内容实体。实体可以在某些请求和响应中找到，因为它们是可选的。
 使用实体的请求被称为实体封装请求。
 HTTP 规范定义了两个实体封装请求的方法：POST 和 PUT。HTTP 响应通常期望包含有内容实体。
当然也有例外的情况，例如响应为 204 No Content， 304 Not Modified，205 Reset Content 的时候不包含内容实体。
 HttpClient 根据其内容来源区分三种实体：

1. 流式传输（streamed）：内容是从流中接收到的，或者即时生成。特别地，该类别包括从 HTTP 响应接收到的实体。流式实体通常不可重复。
2. 自包含（self-contained）：内容在内存中或通过独立于连接或其他实体的方式获取。自包含的实体通常是可重复的。这种类型的实体将主要用于封闭 HTTP 请求的实体。
3. 包装（wrapping）：内容是从另一个实体获得的。

当从 HTTP 响应流出内容时，这个区别对于连接管理很重要。对于由应用程序创建并且仅使用 HttpClient 发送的请求实体，流和自包含的区别并不是重要。因而可以通过是否可重复来区分它们，将不可重复的实体视为流式的，将可重复的实体视为自包含的。

**可重复的实体**

一个实体可以重复，这意味着它的内容可以被读取多次。这种情况仅仅可能出现在自包含的实体中（例如 ByteArrayEntity 或 StringEntity）。

**使用 HTTP 实体**

由于实体可以表示**二进制**和**字符内容**，因此它支持字符编码（为了支持字符内容）。

当执行带有封闭内容的请求时，或者当请求成功后使用响应主体将结果发送回客户端时，实体被创建。

要从实体读取内容，可以通过 `HttpEntity#getContent()` 方法来检索输入流，该方法返回一个 `java.io.InputStream`，或者可以向  `HttpEntity#writeTo(OutputStream)` 方法提供输出流，一旦所有内容都被写入给定流，该方法将返回。

当实体和传入消息已经被接收，方法 `HttpEntity#getContentType()` 和
 `HttpEntity#getContentLength()` 可用于读取公共 metadata，例如 Content-Type 与 Content-Length 报头（如果可用的话）。
 Content-Type 报头可以包含 `text/plain` 或 `text/html` 等文本 MIME 类型的字符编码，方法 `HttpEntity#getContentEncoding()` 可以用于读取此信息。
 如果报头不可用，则返回 Content-Length 为 -1，Content-Type 为 NULL。如果 Content-Type 报头可用，将返回一个 Header 对象。

当为外发消息创建实体时，该 metadata 必须由实体的创建者来提供。

```bash
 stdout >
 Content-Type: text/plain; charset=utf-8
 17
 important message
 17
```

```java
StringEntity myEntity = new StringEntity("important message",
        ContentType.create("text/plain", "UTF-8"));

System.out.println(myEntity.getContentType());
System.out.println(myEntity.getContentLength());
System.out.println(EntityUtils.toString(myEntity));
System.out.println(EntityUtils.toByteArray(myEntity).length);
```

### 确保释放低级别的资源

为了确保正确释放系统资源，必须关闭与实体相关联的内容流或者响应本身。

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        InputStream instream = entity.getContent();
        try {
            //做一些有用的事情
        } finally {
            instream.close();
        }
    }
} finally {
    response.close();
}
```

关闭内容流和关闭响应之间的区别是，前者将尝试通过消费实体内容来保持底层的连接，而后者会立即关闭并丢弃当前连接。

请注意，一旦方法 `HttpEntity#writeTo(OutputStream)` 的实体完全写出，也需要确保合适的释放系统资源。如果这个方法通过调用 `HttpEntity#getContent()` 获取一个 `java.io.InputStream` 实例，那么也应该在 finally 子句中关闭流。

当处理流实体时，可以使用 `EntityUtils#consume(HttpEntity)` 方法来确保实体内容已被完全消耗，并且底层流已经被关闭。

然而，可能会有一种情况，当仅需要检索整个响应内容的一小部分时，消耗剩余内容并使连接可重用的性能损失太高，这时可以通过关闭响应来终止内容流。

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        InputStream instream = entity.getContent();
        int byteOne = instream.read();
        int byteTwo = instream.read();
        //不需要休息
    }
} finally {
    response.close();
}
```

连接不会被重用，但由其持有的所有级别的资源将会被正确地释放。

### 消费实体内容

消费实体内容推荐使用 `HttpEntity#getContent()` 或 `HttpEntity#writeTo(OutputStream)` 方法。

HttpClient 还附带了 `EntityUtils` 类，它暴露了几种静态方法，来方便我们从实体中读取内容或信息。可以通过使用这个类的方法取回整个内容体的字符串或者字节数组，而不是通过去直接读取 `java.io.InputStream` 来获取内容。但是，**除非响应实体来自受信任的 HTTP 服务器，并且已知其长度有限，否则强烈建议不要使用 `EntityUtils` 的方法**。

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        long len = entity.getContentLength();
        if (len != -1 && len < 2048) {
            System.out.println(EntityUtils.toString(entity));
        } else {
            //流内容
        }
    }
} finally {
    response.close();
}
```

在某些情况下，可能需要多次读取实体内容。在这种情况下，实体内容必须以某种方式缓存，无论是在内存还是在磁盘上。

最简单的方式是通过使用 `BufferedHttpEntity` 类包装原始实体。这会使原始实体的内容被读入内存缓冲区。在所有其他方式中，实体包装器将具有原始内容。

```java
CloseableHttpResponse response = <...>
HttpEntity entity = response.getEntity();
if (entity != null) {
    entity = new BufferedHttpEntity(entity);
}
```

### 创建实体内容

HttpClient 提供了几个类，可以通过 HTTP 连接高效地流出内容。这些类的实例可以将那些需要包装其关联的实体内容的 HTTP 请求（例如 POST 和 PUT）包装实体去流出 HTTP 请求端。HttpClient 为最常见的数据的容器（如字符串，字节数组，输入流和文件）提供了几个类：`StringEntity`，`ByteArrayEntity`，`InputStreamEntity` 和 `FileEntity`。

请注意 InputStreamEntity 不可重复，因为它只能从基础数据流中读取一次。通常建议实现一个自包含 HttpEntity 的自定义类，而不是使用一般的 InputStreamEntity。 FileEntity 可以是一个很好的开始。

```java
File file = new File("somefile.txt");
FileEntity entity = new FileEntity(file,
        ContentType.create("text/plain", "UTF-8"));

HttpPost httppost = new HttpPost("http://localhost/action.do");
httppost.setEntity(entity);
```

### HTML 表单

许多应用程序需要模拟提交 HTML 表单的过程，例如，登录到 Web 应用程序或提交输入数据。HttpClient 提供实体类 `UrlEncodedFormEntity` 来简化这个过程。
 本 `UrlEncodedFormEntity` 实例将使用所谓的 URL 编码参数进行编码并生成以下内容：

```txt
param1=value1&param2=value2
```

```java
List<NameValuePair> formparams = new ArrayList<NameValuePair>();
formparams.add(new BasicNameValuePair("param1", "value1"));
formparams.add(new BasicNameValuePair("param2", "value2"));
UrlEncodedFormEntity entity = new UrlEncodedFormEntity(formparams, Consts.UTF_8);
HttpPost httppost = new HttpPost("http://localhost/handler.do");
httppost.setEntity(entity);
```

### 内容分块

一般建议让 HttpClient 根据正在传输的 HTTP 消息的属性选择最合适的传输编码。然而，可以通知 HttpClient，通过设置 `HttpEntity#setChunked()` 为 true，优先选择块编码。

请注意，HttpClient 只会使用此标志作为提示。当使用不支持块编码的 HTTP 协议版本（如 HTTP/1.0）时，此值将被忽略。

```java
StringEntity entity = new StringEntity("important message",
        ContentType.create("plain/text", Consts.UTF_8));
entity.setChunked(true);
HttpPost httppost = new HttpPost("http://localhost/acrtion.do");
httppost.setEntity(entity);
```

### 响应处理程序

处理响应的最简单和最方便的方法是使用 ResponseHandler 接口，它包含 `handleResponse(HttpResponse response)` 方法。该方法完全消除用户对连接管理的担心。

使用 `ResponseHandler` 时，无论请求执行成功还是出现异常，HttpClient 都将自动保证将该连接释放回连接管理器。

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/json");

ResponseHandler<MyJsonObject> rh = new ResponseHandler<MyJsonObject>() {

    @Override
    public JsonObject handleResponse(final HttpResponse response) throws IOException {
        StatusLine statusLine = response.getStatusLine();
        HttpEntity entity = response.getEntity();
        if (statusLine.getStatusCode() >= 300) {
            throw new HttpResponseException(
                    statusLine.getStatusCode(),
                    statusLine.getReasonPhrase());
        }
        if (entity == null) {
            throw new ClientProtocolException("Response contains no content");
        }
        Gson gson = new GsonBuilder().create();
        ContentType contentType = ContentType.getOrDefault(entity);
        Charset charset = contentType.getCharset();
        Reader reader = new InputStreamReader(entity.getContent(), charset);
        return gson.fromJson(reader, MyJsonObject.class);
    }
};
MyJsonObject myjson = client.execute(httpget, rh);
```

### HttpClient 接口

HttpClient 接口代表 HTTP 请求执行最基本的协议。它对请求执行过程没有施加任何限制或特定细节，并将连接管理，状态管理，认证和重定向处理的具体细节留给一个具体实现。这使得可以更容易使用附加功能（如响应内容缓存）来装饰接口。

一般来说，HttpClient 实现作为一个特殊目的处理程序或策略接口实现的立面，负责处理 HTTP 协议的特定方面，如重定向或身份验证处理或决定连接持久性并保持活动持续时间。这使得用户能够选择性地将这些方面的默认实现替换为自己的实现或者为特定应用程序设计的实现。

```java
ConnectionKeepAliveStrategy keepAliveStrat = new DefaultConnectionKeepAliveStrategy() {

    @Override
    public long getKeepAliveDuration(
            HttpResponse response,
            HttpContext context) {
        long keepAlive = super.getKeepAliveDuration(response, context);
        if (keepAlive == -1) {
            // 如果 keep-alive 的值没有被服务器显式设置，就设置为 5 秒
            keepAlive = 5000;
        }
        return keepAlive;
    }
};

CloseableHttpClient httpclient = HttpClients.custom()
        .setKeepAliveStrategy(keepAliveStrat)
        .build();
```

### HTTP 执行上下文

最初 HTTP 被设计为无状态的，请求-响应的协议。然而，现实世界的应用程序通常需要通过几个逻辑相关的请求-响应交换来保持状态信息。

为了使应用程序能够保持处理状态，HttpClient 允许在特定执行上下文（称为 HTTP context）中执行 HTTP 请求。

如果在连续请求之间重复使用相同的上下文，则多个逻辑相关请求可以参与逻辑会话。HTTP 上下文功能类似于一个 `java.util.Map`。它只是一个任意命名值的集合。应用程序可以在请求执行之前填充上下文属性，或者在执行完成后检查上下文。

HttpContext 可以包含任意对象，因此可能不安全地在多个线程之间共享。建议每个执行线程维护自己的上下文。

在 HTTP 请求执行过程中，HttpClient 将以下属性添加到执行上下文中：

- `HttpConnection` 表示与目标服务器的实际连接的实例。
- `HttpHost` 表示连接目标的实例。
- `HttpRoute` 表示完整的连接路由的实例
- `HttpRequest` 表示实际 HTTP 请求的实例。执行上下文中的最终 HttpRequest 对象总是表示消息的状态与发送到目标服务器的状态完全相同。默认 HTTP / 1.0 和 HTTP / 1.1 使用相对请求 URI。然而，如果请求是通过非隧道模式下的代理发送的，则 URI 将是绝对地址。
- `HttpResponse` 表示实际的 HTTP 响应。
- `java.lang.Boolean` 表示实际请求是否被完全发送到连接目标的标志的对象。
- `RequestConfig` 表示实际请求配置的对象。
- `java.util.List` 表示在请求执行过程中接收到的所有重定向位置的集合的对象。

可以使用 HttpClientContext 适配器类来简化与上下文状态的分离。

```java
HttpContext context = <...>
HttpClientContext clientContext = HttpClientContext.adapt(context);
HttpHost target = clientContext.getTargetHost();
HttpRequest request = clientContext.getRequest();
HttpResponse response = clientContext.getResponse();
RequestConfig config = clientContext.getRequestConfig();
```

表示逻辑相关会话的多个请求序列应该使用相同的 HttpContext 实例执行，以确保在请求之间自动传播会话上下文和状态信息。

在以下示例中，由初始请求设置的请求配置将保留在执行上下文中，并被传播到共享相同上下文的连续请求。

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
RequestConfig requestConfig = RequestConfig.custom()
        .setSocketTimeout(1000)
        .setConnectTimeout(1000)
        .build();

HttpGet httpget1 = new HttpGet("http://localhost/1");
httpget1.setConfig(requestConfig);
CloseableHttpResponse response1 = httpclient.execute(httpget1, context);
try {
    HttpEntity entity1 = response1.getEntity();
} finally {
    response1.close();
}
HttpGet httpget2 = new HttpGet("http://localhost/2");
CloseableHttpResponse response2 = httpclient.execute(httpget2, context);
try {
    HttpEntity entity2 = response2.getEntity();
} finally {
    response2.close();
}
```

### HTTP 协议拦截器

HTTP 协议拦截器是一个实现 HTTP 协议特定切面的程序。通常协议拦截器期望作用于输入消息的一个特定头部或一组相关头部，或者使用一个特定头部或一组相关头部填充传出的消息。

协议拦截器还可以操纵包含消息的内容实体 - 显式内容压缩/解压缩就是一个很好的例子。通常这是通过使用“装饰器”模式来实现的，其中包装器实体类用于装饰原始实体。几个协议拦截器可以组合形成一个逻辑单元。

协议拦截器可以通过 HTTP 执行上下文共享信息（如处理状态）进行协作。协议拦截器可以使用 HTTP 上下文来存储一个请求或多个连续请求的处理状态。通常执行拦截器的顺序并不重要，只要它们不依赖执行上下文的特定状态即可。如果协议拦截器具有相互依赖关系，那么必须以特定顺序执行，应将它们按照与其预期执行顺序相同的顺序添加到协议处理器中。

协议拦截器必须实现为线程安全。与 servlet 类似，协议拦截器不应该使用实例变量，除非对这些变量的访问是同步（synchronized）的。

下面是一个例子，说明如何使用本地上下文来持续连续请求之间的处理状态：



```java
CloseableHttpClient httpclient = HttpClients.custom()
        .addInterceptorLast(new HttpRequestInterceptor() {

            public void process(
                    final HttpRequest request,
                    final HttpContext context) throws HttpException, IOException {
                AtomicInteger count = (AtomicInteger) context.getAttribute("count");
                request.addHeader("Count", Integer.toString(count.getAndIncrement()));
            }

        })
        .build();

AtomicInteger count = new AtomicInteger(1);
HttpClientContext localContext = HttpClientContext.create();
localContext.setAttribute("count", count);

HttpGet httpget = new HttpGet("http://localhost/");
for (int i = 0; i < 10; i++) {
    CloseableHttpResponse response = httpclient.execute(httpget, localContext);
    try {
        HttpEntity entity = response.getEntity();
    } finally {
        response.close();
    }
}
```

### 异常处理

HTTP 协议处理器可以抛出两种类型的异常：`java.io.IOException` 表示 I/O 故障（例如套接字超时或套接字复位），而 `HttpException` 表示 HTTP 失败，例如违反 HTTP 协议。通常 I/O 错误被认为是非致命和可恢复的，而 HTTP 协议错误被认为是致命的，不能自动恢复。

请注意，HttpClient 实现重新抛出 HttpException 作为 ClientProtocolException，它是一个 java.io.IOException 的子类 。这使得 HttpClient 的用户能够在单个 catch 子句同时处理 I/O 错误和协议错误。

### HTTP 传输安全

重要的是要了解 HTTP 协议并不适合所有类型的应用程序。HTTP 是一种简单的面向 请求/响应 的协议，最初被设计为支持静态或动态生成的内容检索。它从来没有意图支持事务性操作。

例如：

- 如果 HTTP 服务器成功接收和处理请求，生成响应并将状态代码发送回客户端，则 HTTP 服务器将考虑其部分合同。
- 如果客户端由于读取超时，请求取消或系统崩溃而无法全部收到响应，服务器将不会尝试回滚事务。
- 如果客户端决定重试相同的请求，服务器将不可避免地最终不止一次地执行相同的事务。

在某些情况下，这可能导致应用程序数据损坏或应用程序状态不一致。

即使 HTTP 从未被设计为支持事务处理，但是如果满足某些条件，则仍然可以将其用作任务关键应用程序的传输协议。

为了确保 HTTP 传输层的安全，系统必须确保应用层上的 HTTP 方法的幂等性。

### 中止请求

在某些情况下，由于目标服务器的负载过高或在客户端发出的太多并发请求，HTTP 请求执行在预期时间内无法完成。在这种情况下，可能需要提前终止请求，并在 I/O 操作中解除阻塞执行线程。

HttpClient 执行的 HTTP 请求可以通过调用 `HttpUriRequest#abort()` 方法在执行的任何阶段中止 。该方法是线程安全的，可以从任何线程调用。
 当一个 HTTP 请求中止时，它的执行线程 - 即使当前在 I/O 操作中被阻塞 - 也可以通过抛出一个 InterruptedIOException 来疏通阻塞线程。

### 重定向处理

HttpClient 会自动处理所有类型的重定向，除了需要用户干预的 HTTP 规范明确禁止的重定向之外。

See Other（状态码 303）重定向 POST 和 PUT 请求转换为 GET 请求作为 HTTP 规范的要求。

可以使用自定义重定向策略来放对 HTTP 规范强加的 POST 方法的自动重定向的限制。

```java
LaxRedirectStrategy redirectStrategy = new LaxRedirectStrategy();
CloseableHttpClient httpclient = HttpClients.custom()
        .setRedirectStrategy(redirectStrategy)
        .build();
```

HttpClient 经常在其执行过程中重写请求消息。默认情况下，HTTP/1.0 和 HTTP/1.1 通常使用相对请求URI。

同样，原始请求可能会从一个地址重定向到另一个地址多次。最终绝对 HTTP 地址定位可以使用原始请求和上下文构建出来。

一个实用的方法 `URIUtils#resolve` 可用于构建用于生成最终 HTTP 请求绝对的 URI。该方法包括来自重定向请求或原始请求的最后一个片段标识符。

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpClientContext context = HttpClientContext.create();
HttpGet httpget = new HttpGet("http://localhost:8080/");
CloseableHttpResponse response = httpclient.execute(httpget, context);
try {
    HttpHost target = context.getTargetHost();
    List<URI> redirectLocations = context.getRedirectLocations();
    URI location = URIUtils.resolve(httpget.getURI(), target, redirectLocations);
    System.out.println("Final HTTP location: " + location.toASCIIString());
    //预期是绝对的URI
} finally {
    response.close();
}
```



