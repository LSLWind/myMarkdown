## 基础

**同步消息**：当客户端调用远程方法时，客户端必须等到远程方法完成后，才能继续执行。即使远程方法不向客户端返回任何信息，客户端也要被阻塞直到服务完成。

<img src="https://i.loli.net/2020/03/09/B9xLolKQi7kFqEW.png" alt="image.png" style="zoom:50%;" />

**异步消息**：客户端不需要等待服务处理消息，甚至不需要等待消息投递完成。客户端发送消息，然后继续执行，这是因为客户端假定服务最终可以收到并处理这条消息。

<img src="https://i.loli.net/2020/03/09/s9pYfxCl46Qn3aj.png" alt="image.png" style="zoom:50%;" />

在异步消息中有两个主要的概念：**消息代理**（message broker）和**目的地**（destination）。当一个应用发送消息时，会将消息交给一个消息代理。消息代理实际上类似于邮局。消息代理可以确保消息被投递到指定的目的地，同时解放发送者，使其能够继续进行其他的业务。

当我们通过邮局邮递信件时，最重要的是要写上地址，这样邮局就可以知道这封信应该被投递到哪里。与此类似，每条异步消息都带有一个目的地，目的地就好像一个邮箱，可以将消息放入这个邮箱，直到有人将它们取走。

但是，并不像信件地址那样必须标识特定的收件人或街道地址，消息中的目的地相对来说并不那么具体。目的地只关注消息应该从**哪里**获得——而不关心是由**谁**取走消息的。这种情况下，目的地就如同信件的地址为“本地居民”。

尽管不同的消息系统会提供不同的消息路由模式，但是有两种通用的目的地：队列（queue）和主题（topic）。每种类型都与特定的消息模型相关联，分别是点对点模型（队列）和发布/订阅模型（主题）。

### 点对点消息模型

在点对点模型中，每一条消息都有一个发送者和一个接收者。当消息代理得到消息时，它将消息放入一个队列中。当接收者请求队列中的下一条消息时，消息会从队列中取出，并投递给接收者。因为消息投递后会从队列中删除，这样就可以保证消息只能投递给一个接收者。

<img src="https://i.loli.net/2020/03/09/rLw65yvQVqxZgoc.png" alt="image.png" style="zoom:50%;" />

尽管消息队列中的每一条消息只被投递给一个接收者，但是并不意味着只能使用一个接收者从队列中获取消息。事实上，通常可以使用几个接收者来处理队列中的消息。不过，每个接收者都会处理自己所接收到的消息。

这与在银行排队等候类似。在等待时，我们可能注意到很多银行柜员都可以帮助我们处理金融业务。在柜员帮助客户完成业务后，她就空闲了，此时，她会要求排队等候的下一个人前来办理业务。如果我们排在队伍的最前边时，我们就会被叫到，然后由其中的一个空闲柜员来帮助我们处理业务，而其他的柜员则会帮助其他的银行客户。

从另一个角度看，我们在银行排队时，并不知道哪一个柜员会帮助我们办理业务。我们可以计算队伍中有多少人，与柜员的数目进行比较，注意哪一个柜员业务办理速度最快，然后猜测会由哪一个柜员办理我们的业务。但是，一般情况下我们都会猜错，最终会由另一个柜员来办理。

同样，在点对点的消息中，如果有多个接收者监听队列，我们也无法知道某条特定的消息会由哪一个接收者处理。这种不确定性实际上有很多好处，因为我们只需要简单地为队列添加新的监听器就能提高应用的消息处理能力。

### 发布—订阅消息模型

在发布—订阅消息模型中，消息会发送给一个主题。与队列类似，多个接收者都可以监听一个主题。但是，与队列不同的是，消息不再是只投递给一个接收者，而是主题的所有订阅者都会接收到此消息的副本

<img src="https://i.loli.net/2020/03/09/MsBXbm6ZkxIlPSR.png" alt="image.png" style="zoom:50%;" />

对于异步消息来讲，发布者并不知道谁订阅了它的消息。发布者只知道它的消息要发送到一个特定的主题——而不知道有谁在监听这个主题。也就是说，发布者并不知道消息是如何被处理的。

## 使用JMS发送消息

