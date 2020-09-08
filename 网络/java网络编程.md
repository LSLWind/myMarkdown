## Socket

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

## HTTP

### URLConnection

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

### 实例

```java
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

