ZooKeeper是分布式系统中的协调者。也就是动物管理员。

#### 分布式协调系统

举例说明，假如我们搭建了一个数据库集群，里面有一个Master，多个Slave，Master负责写，Slave只读，我们需要一个系统，来告诉客户端，哪个是Master。 

一种想法就是Client存储所有节点信息，维护一个map, key:master，master即机器对应的ip。该机器如果挂掉，那么整个集群便不可写，因此横向扩展，维护多个机器，客户端可以任意调用，但这时便涉及到了数据同步的问题，如果数据没能及时同步，客户端很可能拿到旧数据，因此数据同步时必须阻塞对master的ip的提供，这样，请求便会变慢。

假设这个数据同步时间无限短，比如是1微妙，可以忽略不计，那么其实这个分布式系统，就和我们之前单机的系统一样，既可以保证数据的一致，又让外界感知不到请求阻塞，同时，又不会有单点失效SPOF（Single Point of Failure）的风险，即不会因为一台机器的宕机，导致整个系统不可用。

 **这样的系统，就叫分布式协调系统。谁能把这个数据同步的时间压缩的更短，谁的请求响应就更快，谁就更出色，Zookeeper就是其中的佼佼者。** **它用起来像单机一样，能够提供数据强一致性，但是其实背后是多台机器构成的集群，不会有SPOF。** 

引用官网的话，即是：

```text
ZooKeeper: A Distributed Coordination Service for Distributed Applications
```

#### CPA理论

**CPA理论**：C-->Consistency（一致性）：数据一致更新，所有数据变动都是同步的。A-->Availability（可用性）：好的相应性能。P-->Partition tolerance（分区容忍性）：可靠性。
一个分布式系统不可能同时满足C（一致性）、A（可用性）和P（分区容错性）。由于分区容错性在分布式系统中是必须要保证的，因此我们只能在A和C之间权衡。Zookeeper保证的是CP。

#### ZooKeeper解决的问题及功能

 Zookeeper 作为一个分布式的服务框架，主要用来解决分布式集群中应用系统的一致性问题，它能提供基于类似于文件系统的目录节点树方式的数据存储，但是 Zookeeper 并不是用来专门存储数据的，它的作用主要是用来维护和监控你存储的数据的状态变化。通过监控这些数据状态的变化，从而可以达到基于数据的集群管理 。

 ZooKeeper 主要是用来维护和监控一个目录节点树中存储的数据的状态，所有我们能够操作 ZooKeeper 的也和操作目录节点树大体一样，如创建一个目录节点，给某个目录节点设置数据，获取某个目录节点的所有子目录节点，给某个目录节点设置权限和监控这个目录节点的状态变化。 

**ZooKeeper主要提供的服务为： 统一配置管理、统一命名服务、分布式锁、集群管理。 **

#### 数据结构