Java消息服务（Java Message Service ，JMS）是一个Java标准，定义了使用消息代理的通用API。在JMS出现之前，每个消息代理都有私有的API，这就使得不同代理之间的消息代码很难通用。但是借助JMS，所有遵从规范的实现都使用通用的接口，这就类似于JDBC为数据库操作提供了通用的接口一样。

Spring通过基于模板的抽象为JMS功能提供了支持，这个模板也就是`JmsTemplate`。使用`JmsTemplate`，能够非常容易地在消息生产方发送队列和主题消息，在消费消息的那一方，也能够非常容易地接收这些消息。Spring还提供了消息驱动POJO的理念：这是一个简单的Java对象，它能够以异步的方式响应队列或主题上到达的消息。

### 在Spring中搭建消息代理

ActiveMQ是一个伟大的开源消息代理产品，也是使用JMS进行异步消息传递的最佳选择。

**创建连接工厂**

在所有的示例中，我们都需要借助JMS连接工厂通过消息代理发送消息。因为选择了ActiveMQ作为我们的消息代理，所以我们必须配置JMS连接工厂，让它知道如何连接到ActiveMQ。`ActiveMQConnectionFactory`是ActiveMQ自带的连接工厂，在Spring中可以使用如下方式进行配置：

```java
<bean id="connectionFactory"
      class="org.apache.activemq.spring.ActiveMQConnectionFactory" />
```

默认情况下，`ActiveMQConnectionFactory`会假设ActiveMQ代理监听localhost的61616端口。对于开发环境来说，这没有什么问题，但是在生产环境下，ActiveMQ可能会在不同的主机和/端口上。如果是这样的话，我们可以使用`brokerURL`属性来指定代理的URL：

```java
<bean id="connectionFactory"
      class="org.apache.activemq.spring.ActiveMQConnectionFactory"
      p:brokerURL="tcp://localhost:61616"/>
```

配置连接工厂还有另外一种方式，既然我们知道正在与ActiveMQ打交道，那我们就可以使用ActiveMQ自己的Spring配置命名空间来声明连接工厂（适用于ActiveMQ 4.1之后的所有版本）。首先，我们必须确保在Spring的配置文件中声明了`amq`命名空间：

```xml
<?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:jms="http://www.springframework.org/schema/jms"
   xmlns:amq="http://activemq.apache.org/schema/core"
   xsi:schemaLocation="http://activemq.apache.org/schema/core
     http://activemq.apache.org/schema/core/activemq-core.xsd
     http://www.springframework.org/schema/jms
     http://www.springframework.org/schema/jms/spring-jms.xsd
     http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans.xsd">
   ...
  </beans>
```

现在我们就可以使用`<amq:connectionFactory>`元素声明连接工厂：

```xml
<amq:connectionFactory id="connectionFactory"
                       brokerURL="tcp://localhost:61616"/>
```

注意，`<amq:connectionFactory>`元素很明显是为ActiveMQ所准备的。如果我们使用不同的消息代理实现，它们不一定会提供Spring配置命名空间。如果没有提供的话，那我们就需要使用`<bean>`来装配连接工厂。

**声明ActiveMQ消息目的地**

除了连接工厂外，我们还需要消息传递的目的地。目的地可以是一个队列，也可以是一个主题，这取决于应用的需求。

不论使用的是队列还是主题，我们都必须使用特定的消息代理实现类在Spring中配置目的地bean。例如，下面的`<bean>`声明定义了一个ActiveMQ队列：

```xml
<bean id="queue"
      class="org.apache.activemq.command.ActiveMQQueue"
      c:_="spitter.queue" />
```

同样，下面的`<bean>`声明定义了一个ActiveMQ主题：

```xml
<bean id="topic"
      class="org.apache.activemq.command.ActiveMQTopic"
      c:_="spitter.queue" />
```

在第一个示例中，构造器指定了队列的名称，这样消息代理就能获知该信息，而在接下来示例中，名称则为`spitter.topic`。

与连接工厂相似的是，ActiveMQ命名空间提供了另一种方式来声明队列和主题。对于队列，我们可以使用`<amq:quence>`元素来声明：

```xml
<amq:queue id="spittleQueue" physicalName="spittle.alert.queue" />
```

如果是JMS主题，我们可以使用`<amq:topic>`元素来声明：

