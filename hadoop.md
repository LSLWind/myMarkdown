### 安装Hadoop

 https://cloud.tencent.com/developer/article/1138661 

**前置：安装java**

 https://blog.csdn.net/qq_36801710/article/details/79306319 

**1.下载解压**

```js
tar -xf hadoop-2.5.1.tar.gz -C /usr/local/
```

**2.将hadoop的bin目录添加到系统Path以便可以通过命令行调用hadoop**

打开 ~/ .profile，添加系统Path

```js
export HADOOP_PREFIX=/usr/local/hadoop-2.5.0 #设置路径
export PATH=$HADOOP_PREFIX/bin:$PATH  #引用设置Path
```

**3.对hadoop设置java 路径以便引用jdk**

```java
export JAVA_HOME=/usr/lib/jvm/java-7-oracle
export HADOOP_PREFIX=/usr/local/hadoop-2.5.1
```

改完之后重启以生效

#### 启动伪分布式

 https://cloud.tencent.com/developer/article/1138661 

**1.配置文件**

要启用伪分布式模式，需要编辑以下两个XML文件。这些XML文件在**单个配置元素中**包含多个属性元素。属性元素包含名称和值元素。

```js
etc/hadoop/core-site.xml
etc/hadoop/hdfs-site.xml
```

编辑core-site.xml并修改以下属性。*fs.defaultFS* 属性保存的是NameNode的位置。hadoop.tmp.dir是保存的数据的目录

```js
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/var/lib/hadoop</value>
    </property>
</configuration>
```

编辑hdfs-site.xml并修改以下属性。*dfs.replication* 属性保存的是每个HDFS块应该被复制的次数。

```js
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

 #### 导入hadoop的jar开发包

https://blog.csdn.net/weixin_41122339/article/details/81140616 

### HDFS

分布式文件系统

#### 架构(HDFS Architecture)

使用块（chunk）作为独立存储单元，一个文件由多个块组成，默认设置为128MB

master/slave体系结构，master为NameNode，只有一个，包含多个slave,slave为DataNode。

NameNode管理文件系统，维护文件系统树。NameNode的容错机制有两种：

1. 进行备份，将构成文件系统的元数据进行备份
2. 运行一个辅助namenode，定期合并编辑日志与命名空间镜像，同样是维护元数据的状态

datanode用于从磁盘中读取块，由namenode调度并定期向namenode汇报信息，对于频繁访问的文件，datanode将数据缓存在内存中

![](https://pic.superbed.cn/item/5dbfc3ba8e0e2e3ee9f13d25.jpg)

HDFS对外提供接口与API供客户端操作响应，如命令行接口，也就是说，可以像正常使用文件系统那样使用HDFS，也可以通过API操控HDFS文件或者通过Http方式或Http代理方式访问HDFS

HDFS命令行由bin/hdfs脚本回调执行，HDFS解决了海量数据如何存储的问题，即使用分布式文件系统，维护主NameNode，管控DataNode，文件拆分成块，每个块进行多个备份，文件系统的数据信息保存在NameNode上，可以使用命令行与API控制文件系统。

#### 文件系统命令行操作                                                                                                                                                                                                                                                                                                                                                                   

有关操作，参见官网：

 https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html 

hadoop fs：该命令可以用于其他文件系统，不止是hdfs文件系统内，也就是说该命令的使用范围更广

hadoop dfs：专门针对hdfs分布式文件系统

hdfs dfs：和上面的命令作用相同，相比于上面的命令更为推荐，并且当使用hadoop dfs时内部会被转为hdfs dfs命令，给几个例子：

```shell
 hadoop fs -appendToFile localfile /user/hadoop/hadoopfile 
 hadoop fs -copyFromLocal ./os /input
 hadoop fs -cat /input/os/test1.c
 hdfs dfs -mkdir /input