zookeeper是一种树形结构：

 ![ZooKeeper's Hierarchical Namespace](https://zookeeper.apache.org/doc/r3.5.5/images/zknamespace.jpg) 

每一个节点由一个路径标识，节点(ZNode)有两种类型：

- **短暂/临时(Ephemeral)**：当客户端和服务端断开连接后，所创建的Znode(节点)**会自动删除**
- **持久(Persistent)**：当客户端和服务端断开连接后，所创建的Znode(节点)**不会删除**

#### 监听器

常见的监听场景有以下两项：

- 监听Znode节点的**数据变化**
- 监听子节点的**增减变化**

### 统一配置管理

**为什么要用统一配置？**

我们做项目时用到的配置比如数据库配置等...我们都是写死在项目里面，如果需要更改，那么也实地修改配置文件然后再投产上去，那么问题来了，如果做集群的呢，有100台机器，这时候做修改那就太不切实际了；那么就需要用到统一配置管理啦。

**解决思路**

1.把公共配置抽取出来

2.对公共配置进行维护

3.修改公共配置后应用不需要重新部署

**采用方案**

1.公共配置抽取存放于zookeeper中并落地数据库

2.对公共配置修改后发布到zookeeper中并落地数据库

3.对应用开启配置实时监听，zookeeper配置文件一旦被修改，应用可实时监听到并获取

 ![img](https://img-blog.csdn.net/20171207163808475?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMTMyMDc0MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 



参考：

https://blog.csdn.net/u011320740/article/details/78742625

### 统一命名服务

统一命名服务的理解其实跟**域名**一样，是我们为这某一部分的资源给它**取一个名字**，别人通过这个名字就可以拿到对应的资源。

比如说，现在我有一个域名`www.java.com`，但我这个域名下有多台机器：

- 192.168.1.1
- 192.168.1.2
- 192.168.1.3
- 192.168.1.4

别人访问`www.java.com`即可访问到我的机器，而不是通过IP去访问。

 ![img](https://pic3.zhimg.com/v2-4b86e886479dc91b9527f46fe125e45a_b.jpg) 

### 分布式锁

应用于多个系统同时去访问一个资源，为实现并发访问控制而加的锁，一种乐观锁，CAS实现。

我们可以使用ZooKeeper来实现分布式锁：

系统A、B、C都去访问`/locks`节点

 ![img](https://pic4.zhimg.com/v2-4d762a6ece13303b72f33b46a15f0097_b.jpg) 

 访问的时候会创建**带顺序号的临时/短暂**(`EPHEMERAL_SEQUENTIAL`)节点，比如，系统A创建了`id_000000`节点，系统B创建了`id_000002`节点，系统C创建了`id_000001`节点。 

 ![img](https://pic3.zhimg.com/v2-338b221850de334723018c9164804576_b.jpg) 

接着，拿到`/locks`节点下的所有子节点(id_000000,id_000001,id_000002)，**判断自己创建的是不是最小的那个节点**

- 如果是，则拿到锁。释放锁：执行完操作后，把创建的节点给删掉
- 如果不是，则监听比自己要小1的节点变化

### 集群管理

还是使用树形节点与监听实现，为每一台PC都创建一个节点，不断监听，如果PC挂掉则删除该节点，即能感知集群状态。

### 负载均衡

负载均衡是一种手段，用来把对某种资源的访问分摊给不同的设备，从而**减轻单点**的压力。

实现的思路:

1. 首先建立 Servers 节点，并建立监听器监视 Servers 子节点的状态（用于在服务器增添时及时同步当前集群中服务器列表）
2. 在每个服务器启动时，在 Servers 节点下建立**临时子节点** Worker Server，并在对应的字节点下存入服务器的相关信息，包括服务的地址，IP，端口等等
3. 可以**自定义一个负载均衡算法**，在每个请求过来时从 ZooKeeper 服务器中获取当前集群服务器列表，根据算法选出其中一个服务器来处理请求



![img](https://pic2.zhimg.com/80/v2-d1992d74a92087155c492e1732dbffa1_hd.jpg)


 分布式队列

### FIFO

使用 ZooKeeper 实现 FIFO 队列，**入队操作**就是在 `queue_fifo` 下创建自增序的子节点，并把数据（队列大小）放入节点内。**出队操作**就是先找到 `queue_fifo` 下序号最下的那个节点，取出数据，然后删除此节点。

```text
/queue_fifo
|
├── /host1-000000001
├── /host2-000000002
├── /host3-000000003
└── /host4-000000004
```

创建完节点后，根据以下步骤确定执行顺序:

1. 通过 `get_children()` 接口获取 `/queue_fifo` 节点下所有子节点
2. 通过自己的节点序号在所有子节点中的顺序
3. 如果不是最小的子节点，那么进入等待，同时**向比自己序号小的最后一个子节点注册 Watcher 监听**
4. 接收到 Watcher 通知后重复 1

参考资料：

- 分布式服务框架 Zookeeper

- - [https://www.ibm.com/developerworks/cn/opensource/os-cn-zookeeper/index.html](https://link.zhihu.com/?target=https%3A//www.ibm.com/developerworks/cn/opensource/os-cn-zookeeper/index.html)

- ZooKeeper初识整理(老酒装新瓶) 

- - [https://lxkaka.wang/2017/12/21/zookeeper/](https://link.zhihu.com/?target=https%3A//lxkaka.wang/2017/12/21/zookeeper/)

- ZooKeeper

- - [https://www.cnblogs.com/sunshine-long/p/9057191.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/sunshine-long/p/9057191.html)

- ZooKeeper 的应用场景

- - https://zhuanlan.zhihu.com/p/59669985