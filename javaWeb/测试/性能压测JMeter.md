## 基础

### 线程组

启动多个线程对服务器进行压力测试

【添加】→【线程】→【线程组】

* Ramp-Up：线程全部启动的最晚时间

#### HTTP请求

配置压测目标

在线程组下【添加】→【取样器】→【HTTP请求】

**高级**

【客户端实现】选择java，勾选【KeepAlive】保持长连接以压测响应时间，减少网络建立连接的事件，keepAlive希望服务端不要断开链接，后面可以复用链接

#### 结果树

打印出服务器执行的请求响应结果

在线程组下【添加】→【监听器】→【查看结果树】

#### 聚合报告

打印出线程组执行后的服务器总体请求运行状况

在线程组下【添加】→【监听器】→【聚合报告】

* Average：平均响应时间
* Median：中位数响应时间
* 90%Line：表示有90%的请求的响应时间的最大值



资源使用

服务器拒绝连接，server端线程数上不去

Tomcat配置

```
spring-configuration-medata.json
```

下配置，可看到配置参数，以server.tomcat开头

accept-count：等待队列长度，默认100，塞满后拒绝连接

max-connections：最大可被连接数，默认10000

max-threads：最大工作线程数，默认200

min-spare-threads：最小工作线程数

默认下，请求超过200+100后拒绝处理，连接超过10000拒绝连接



配置建议

max-threads=800，4核8G(800-1000最佳)

min-spare-threads=100 





keepAlive设置

keepAliveTimeOut：多少毫秒后不响应则断开连接

maxKeepAliveRequests：多少次请求后断开链接



**定制化内嵌Tomcat**

使用WebServerConfiguration

```java
@Component
//当容器内没有TomcatEmbeddedServletContainerFactory时将该bean注入容器
public class WebServerConfiguration implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
@Override
    public void customize(ConfigurableWebServerFactory configurableWebServerFactory){
    //使用对应工厂类提供的接口定制化 tomcat connector
    ((TomcatServletWebServerFactory)configurableWebServerFactory).addConnectorCustomizers(new TomcatConnectorCustomizer() {
        @Override
        public void customize(Connector connector) {
            Http11NioProtocol protocol=(Http11NioProtocol)connector.getProtocolHandler();
            //设置30s内没有请求则断开链接
            protocol.setKeepAliveTimeout(30000);
            //设置超过10000个请求则自动断开keepAlive
            protocol.setMaxKeepAliveRequests(10000);

            //protocol.setMaxConnections();
        }
    });
    }
}
```



**Mysql的QPS**

千万级别的数据

主键查询：1-10毫秒

唯一索引：10-100毫秒

非唯一索引：100-1000毫秒

无索引：百万条数据 1000毫秒+

不命中索引就全表扫描（不可接受）

## 分布式扩展

### 横向扩展

数据库服务器，多态应用服务器

数据库服务器，内网访问，避免网络传输耗时



### nginx反向代理负载均衡

横向扩展，有多个服务器可部署jar包，访问同一个域名，调用多个ip提供服务

**使用nginx作为Web服务器**

作为Web服务器仅提供静态资源

<img src="https://i.loli.net/2020/03/23/tjAnzycQZFoWJx6.png" alt="image.png" style="zoom:67%;" />

请求静态资源的时候通过miaoshaserver/resources进行访问，nginx直接代理，从本地磁盘中读取数据并直接返回，NAS作为网络附属存储将存储服务从服务器上卸载，提供更大的容量和更低的成本

#### nginx部署配置

官网上的nginx基于源码，需要手动编译，nginx有许多模块，但需要在编译时手动加载进入，同时需要配置各种参数，方便起见，可使用OpenResty框架（打包nginx，同时已经配置好了各种参数），直接下载OpenRest编译即可。

下载OpenRest，执行.configure配置命令（会报错，因为需要安装前置条件，按OpenRest官网提示安装所需依赖即可），之后执行make完成编译，最后执行make install进行安装。

安装之后进入nginx文件，conf目录放置配置文件，html放置静态资源文件，logs放日志，sbin放置nginx可执行文件

#### 配置nginx为web服务器

执行sbin/nginx -c cong/nginx.conf执行指定配置文件启动nginx

#### nginx配置文件

需要遵循指定的配置规则

server：服务器

location：配置根url路径，指定url映射key，如/就映射为根访问路径，/resources就映射为ip/resources下的所有路由

```nginx
location /resources/{
	alias /usr/local.openresty/nginx/html/resources/;
	index idnex.html index.htm
}
```

alias指定为替换规则，当请求url映射到resources/时替换为alias后面的资源路径

修改完配置文件之后可使用sbin/nginx -s reload无缝重启

#### nginx配置反向代理服务器

**设置upstream server**

修改nginx.conf，配置upstream，nginx与后端服务器默认是短链接状态

```nginx
upstream backend_server{ #反向代理集群名称
	server 服务器ip weight=1; #weight为权重，反向代理时轮询调用的权重
    keepalive 30; #使用长链接
}
```

之后配置反向代理的映射

```nginx
location /{
    proxy_pass http://backend_server;#proxy_pass代表将请求反向代理到指定的url
    proxy_set_header Host $http_host:$proxy_port #指定Host配置，即配置访问的域名替换为指定的ip及端口
        proxy_set_header X-Real_IP $remote_addr; #设置为远程地址
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; #作为代理服务器转发请求
    proxy_http_version 1.1; #设定http协议版本为1.1，支持长连接
    proxy_set_header Connection ""; #设置长连接发送为空
}
```

**开启tomcat access log验证**

该日志记录所有请求的请求时间，响应时间，请求消耗等信息

```properties
server.tomcat.accesslog.enabled=true;
server.tomcat.accesslog.directory=/xxx; #指定日志目录
server.tomcat.accesslog.pattern=%h %l %t "%r" %s %b %D
```

%h：请求的ip、%u：远端主机的user、%t：处理时长、"%r"：打印请求方法、%s：http返回状态码、%b：请求响应大小、%D：处理请求的时长







## 数据库

只允许ip白名单进行数据库访问

数据库mysql进行mysql配置

表user配置用户

host,user,password

grant all privileges on *.* to root@"%" identified by root

授予所有权限给root用户，针对所有ip

flush privileges 刷新权限