```

### Yarn

全称Yet Another Resource Negotiator ，即另一种资源调度器，资源管理框架， 负责资源管理和调度 

 ![img](https://pic2.zhimg.com/v2-b5804eb2bb1b3f2ff091c18bbf915819_b.jpg) 

#### 容器Container

 容器（Container）是 Yarn 对资源做的一层抽象。  Yarn 将CPU核数，内存这些计算资源都封装成为一个个的容器（Container）

- 容器由 NodeManager 启动和管理，并被它所监控。
- 容器被 ResourceManager 进行调度。

#### ResourceManager

我们先来说说上图中最中央的那个 ResourceManager（RM）。从名字上我们就能知道这个组件是负责资源管理的，整个系统有且只有一个 RM ，来负责资源的调度。它也包含了两个主要的组件：定时调用器(Scheduler)以及应用管理器(ApplicationManager)。

1. 定时调度器(Scheduler)：从本质上来说，定时调度器就是一种策略，或者说一种算法。当 Client 提交一个任务的时候，它会根据所需要的资源以及当前集群的资源状况进行分配。注意，它只负责向应用程序分配资源，并不做监控以及应用程序的状态跟踪。
2. 应用管理器(ApplicationManager)：同样，听名字就能大概知道它是干嘛的。应用管理器就是负责管理 Client 用户提交的应用。上面不是说到定时调度器（Scheduler）不对用户提交的程序监控嘛，其实监控应用的工作正是由应用管理器（ApplicationManager）完成的。

#### ApplicationMaster

每当 Client 提交一个 Application 时候，就会新建一个 ApplicationMaster 。由这个 ApplicationMaster 去与 ResourceManager 申请容器资源，获得资源后会将要运行的程序发送到容器上启动，然后进行分布式计算。

这里可能有些难以理解，为什么是把运行程序发送到容器上去运行？如果以传统的思路来看，是程序运行着不动，然后数据进进出出不停流转。但当数据量大的时候就没法这么玩了，因为海量数据移动成本太大，时间太长。但是中国有一句老话**山不过来，我就过去。**大数据分布式计算就是这种思想，既然大数据难以移动，那我就把容易移动的应用程序发布到各个节点进行计算呗，这就是大数据分布式计算的思路。

#### NodeManager

NodeManager 是 ResourceManager 在每台机器的上代理，负责容器的管理，并监控他们的资源使用情况（cpu，内存，磁盘及网络等），以及向 ResourceManager/Scheduler 提供这些资源使用报告。

Yarn应用的运行机制如下：

 ![img](https://pic3.zhimg.com/v2-8b32b881d0d8ea046455a6d77868aebe_b.jpg) 

对资源进行封装：

```java
class Resource{
 int cpu;  //cpu核心个数
 int memory-mb; //内存的MB数
}
```

向Yarn申请资源：

```java
class ResourceRequest{
 int numContainers; //需要的container个数
 Resource capability;//每个container的资源
}
```

Yarn响应作业并分配资源：

```java
class Container{
 ContainerId containerId; //YARN全局唯一的container标示
 Resource capability;  //该container的资源信息
 String nodeHttpAddress; //该container可以启动的NodeManager的hostname
}
```

分配资源后交给NodeManager去处理

![MlAUr4.md.png](https://s2.ax1x.com/2019/11/11/MlAUr4.md.png)



#### Yarn调度

### MapReduce

hadoop中的分布式计算框架，一种编程模型，用于大数据集的并行运算，map映射与reduce规约是其核心思想。

假如要对海量数据进行计算，交给同一个程序进行运算，有多个PC机，将海量数据分成多个，有一个master，多个Worker，Worker有分为Mapper与Reducer，每一份数据map给一台机器（Mapper），执行完毕后对计算过的数据进行规约reduce。MapReduce更像是一种编程规范，一种分布式计算思想，实际上分布式计算框架使用Spark更多。input→Split→Map→Shuffle→Reduce

以词频统计为例：

![preview](https://pic2.zhimg.com/v2-22fa92c7964c1670fe2e3ba382a7b8d5_r.jpg)  

MapReduce提交任务的方式为：

[![Mlld5F.md.png](https://s2.ax1x.com/2019/11/11/Mlld5F.md.png)](https://imgchr.com/i/Mlld5F)

#### 示例

