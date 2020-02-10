### 基础概念

#### WebSocket

 WebSocket并不是全新的协议，而是利用了HTTP协议来建立连接,与Http最大的不同在于WebSocket可以建立起持续的通信连接，服务器可以向客户端发送数据，常用于实时更新（如果使用Http则要进行刷新，而使用WebSocket则进行实时更新）用户页面，如商品竞价，网页聊天室，游戏等）

举个例子：

 在WebSocket概念出来之前，如果页面要不停地显示最新的价格，那么必须不停地刷新页面，或者用一段js代码每隔几秒钟发消息询问服务器数据。 

而使用WebSocket技术之后，当服务器有了新的数据，会主动通知浏览器，浏览器立马接收到消息并更新数据。

首先，WebSocket连接必须由浏览器发起，因为请求协议是一个标准的HTTP请求，格式如下：

```
GET ws://localhost:3000/ws/chat HTTP/1.1
Host: localhost
Upgrade: websocket
Connection: Upgrade
Origin: http://localhost:3000
Sec-WebSocket-Key: client-random-string
Sec-WebSocket-Version: 13
```

该请求和普通的HTTP请求有几点不同：

1. GET请求的地址不是类似`/path/`，而是以`ws://`开头的地址；
2. 请求头`Upgrade: websocket`和`Connection: Upgrade`表示这个连接将要被转换为WebSocket连接；
3. `Sec-WebSocket-Key`是用于标识这个连接，并非用于加密数据；
4. `Sec-WebSocket-Version`指定了WebSocket的协议版本。

随后，服务器如果接受该请求，就会返回如下响应：

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: server-random-string
```

该响应代码`101`表示本次连接的HTTP协议即将被更改，更改后的协议就是`Upgrade: websocket`指定的WebSocket协议。

版本号和子协议规定了双方能理解的数据格式，以及是否支持压缩等等。如果仅使用WebSocket的API，就不需要关心这些。

现在，一个WebSocket连接就建立成功，浏览器和服务器就可以随时主动发送消息给对方。消息有两种，一种是文本，一种是二进制数据。通常，我们可以发送JSON格式的文本，这样，在浏览器处理起来就十分容易。

#### 优点

* 节约带宽。 不停地轮询服务端数据这种方式，使用的是http协议，head信息很大，有效数据占比低， 而使用WebSocket方式，头信息很小，有效数据占比高。
* 无浪费。 轮询方式有可能轮询10次，才碰到服务端数据更新，那么前9次都白轮询了，因为没有拿到变化的数据。 而WebSocket是由服务器主动回发，来的都是新数据。
*  实时性，考虑到服务器压力，使用轮询方式不可能很短的时间间隔，否则服务器压力太多，所以轮询时间间隔都比较长，好几秒，设置十几秒。 而WebSocket是由服务器主动推送过来，实时性是最高的 				 

#### Ajax VS WebSocket

WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

WebSocket之前，为了实现推送技术，所用的技术都是 Ajax  轮询。轮询是在特定的的时间间隔（如每1秒），由浏览器对服务器发出HTTP请求，然后由服务器返回最新的数据给客户端的浏览器。这种传统的模式带来很明显的缺点，即浏览器需要不断的向服务器发出请求，然而HTTP请求可能包含较长的头部，其中真正有效的数据可能只是很小的一部分，显然这样会浪费很多的带宽等资源。

 ![img](https://www.runoob.com/wp-content/uploads/2016/03/ws.png) 

### 基于java使用WebSocket

#### 添加webSocket依赖

这里使用SpringBoot进行Web开发

```xml
        <!--websocket-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
```

#### 使用 @ServerEndpoint() 

@ServerEndpoint() 声明在一个类上，表明是一个WebSocket服务器，括号内需定义一个该websocket服务器端url地址

 @ServerEndpoint 是一个类层次的注解，它的功能主要是将目前的类定义成一个websocket服务器端,注解的值将被用于监听用户连接的终端访问URL地址,客户端可以通过这个URL来连接到WebSocket服务器端

#### SpringBoot使用WebSocket前端出现404

一个比较大的坑，javaEE对WebSocket进行了实现，使用SpringBoot整合WebSocket时要注意一下进行配置

首先创建websocket的config类，注入ServerEndpointExporter（使用外置tomcat就无需注入）。注册了这个类之后就会对@ServerEndpoint声明类进行注入，成websocket服务类，进行监听。

（MySpringConfigurator类是为了使WebSocketServer类能够注入其他bean（如图中RedisService的bean），因为websocket注册的bean默认是自己管理，没有托管给spring，所以，此类是为了将websocket的bean托管给spring容器。）

@Configuration
@ConditionalOnWebApplication
public class WebSocketConfig  {

    //使用boot内置tomcat时需要注入此bean
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }

 

    @Bean
    public MySpringConfigurator mySpringConfigurator() {
        return new MySpringConfigurator();
    }
}
/**
 *  以websocketConfig.java注册的bean是由自己管理的，需要使用配置托管给spring管理
 */
public class MySpringConfigurator extends ServerEndpointConfig.Configurator implements ApplicationContextAware {

    private static volatile BeanFactory context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        MySpringConfigurator.context = applicationContext;
    }

    @Override
    public <T> T getEndpointInstance(Class<T> clazz) throws InstantiationException {
        return context.getBean(clazz);
    }
}
创建websocketServer类（太长了，忽略@onopen等方法）

/*
    websocket所需注解,设置监听路径等，此注解会为每一个连接请求创建一个point实例对象，
    此注解会将编解码成一个websocket服务，url是发布路径。
 */
@Component
@ServerEndpoint(value = "/webSocket/{sid}", configurator = MySpringConfigurator.class)
public class WebSocketServer {

     @Autowired
     private RedisService redisService;
     
     private Session session;//每个客户端锁相对应的session，服务端根据session和客户端进行数据交互
    private String sid = "";
    private static CopyOnWriteArraySet<WebSocketServer> copyOnWriteArraySet
            = new CopyOnWriteArraySet<WebSocketServer>();//存储每个客户端连接
    
    ....实现@onopen等方法...

}
前端页面网上很多，这里不放了。
