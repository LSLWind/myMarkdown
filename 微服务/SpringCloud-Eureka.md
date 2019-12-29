### 基础概念

#### 微服务

单体架构的缺陷：

1. 如果要修改数据部分的代码， 那么必须把整个项目重新编译打包部署。 虽然展示部分，什么都没变但是也会因为重新部署而暂时不能使用，要部署完了，才能使用。

2. 如果提供数据部分出现了问题，比如有的开发人员改错了，抛出了异常，会导致整个项目不能使用，展示数据部分也因此受到影响。
3.  性能瓶颈难以突破 

微服务即将整个系统的服务功能模块进行拆分，使每个服务做的事情更加纯粹单纯

微服务简单说，一个 springboot 就是一个微服务，并且这个 springboot 做的事情很单纯。 比如可以将传统的单体式应用项目拆成两个微服务，分别是数据微服务和视图微服务，其实就是俩 springboot, 只是各自做的事情都更单纯。

#### 服务注册--eureka

**服务注册**

有了微服务，就存在如何管理这个微服务，以及这两个微服务之间如何通信的问题，所以就要引入一个微服务注册中心概念，这个微服务注册中心在 springcloud 里就叫做 eureka server, 通过它把就可以把微服务注册起来，以供将来调用。 				 

微服务之间存在互相调用的关系，之间的调用也需要通过注册中心调度

 在业务逻辑上， 视图微服务需要数据微服务的数据，所以就存在一个微服务访问另一个微服务的需要。 
而这俩微服务已经被注册中心管理起来了，所以视图微服务就可以通过注册中心定位并访问数据微服务了。 

在服务治理框架中，通常会构建一个注册中心，每个服务单元向注册中心登记自己提供的服务，将主机与端口号、版本号、通信协议等信息告知注册中心，注册中心维护一个服务清单，定期发送心跳检测清单中的服务是否可用。

**服务发现**

即服务间通过服务中心的互相调用，需要通过服务中心进行调度，涉及到客户端负载均衡

#### 微服务中的分布式

简单说，原来是在一个 springboot里就完成的事情，现在分布在多个springboot里做，这就是初步具备分布式雏形了。
那么分布式有什么好处呢？ 

1. 如果我要更新数据微服务，视图微服务是不受影响的
2. 可以让不同的团队开发不同的微服务，他们之间只要约定好接口，彼此之间是低耦合的。
3. 如果视图微服务挂了，数据微服务依然可以继续使用 

#### 微服务中的集群

 原来数据微服务只有这一个springboot, 现在做同样数据微服务的，有两个 springboot, 他们提供的功能一模一样，只是端口不一样，这样就形成了集群。
那么集群有什么好处呢？

1. 比起一个 springboot,  两个springboot 可以分别部署在两个不同的机器上，那么理论上来说，能够承受的负载就是 x 2. 这样系统就具备通过横向扩展而提高性能的机制。
2. 如果 8001 挂了，还有 8002 继续提供微服务，这就叫做高可用 。

#### 围绕分布式和集群的周边服务  	 

 以上是很简单的分布式结构，围绕这个结构，我们有时候还需要做如下事情：