```xml
<amq:topic id="spittleTopic" physicalName="spittle.alert.topic" />
```

不管是哪种类型，都是借助`physicalName`属性指定消息通道的名称。

### 使用Spring的JMS模板

正如我们所看到的，JMS为Java开发者提供了与消息代理进行交互来发送和接收消息的标准API，而且几乎每个消息代理实现都支持JMS，因此我们不必因为使用不同的消息代理而学习私有的消息API。

虽然JMS为所有的消息代理提供了统一的接口，但是这种接口用起来并不是很方便。使用JMS发送和接收消息并不像拿一张邮票并贴在信封上那么简单。正如我们将要看到的，JMS还要求我们为邮递车加油（只是比喻的说法）。

针对如何消除冗长和重复的JMS代码，Spring给出的解决方案是`JmsTemplate`。`JmsTemplate`可以创建连接、获得会话以及发送和接收消息。这使得我们可以专注于构建要发送的消息或者处理接收到的消息。

为了使用`JmsTemplate`，我们需要在Spring的配置文件中将它声明为一个bean。

```xml
<bean id="jmsTemplate"
      class="org.springframework.jms.core.JmsTemplate"
      c:_-ref="connectionFactory" />
```

因为`JmsTemplate`需要知道如何连接到消息代理，所以我们必须为`connection-Factory`属性设置实现了JMS的`ConnectionFactory`接口的bean引用，也就是上述配置中指定的\<amq:connectionFactory\>元素所声明的ip地址。在这里，使用上述配置

### 发送消息

在我们想建立的Spittr应用程序中，其中有一个特性就是当创建Spittle的时候提醒其他用户（或许是通过E-mail）。我们可以在增加Spittle的地方直接实现该特性。但是搞清楚发送提醒给谁以及实际发送这些提醒可能需要一段时间，这会影响到应用的性能。当增加一个新的Spittle时，我们希望应用是敏捷的，能够快速做出响应。

与其在增加Spittle时浪费时间发送这些信息，不如对该项工作进行排队，在响应返回给用户之后再处理它。与直接发送消息给其他用户所花费的时间相比，发送消息给队列或主题所花费的时间是微不足道的。

为了在Spittle创建的时候异步发送spittle提醒，让我们为Spittr应用引入`AlertService`：

```java
package com.habuma.spittr.alerts;
import com.habuma.spittr.domain.Spittle;

public interface AlertService {
  void sendSpittleAlert(Spittle spittle);
}
```

正如我们所看到的，`AlertService`是一个接口，只定义了一个操作—— `sendSpittleAlert()`。

下述示例中，`AlertServiceImpl`实现了`AlertService`接口，它使用`JmsOperation`（`JmsTemplate`所实现的接口）将`Spittle`对象发送给消息队列，而队列会在稍后得到处理。

<img src="https://i.loli.net/2020/03/09/bgvjSXJPq9rLInh.png" alt="image.png" style="zoom:50%;" />

`JmsOperations`的`send()`方法的第一个参数是JMS目的地名称，标识消息将发送给谁。当调用`send()`方法时，`JmsTemplate`将负责获得JMS连接、会话并代表发送者发送消息。我们使用`MessageCreator`（在这里的实现是作为一个匿名内部类）来构造消息。在`MessageCreator`的`createMessage()`方法中，我们通过session创建了一个对象消息：传入一个`Spittle`对象，返回一个对象消息。

**设置默认目的地**

上述示例中，我们明确指定了一个目的地，在`send()`方法中将Spittle消息发向此目的地。当我们希望通过程序选择一个目的地时，这种形式的`send()`方法很适用。但是在`AlertServiceImpl`案例中，我们总是将Spittle消息发给相同的目的地，所以这种形式的`send()`方法并不能带来明显的好处。

与其每次发送消息时都指定一个目的地，不如我们为`JmsTemplate`装配一个默认的目的地：

```xml
<bean id="jmsTemplate"
      class="org.springframework.jms.core.JmsTemplate"
      c:_-ref="connectionFactory"
      p:defaultDestinationName="spittle.alert.queue" />
```

在这里，将目的地的名称设置为`spittle.alert.queue`，但它只是一个名称：它并没有说明你所处理的目的地是什么类型。如果已经存在该名称的队列或主题的话，就会使用已有的。如果尚未存在的话，将会创建一个新的目的地（通常会是队列）。但是，如果你想指定要创建的目的地类型的话，那么你可以将之前创建的队列或主题的目的地bean装配进来：

