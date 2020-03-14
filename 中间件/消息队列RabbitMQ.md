## 基础

RabbitMQ是一个流行的开源消息代理，它实现了AMQP。Spring AMQP为RabbitMQ提供了支持，包括RabbitMQ连接工厂、模板以及Spring配置命名空间。

### 配置

配置RabbitMQ连接工厂最简单的方式就是使用Spring AMQP所提供的`rabbit`配置命名空间。为了使用这项功能，需要确保在Spring配置文件中已经声明了该模式：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/rabbit"
  xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/rabbit
   http://www.springframework.org/schema/rabbit/spring-rabbit-1.0.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

...

</beans:beans>
```

尽管不是必须的，但我还选择在这个配置中将`rabbit`作为首选的命名空间，将`beans`作为第二位的命名空间。这是因为在这个配置中，我会更多的声明`rabbit`而不是bean，这样的话，只会有少量的bean元素使用“`beans:`”前缀，而rabbit元素就能够避免使用前缀了。

`rabbit`命名空间包含了多个在Spring中配置RabbitMQ的元素。但此时，你最感兴趣的可能就是`<connection-factory>`。按照其最简单的形式，我们可以在配置RabbitMQ连接工厂的时候没有任何属性：

```xml
<connection-factory/>
```

这的确能够运行起来，但是所导致的结果就是连接工厂bean没有可用的bean ID，这样的话就难将连接工厂装配到需要它的bean中。因此，我们可能希望通过`id`属性为其设置一个bean ID：

```xml
<connection-factory id="connectionFactory" />
```

默认情况下，连接工厂会假设RabbitMQ服务器监听`localhost`的5672端口，并且用户名和密码均为guest。对于开发来讲，这是合理的默认值，但是对于生产环境，我们可能希望修改这些默认值。如下`<connection-factory>`的设置重写了默认的做法：

```xml
<connection-factory id="connectionFactory"
    host="${rabbitmq.host}"
    port="${rabbitmq.port}"
    username="${rabbitmq.username}"
    password="${rabbitmq.password}" />
```

### 声明队列、Exchange以及binding

在JMS中，队列和主题的路由行为都是通过规范建立的，AMQP与之不同，它的路由更加丰富和灵活，依赖于如何定义队列和Exchange以及如何将它们绑定在一起。声明队列、Exchange和binding的一种方式是使用RabbitMQ `Channel`接口的各种方法。但是直接使用RabbitMQ的`Channel`接口非常麻烦。Spring AMQP能否帮助我们声明消息路由组件呢？

幸好，rabbit命名空间包含了多个元素，帮助我们声明队列、Exchange以及将它们结合在一起的binding。表17.3中列出了这些元素。

![image.png](https://i.loli.net/2020/03/09/pXKsmWaRzi8yVIe.png)

这些配置元素要与`<admin>`元素一起使用。`<admin>`元素会创建一个RabbitMQ管理组件（administrative component），它会自动创建（如果它们在RabbitMQ代理中尚未存在的话）上述这些元素所声明的队列、Exchange以及binding。

例如，如果你希望声明名为`spittle.alert.queue`的队列，只需要在Spring配置中添加如下的两个元素即可：

```xml
<admin connection-factory="connectionFactory"/>
 <queue id="spittleAlertQueue" name="spittle.alerts" />
```

对于简单的消息来说，我们只需做这些就足够了。这是因为默认会有一个没有名称的direct Exchange，所有的队列都会绑定到这个Exchange上，并且routing key与队列的名称相同。在这个简单的配置中，我们可以将消息发送到这个没有名称的Exchange上，并将routing key设定为`spittle.alert.queue`，这样消息就会路由到这个队列中。实际上，我们重新创建了JMS的点对点模型。

但是，更加有意思的路由需要我们声明一个或更多的Exchange，并将其绑定到队列上。例如，如果要将消息路由到多个队列中，而不管routing key是什么，我们可以按照如下的方式配置一个fanout以及多个队列：

```xml
<admin connection-factory="connectionFactory" /
　　　> <queue name="spittle.alert.queue.1" > <queue name="spittle.alert.queue
　　　.2" > <queue name="spittle.alert.queue.3" > <fanout-
　　　exchange name="spittle.fanout">  <bindings>   <binding queue="spittle.al
　　　ert.queue.1" />   <binding queue="spittle.alert.queue.2" /
　　　>   <binding queue="spittle.alert.queue.3" />  </bindings> </fanout-
　　　exchange>
```