1. 哪些微服务是如何彼此调用的？ sleuth 服务链路追踪
2. 如何在微服务间共享配置信息？配置服务 Config Server
3. 如何让配置信息在多个微服务之间自动刷新？ RabbitMQ 总线 Bus
4. 如果数据微服务集群都不能使用了， 视图微服务如何去处理?  断路器 Hystrix
5. 视图微服务的断路器什么时候开启了？什么时候关闭了？  断路器监控 Hystrix Dashboard
6. 如果视图微服务本身是个集群，那么如何进行对他们进行聚合监控？  断路器聚合监控 Turbine Hystrix Dashboard
7.  如何不暴露微服务名称，并提供服务？ Zuul 网关 

 绿色的为常用组件![img](https://img2018.cnblogs.com/blog/398358/201907/398358-20190719145209332-1168967345.png) 

### 服务注册中心--eureka

#### eureka

[Eureka](https://github.com/Netflix/Eureka) 是 [Netflix](https://github.com/Netflix) 开发的，一个基于 REST 服务的，服务注册与发现的组件

它主要包括两个组件：Eureka Server 和 Eureka Client

- Eureka Client：主要处理服务的注册与发现，客户端向注册中心注册自身提供的服务并周期性的发送心跳以更新服务租约，查询注册信息并缓存到本地然后周期性刷新服务庄状态
- Eureka Server：服务注册中心

![img](https://img2018.cnblogs.com/blog/398358/201907/398358-20190722105850485-951984065.png)

各个微服务启动时，会通过 Eureka Client 向 Eureka Server 注册自己，Eureka Server 会存储该服务的信息

也就是说，每个微服务的客户端和服务端，都会注册到 Eureka Server，这就衍生出了微服务相互识别的话题

- 同步：每个 Eureka Server 同时也是 Eureka Client（逻辑上的）
  　　　多个 Eureka Server 之间通过复制的方式完成服务注册表的同步，形成 Eureka 的高可用
- 识别：Eureka Client 会缓存 Eureka Server 中的信息
  　　　即使所有 Eureka Server 节点都宕掉，服务消费者仍可使用缓存中的信息找到服务提供者
- 续约：微服务会周期性（默认30s）地向 Eureka Server 发送心跳以Renew（续约）信息（类似于heartbeat）
- 续期：Eureka Server 会定期（默认60s）执行一次失效服务检测功能
  　　　它会检查超过一定时间（默认90s）没有Renew的微服务，发现则会注销该微服务节点

Spring Cloud 已经把 Eureka 集成在其子项目 Spring Cloud Netflix 里面

#### 创建eureka项目--Server

使用IDEA+Maven创建父子项目，父项目pom.xml中引入SpringCloud：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>lsl</groupId>
    <artifactId>maven-modulesProject</artifactId>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>childMavenProject</module>
    </modules>

    <!--包为pom，即父子聚合项目-->
    <packaging>pom</packaging>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>4.3.1</version>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>
```

子项目即使用eureka，创建服务注册中心，该子模块其实就是一个springboot项目，使用IDEA创建该子项目，选用模板时选择Spring Cloud Discover ，其中有几个选项，可以看出这是为SpringCloud服务发现提供服务，有Eureka与Zookeeper等，这里勾选Eureka相关服务，最终生成的pom.xml中会自动引入eureka-server，大致为：

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
```

#### EurekaServerApplication  	 

EurekaServerApplication是EurekaServer的启动类，名称是项目名加上一个Application，如同传统的SpringBoot项目一样：

```java
@EnableEurekaServer
@SpringBootApplication
public class SpringcloudEurekaLearnApplication {

    public static void main(String[] args) {
        //注意这行代码，用于（配置）启动eureka服务
        new SpringApplicationBuilder(SpringcloudEurekaLearnApplication.class).run(args);
    }

}
```

注意加上注解@EnableEurekaServer，表示这是一个EurekaServer

#### src/main 下的配置文件

格式可以为application.properties或者application.yml，关于eureka的主要配置都可以在这里进行更改默认选项或指定必须配置项，主要配置项有：

 hostname: localhost 表示主机名称。
registerWithEureka：false. 表示是否注册到服务器。 因为它本身就是服务器，所以就无需把自己注册到服务器了。
fetchRegistry: false. 表示是否获取服务器的注册信息，和上面同理，这里也设置为 false。
defaultZone： http://${eureka.instance.hostname}:${server.port}/eureka/   自己作为服务器，公布出来的地址。 比如后续某个微服务要把自己注册到 eureka server, 那么就要使用这个地址：  http://localhost:8761/eureka/

name: eurka-server 表示这个微服务本身的名称是 eureka-server 		

server.port：项目端口

```properties
## 端口
server.port=1111

# 设置是否将自己作为客户端注册到注册中心（缺省true）
# 这里为不需要（查看@EnableEurekaServer注解的源码，会发现它间接用到了@EnableDiscoveryClient）
eureka.client.register-with-eureka=false
# 设置是否从注册中心获取注册信息（缺省true）
# 因为这是一个单点的EurekaServer，不需要同步其它EurekaServer节点的数据，故设为false
eureka.client.fetch-registry=false
#注册中心路径，表示我们向这个注册中心注册服务，如果向多个注册中心注册，用“，”进行分隔
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
#是否开启自我保护模式，默认为true。
eureka.server.enable-self-preservation=true
#续期时间，即扫描失效服务的间隔时间（缺省为60*1000ms）
eureka.server.eviction-interval-timer-in-ms=10000
```

#### 访问eureka

使用上面的配置启动项目，访问http://localhost:1111/即可进入页面

 ![img](https://img2018.cnblogs.com/blog/398358/201907/398358-20190722112013106-1317411812.png) 

 看Instances currently registered with Eureka， 可以发现信息是：No instances available。
这表示 暂时还没有微服务注册进来。 

#### 创建eureka项目--Client

同上，使用IDEA创建SpringBoot项目，选用Spring Cloud模板中的Eureka Client即创建客户端项目

主要依赖为：

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```

需要注意的是，这是一个实际的SpringBoot微服务，是解耦中的服务的一部分，要实现什么功能是根据需求来的，同时微服务是可以横向扩展的。

#### client配置

配置文件是关键部分，只有指明具体的配置信息才能构成整个微服务应用，示例为：

```properties
##eureka服务注册路径
eureka.client.serviceUrl.defaultZone=http://localhost:8081/eureka/
##实例id，也就是在注册中心页面显示的微服务名
eureka.instance.instance-id=${spring.application.name}:${server.port}
# 设置微服务调用地址为IP优先（缺省为false）
eureka.instance.prefer-ip-address=true
# 心跳时间，即服务续约间隔时间（缺省为30s）
eureka.instance.lease-renewal-interval-in-seconds=30
# 发呆时间，即服务续约到期时间（缺省为90s）
eureka.instance.lease-expiration-duration-in-seconds=90
```

Eureka 首页显示的微服务名默认为：`机器主机名:应用名称:应用端口，`也就是：`${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}`

eureka.client.serviceUrl.defaultZone这个配置可以配置单个注册中心的地址，也可配置多个,逗号隔开，如：

eureka.client.serviceUrl.defaultZone=http://127.0.0.1:8000/eureka/,http://127.0.0.1:8081/eureka/

