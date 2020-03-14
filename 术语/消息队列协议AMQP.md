**高级消息队列协议**即**Advanced Message Queuing Protocol**（AMQP）是[面向消息中间件](https://zh.wikipedia.org/w/index.php?title=面向消息中间件&action=edit&redlink=1)提供的开放的应用层协议，其设计目标是对于*消息*的排序、路由（包括点对点和[订阅-发布](https://zh.wikipedia.org/wiki/发布/订阅)）、保持可靠性、保证安全性[[1\]](https://zh.wikipedia.org/wiki/高级消息队列协议#cite_note-acmqueue-1)。AMQP规范了消息传递方和接收方的行为，以使消息在不同的提供商之间实现[互操作性](https://zh.wikipedia.org/wiki/互操作性)，就像[SMTP](https://zh.wikipedia.org/wiki/SMTP)，[HTTP](https://zh.wikipedia.org/wiki/HTTP)，[FTP](https://zh.wikipedia.org/wiki/FTP)等协议可以创建交互系统一样。与先前的中间件标准（如[Java消息服务](https://zh.wikipedia.org/wiki/Java消息服务)）不同的是，JMS在特定的API接口层面和实现行为上进行了统一，而高级消息队列协议则关注于各种消息如何以字节流的形式进行传递。因此，使用了符合协议实现的任意应用程序之间可以保持对消息的创建、传递。

高级消息队列协议是一种二进制[应用层](https://zh.wikipedia.org/wiki/应用层)协议，用于应对广泛的面向消息应用程序的支持。协议提供了消息流控制，保证的一个消息对象的传递过程，如*至多一次*、*保证多次*、*仅有一次*等，和基于[SASL](https://zh.wikipedia.org/wiki/简单认证与安全层)和[TLS](https://zh.wikipedia.org/wiki/TLS)的身份验证和消息加密.

![image.png](https://i.loli.net/2020/03/09/1C47dSyLGbF9tQx.png)

可以看到，消息的生产者将信息发布到一个Exchange。Exchange会绑定到一个或多个队列上，它负责将信息路由到队列上。信息的消费者会从队列中提取数据并进行处理。

Exchange不是简单地将消息传递到队列中，并不仅仅是一种穿透（pass-through）机制。AMQP定义了四种不同类型的Exchange，每一种都有不同的路由算法，这些算法决定了是否要将信息放到队列中。根据Exchange的算法不同，它可能会使用消息的*routing key*和/或参数，并将其与Exchange和队列之间binding的routing key和参数进行对比。（routing key可以大致理解为Email的收件人地址，指定了预期的接收者。）如果对比结果满足相应的算法，那么消息将会路由到队列上。否则的话，将不会路由到队列上。

四种标准的AMQP Exchange如下所示：

- Direct：如果消息的routing key与binding的routing key直接匹配的话，消息将会路由到该队列上；
- Topic：如果消息的routing key与binding的routing key符合通配符匹配的话，消息将会路由到该队列上；
- Headers：如果消息参数表中的头信息和值都与bingding参数表中相匹配，消息将会路由到该队列上；
- Fanout：不管消息的routing key和参数表的头信息/值是什么，消息将会路由到所有队列上。

借助这四种类型的Exchange，很容易就能想到我们可以定义任意数量的路由模式，而不再仅限于点对点和发布-订阅的方式。[[3\]](ms-local-stream://EpubReader_EEF4C438D72341C890C0EE8B22FBF048/Content/OEBPS/Text/part0029.xhtml#anchor173)好消息是，当发送和接收消息的时候，所涉及的路由算法对于如何编写消息的生产者和消费者并没有什么影响。简单来讲，生产者将信息发送给Exchange并带有一个routing key，消费者从队列中获取消息。