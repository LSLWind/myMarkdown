### 架构

 ![img](https://pic2.zhimg.com/v2-1b87e3d9b3b86490febe3a6f91c852b1_b.jpg) 

Tomcat包含一个Server,代表整个服务器，main()从中执行，可开启多个Service

Service包含多个Connector与一个Container，附带Loging，Session等

* Connector用于处理连接相关的事情，并提供Socket与Request和Response相关的转化
* Container用于封装和管理Servlet，以及具体处理Request请求； 

 ![img](https://pic4.zhimg.com/v2-a87983f46acc0b8219827fbd3963b50f_b.jpg) 

一个请求发送到Tomcat之后，首先经过Service然后会交给Connector，Connector用于接收请求并将接收的请求封装为Request和Response来具体处理，Request和Response封装完之后再交由Container进行处理，Container处理完请求之后再返回给Connector，最后在由Connector通过Socket将处理的结果返回给客户端。

Connector最底层使用的是Socket来进行连接的，Request和Response是按照HTTP协议来封装的，所以Connector同时需要实现TCP/IP协议和HTTP协议。

#### Connector

 Connector用于接受请求并将请求封装成Request和Response，然后交给Container进行处理，Container处理完之后在交给Connector返回给客户端。 

 ![img](https://pic2.zhimg.com/v2-2dcdac0a54b22ae6ffb73b421ebe2dd5_b.jpg) 

Connector使用ProtocolHandler来处理请求的，不同的ProtocolHandler代表不同的连接类型，比如：Http11Protocol使用的是普通Socket来连接的，Http11NioProtocol使用的是NioSocket来连接的。

其中ProtocolHandler由包含了三个部件：Endpoint、Processor、Adapter。

（1）Endpoint用来处理底层Socket的网络连接，Processor用于将Endpoint接收到的Socket封装成Request，Adapter用于将Request交给Container进行具体的处理。

（2）Endpoint由于是处理底层的Socket网络连接，因此Endpoint是用来实现TCP/IP协议的，而Processor用来实现HTTP协议的，Adapter将请求适配到Servlet容器进行具体的处理。

（3）Endpoint的抽象实现AbstractEndpoint里面定义的Acceptor和AsyncTimeout两个内部类和一个Handler接口。Acceptor用于监听请求，AsyncTimeout用于检查异步Request的超时，Handler用于处理接收到的Socket，在内部调用Processor进行处理。

Connector只管接收socket连接，解析TCP报文成为Http报文，根据HTTP封装成request与response交给container

#### Container

根据request与response封装管理Servlet。

 ![img](https://pic1.zhimg.com/v2-e7f2c6bacaae08078f749881fb32d208_b.jpg)  

* Engine：引擎，用来管理多个站点，一个Service最多只能有一个Engine； 
* Host：代表一个站点，也可以叫虚拟主机，通过配置Host就可以添加站点； 
* Context：代表一个应用程序，对应着平时开发的一套程序，或者一个WEB-INF目录以及下面的web.xml文件； 
* Wrapper：每一Wrapper封装着一个Servlet， Wrapper 代表一个 Servlet，它负责管理一个 Servlet，包括的 Servlet 的装载、初始化、执行以及资源回收。 

 ![img](https://pic2.zhimg.com/v2-4b9df0f9eefdb94c8d54d61b2f7d6f21_b.jpg) 

 Context和Host的区别是Context表示一个应用，我们的Tomcat中默认的配置下webapps下的每一个文件夹目录都是一个Context，其中ROOT目录中存放着主应用，其他目录存放着子应用，而整个webapps就是一个Host站点。 

Container处理请求是使用Pipeline-Valve管道来处理的！（Valve是阀门之意）

Pipeline-Valve是责任链模式，责任链模式是指在一个请求处理的过程中有很多处理者依次对请求进行处理，每个处理者负责做自己相应的处理，处理完之后将处理后的请求返回，再让下一个处理着继续处理。

* 每个Pipeline都有特定的Valve，而且是在管道的最后一个执行，这个Valve叫做BaseValve，BaseValve是不可删除的；

* 在上层容器的管道的BaseValve中会调用下层容器的管道。

 ![img](https://pic3.zhimg.com/v2-5712e158ebb713058e63593876a2efc6_b.jpg) 

1. Connector在接收到请求后会首先调用最顶层容器的Pipeline来处理，这里的最顶层容器的Pipeline就是EnginePipeline（Engine的管道）；

2. 在Engine的管道中依次会执行EngineValve1、EngineValve2等等，最后会执行StandardEngineValve，在StandardEngineValve中会调用Host管道，然后再依次执行Host的HostValve1、HostValve2等，最后在执行StandardHostValve，然后再依次调用Context的管道和Wrapper的管道，最后执行到StandardWrapperValve。

3. 当执行到StandardWrapperValve的时候，会在StandardWrapperValve中创建FilterChain，并调用其doFilter方法来处理请求，这个FilterChain包含着我们配置的与请求相匹配的Filter和Servlet，其doFilter方法会依次调用所有的Filter的doFilter方法和Servlet的service方法，这样请求就得到了处理！