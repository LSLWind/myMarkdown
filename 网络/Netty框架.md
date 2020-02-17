## 基础

在网络编程领域，Netty是Java的卓越框架。它驾驭了Java高级API的能力，并将其隐藏在一个易于使用的API之后。Netty的主要特性：

| 分　　类 | Netty的特性                                                  |
| -------- | ------------------------------------------------------------ |
| 设计     | 统一的API，支持多种传输类型，阻塞的和非阻塞的                                                                                                  简单而强大的线程模型                                                                                                                                              真正的无连接数据报套接字支持                                                                                                                      链接逻辑组件以支持复用 |
| 易于使用 | 详实的Javadoc和大量的示例集                                  |
| 性能     | 拥有比Java的核心API更高的吞吐量以及更低的延迟                                                                              得益于池化和复用，拥有更低的资源消耗                                                                                                最少的内存复制 |
| 健壮性   | 不会因为慢速、快速或者超载的连接而导致`OutOfMemoryError`                                                        消除在高速网络中NIO应用程序常见的不公平读/写比率 |
| 安全性   | 完整的SSL/TLS以及StartTLS支持                                                                                                             可用于受限环境下，如Applet和OSGI |
| 社区驱动 | 发布快速而且频繁                                             |

### Netty核心组件

**Channel**：Channel是Java NIO的一个基本构造。

> 它代表一个到实体（如一个硬件设备、一个文件、一个网络套接字或者一个能够执行一个或者多个不同的I/O操作的程序组件）的开放连接，如读操作和写操作。

目前，可以把`Channel`看作是传入（入站）或者传出（出站）数据的载体。因此，它可以被打开或者被关闭，连接或者断开连接。

**回调**：一个回调其实就是一个方法，一个指向已经被提供给另外一个方法的方法的引用。Netty在内部使用了回调来处理事件；当一个回调被触发时，相关的事件可以被一个接口`ChannelHandler`的实现处理。如：

```java
public class ConnectHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx)throws Exception {//一个新的连接已经被建立时，channelActive(ChannelHandlerContext)将会被调用
        System.out.println(
            "Client " + ctx.channel().remoteAddress() + " connected");
    }
}
```

当一个新的连接已经被建立时，`ChannelHandler`的`channelActive()`回调方法将会被调用，并将打印出一条信息。

**Future**：`Future`提供了另一种在操作完成时通知应用程序的方式。这个对象可以看作是一个异步操作的结果的占位符；它将在未来的某个时刻完成，并提供对其结果的访问。

JDK预置了`interface java.util.concurrent.Future`，但是其所提供的实现，只允许手动检查对应的操作是否已经完成，或者一直阻塞直到它完成。这是非常繁琐的，所以Netty提供了它自己的实现——`ChannelFuture`，用于在执行异步操作的时候使用。

每个Netty的出站I/O操作都将返回一个`ChannelFuture`；也就是说，它们都不会阻塞，Netty完全是异步和事件驱动的，如：

```java
Channel channel = ...;
// Does not block
ChannelFuture future = channel.connect(     ← --  异步地连接到远程节点
    new InetSocketAddress("192.168.0.1", 25));
```

这里，`connect()`方法将会直接返回，而不会阻塞，该调用将会在后台完成。这究竟什么时候会发生则取决于若干的因素，但这个关注点已经从代码中抽象出来了。因为线程不用阻塞以等待对应的操作完成，所以它可以同时做其他的工作，从而更加有效地利用资源。

```java
   new InetSocketAddress("192.168.0.1", 25));
future.addListener(new ChannelFutureListener() {//注册一个ChannelFutureListener，以便在操作完成时获得通知
    @Override
    public void operationComplete(ChannelFuture future) {//❶ 检查操作的状态
       if (future.isSuccess()){ //如果操作是成功的，则创建一个ByteBuf以持有数据
           ByteBuf buffer = Unpooled.copiedBuffer("Hello",Charset.defaultCharset());
           ChannelFuture wf = future.channel().writeAndFlush(buffer); //将数据异步地发送到远程节点。
返回一个ChannelFuture
            ....
        } else {
            Throwable cause = future.cause();//如果发生错误，则访问描述原因的Throwable
            cause.printStackTrace();
        }
    }
});
```

首先，要连接到远程节点上。然后，要注册一个新的`ChannelFutureListener`到对`connect()`方法的调用所返回的`ChannelFuture`上。当该监听器被通知连接已经建立的时候，要检查对应的状态❶。如果该操作是成功的，那么将数据写到该`Channel`。否则，要从`ChannelFuture`中检索对应的`Throwable`。

回调和`Future`是相互补充的机制；它们相互结合，构成了Netty本身的关键构件块之一。

**事件和ChannelHandler**：Netty使用不同的事件来通知我们状态的改变或者是操作的状态。这使得我们能够基于已经发生的事件来触发适当的动作。每个事件都可以被分发给`ChannelHandler`类中的某个用户实现的方法。这是一个很好的将事件驱动范式直接转换为应用程序构件块的例子

![1zX82j.png](https://s2.ax1x.com/2020/02/15/1zX82j.png)

### 实例