```xml
<bean id="jmsTemplate"
      class="org.springframework.jms.core.JmsTemplate"
      c:_-ref="connectionFactory"
      p:defaultDestination-ref="spittleTopic" />
```

现在，调用`JmsTemplate`的`send()`方法时，我们可以去除第一个参数了：

```java
jmsOperations.send(
  new MessageCreator() {
  ...
  }
);
```

### 在发送时，对消息进行转换

除了`send()`方法，`JmsTemplate`还提供了`convertAndSend()`方法。与`send()`方法不同，`convertAndSend()`方法并不需要`MessageCreator`作为参数。这是因为`convertAndSend()`会使用内置的消息转换器（message converter）为我们创建消息。

当我们使用`convertAndSend()`时，`sendSpittleAlert()`可以减少到方法体中只包含一行代码：

```
public void sendSpittleAlert(Spittle spittle) {
  jmsOperations.convertAndSend(spittle);
}
```

就像变魔术一样，`Spittle`会在发送之前转换为`Message`。不过就像所有的魔术一样，`JmsTemplate`内部会进行一些处理。它使用一个MessageConverter的实现类将对象转换为`Message`。

`MessageConverter`是Spring定义的接口，只有两个需要实现的方法：

```java
public interface MessageConverter {
  Message toMessage(Object object, Session session)
                   throws JMSException, MessageConversionException;
  Object fromMessage(Message message)
                   throws JMSException, MessageConversionException;
}
```

尽管这个接口实现起来很简单，但我们通常并没有必要创建自定义的实现。Spring已经提供了多个实现

| 消息转换器                        | 功　　能                                                     |
| --------------------------------- | ------------------------------------------------------------ |
| `MappingJacksonMessageConverter`  | 使用Jackson JSON库实现消息与JSON格式之间的相互转换           |
| `MappingJackson2MessageConverter` | 使用Jackson 2 JSON库实现消息与JSON格式之间的相互转换         |
| `MarshallingMessageConverter`     | 使用JAXB库实现消息与XML格式之间的相互转换                    |
| `SimpleMessageConverter`          | 实现String与TextMessage之间的相互转换，字节数组与`BytesMessage`之间的相互转换，`Map`与`MapMessage`之间的相互转换以及`Serializable`对象与`ObjectMessage`之间的相互转换 |

默认情况下，`JmsTemplate`在`convertAndSend()`方法中会使用`SimpleMessage Converter`。但是通过将消息转换器声明为bean并将其注入到`JmsTemplate`的`messageConverter`属性中，我们可以重写这种行为。例如，如果你想使用JSON消息的话，那么可以声明一个`MappingJacksonMessageConverter` bean：

```xml
<bean id="messageConverter"
      class="org.springframework.jms.support.converter.MappingJacksonMessageConverter" />
```

然后，我们可以将其注入到`JmsTemplate`中，如下所示：

```xml
<bean id="jmsTemplate"
      class="org.springframework.jms.core.JmsTemplate"
      c:_-ref="connectionFactory"
      p:defaultDestinationName="spittle.alert.queue"
      p:messageConverter-ref="messageConverter" />
```

各个消息转换器可能会有额外的配置，进而实现转换过程的细粒度控制。例如，`MappingJacksonMessageConverter`能够让我们配置转码以及自定义Jackson `ObjectMapper`。可以查阅每个消息转换器的JavaDoc以了解如何更加细粒度地配置它们。

### 接收消息

现在我们已经了解了如何使用`JmsTemplate`发送消息。但如果我们是接收端，那要怎么办呢？JmsTemplate是不是也可以接收消息呢？

没错，的确可以。事实上，使用`JmsTemplate`接收消息甚至更简单，我们只需要调用`JmsTemplate`的`receive()`方法即可

![image.png](https://i.loli.net/2020/03/09/S1ePRUaYphTGDgb.png)

当调用`JmsTemplate`的`receive()`方法时，`JmsTemplate`会尝试从消息代理中获取一个消息。如果没有可用的消息，`receive()`方法会一直等待，直到获得消息为止。

### 消息驱动

待续



