## 基础

**接口与抽象类的区别**

接口只定义接口方法，抽象类除了定义抽象方法，也可以有属性与其它实现方法

接口比抽象类更加灵活，可以实现多个接口，但只可以继承一个类

**如果hashMap的key是一个自定义的类，怎么办**

重写equals与hashcode

**为什么重写equals还要重写hashcode**

equals用于比较两个对象是否相同，hashCode是一个对象的散列码，用于进行hash查找key，在hashmap的put中，hashcode用于定位对象在哪条链表中，equals用于判断对象是否相同，如果不重写hashcode那么有可能逻辑上相同的两个对象定位到了不同的链表中导致了重复插入

**关键字final**

可用于成员变量、本地变量、方法以及类。

* 用于成员变量必须在声明的时候初始化或者在构造器中初始化
* 用于本地变量必须在声明时赋值并且不能修改
* final方法不能被重写。
* final类不能被继承。

**String,StringBuffer,StringBuilder区别**

* String不可变，StringBuffer与StringBuilder可变
* StringBuffer线程安全，StringBuilder线程不安全

**String为什么不可变**

String底部使用char[]实现，但是使用了final修饰，因此不可变

## Collection

### HashMap

**HashMap内部实现**

底部使用hash算法，维护hash表（数组/hash桶数组），使用链地址法解决冲突，内部维护变量

```java
 int threshold;             // 所能容纳的key-value对极限 
 final float loadFactor;    // 负载因子
 int modCount;  
 int size;
```

threshold=length*loadFactory，length为hash数组长度，也就是键值对数目由hash数组长度与负载因子决定,loadFactor常取0.75。即使用数值+单向链表

 在HashMap中，哈希桶数组table的长度length大小必须为2的n次方(一定是合数)，这是一种非常规的设计，常规的设计是把桶的大小设计为素数。相对来说素数导致冲突的概率要小于合数 ，因为分布情况由因数决定，冲突时进行取模运算，合数的因子比素数的因子多，因此对素数取模能使分布更加均匀。

put时取hashCode进行hash运算确定hash桶位置，有key则覆盖，没有则插入，插入成功判断容量，容量满则扩容，get同理，扩容时使用新数组覆盖旧数组与所有元素

jdk1.8进行了优化，当链表节点数量超过8是则使用红黑树进行替代（总数大于64）；1.7在链表头部插入，1.8在尾部插入（头插会导致顺序翻转且并发时导致死循环）。线程不安全

**hashMap和ConcurrentHashMap的区别**

ConcurrentHashMap是线程安全的，1.7中的ConcurrentHashMap使用分段锁策略，划分为多个段segment，维护全局索引，插入时先判断在哪个段，在进行插入；1.8的实现已经抛弃了Segment分段锁机制，利用CAS+Synchronized来保证并发更新的安全

### ArrayList/LinkedList

**ArrayList和LinkedList的区别，如果一直在list的尾部添加元素，用哪个效率高？**

Arraylist底层使用数组实现，LinkedList底层使用链表（双向链表，维护first与last）实现

## JVM

**GC工具用过哪些？**

**讲一下什么情况可以影响到新生代的回收速度。**

## 并发

### 基础

**synchronized**

**synchronized基于指令monitorenter和monitorexit实现，是java提供的基本上锁机制。**

每一个对象都有自己的监视器(monitor),获取锁就相当于拿到该对象的监视器进入同步方法/方法块，否则进入同步队列，陷入BLOCKED(阻塞)状态，当监视器退出后重新竞争锁。

synchronized可用于声明方法，同时可以锁住对象。

- 对于普通同步方法，锁住当前实例对象。
- 对于静态同步方法，锁住当前类的Class对象。
- 对于同步方法块，锁住Synchonized括号里配置的对象。

**volatile**

volatile用于声明变量(字段)使其成为共享变量。

volatile的作用有两个：

1. 保证变量的内存可见性
2. 禁止指令重排序

用关键字volatile声明的字段每次被使用时系统都会将该字段写回到内存，同时使缓存中的该字段失效(即该字段可能保存在缓存中，系统将该字段写回到内存并使缓存失效),这样，每当线程需要使用该字段时都需要从内存中读取该字段，保证了该字段的唯一可见性。**注意,volatile只保证了内存可见性，并不保证原子性，volatile适用于get/set，并不适用于getAndOperate。**



## IO

**bio与nio的区别**

BIO：同步阻塞式IO，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。 

 NIO：同步非阻塞式IO，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。









### java基础

#### java内存模型(JMM)

在多核CPU，多级缓存结构当中，每个核都有自己的缓存，这样在多线程环境下就可能存在**缓存一致性**问题，即每个线程对同一变量可能有不同的缓存结果。同时， 为了使处理器内部的运算单元能够尽量的被充分利用，处理器可能会对输入代码进行乱序执行处理 ，即进行**指令重排序**。

JMM在java虚拟机规范中被定义，用于屏蔽掉各种硬件和操作系统的内存访问差异， **保证Java程序在各种平台下对内存的访问都能保证效果一致。**

 Java内存模型规定：

1. 所有的变量都存储在主内存中（虚拟机内存的一部分），每条线程有自己的工作内存，线程的工作内存中保存使用到的变量的主内存副本拷贝
2. 线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存。
3. 不同的线程之间无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步进行。  

JMM底部提供基本原子操作规定内存交互操作细节并保证原子性，可见性，有序性

原子性：提供 两个高级的字节码指令`monitorenter`和`monitorexit` ，对应关键字为synchronized

可见性：提供关键字volatile，读取变量前从主内存刷新变量值并读取

有序性：synchronized与volatile都保证有序性，volatile禁止指令重排，synchronized保证同一时刻只允许一条线程操作。 

#### 集合Collections

##### Iterator

迭代器，一种设计模式，java.util中的一个接口，定义遍历序列元素的三个方法，可以遍历并选择序列中的对象

* boolean hasNext()
* E next()
* default void remove()

实现了Iterator就表示可迭代内部元素而无需知道底层细节，java.lang中还有一个Iterable接口，实现该接口表示可返回迭代器，定义方法(更像是一种解耦机制，将迭代元素与其他功能分开)：

* Iteratot<T\> iterator()

##### ArrayList

内部维护数组，支持随机访问

##### LinkedList

底层为双向链表，不支持随机访问

##### HashMap1.7/1.8

底部使用hash算法，维护hash表（数组/hash桶数组），使用链地址法解决冲突，内部维护变量

```java
 int threshold;             // 所能容纳的key-value对极限 
 final float loadFactor;    // 负载因子
 int modCount;  
 int size;
```

threshold=length*loadFactory，length为hash数组长度，也就是键值对数目由hash数组长度与负载因子决定,loadFactor常取0.75。即使用数值+单向链表

 在HashMap中，哈希桶数组table的长度length大小必须为2的n次方(一定是合数)，这是一种非常规的设计，常规的设计是把桶的大小设计为素数。相对来说素数导致冲突的概率要小于合数 ，因为分布情况由因数决定，冲突时进行取模运算，合数的因子比素数的因子多，因此对素数取模能使分布更加均匀。

put时取hashCode进行hash运算确定hash桶位置，有key则覆盖，没有则插入，插入成功判断容量，容量满则扩容，get同理，扩容时使用新数组覆盖旧数组与所有元素

jdk1.8进行了优化，当链表节点数量超过8是则使用红黑树进行替代（总数大于64），1.7在链表头部插入，1.8在尾部插入，线程不安全







### 并发

#### 进程与线程

进程是程序运行与资源分配的基本单位，一个程序至少有一个进程，一个进程至少有一个线程。

线程操作系统调度的最小单位，是进程的一个执行单元

#### 创建进程的几种方式

继承Thread，重写run()方法

实现接口Runnable()，实现run()方法，作为参数传递给Thread

实现接口Callable()，实现方法call()，交给线程池返回Future，或者使用FutureTask进行包装交给线程池执行或者传参给Thread(因为FutureTask也实现了Runnable接口)

Runnable与Callable的区别在于Runnable中的run是无返回值的，而Callable中的call有返回值，可配合Future与FutureTask获取异步执行结果

**线程状态**

五态模型，新建、就绪、运行、阻塞、死亡

**wait和sleep的区别**

* wait是Object中的方法，与notify()配合实现等待通知机制；sleep是native方法
* wait需要被唤醒，sleep指定睡眠时间就结束
* wait会释放线程持有的锁，sleep不会

wait()是Object方法，有notify()与notifyAll()配合使用，调用wait()的线程陷入等待，释放锁，等待唤醒，notify()从等待池中随机唤醒一个，notifyAll()唤醒所有

#### 锁升级

检查对象头的Mark Word的ThreadID判断是否持有偏向锁，未持有检查使用对象的线程是否存活和是否仍需要使用对象，不需要则置对象为无锁状态，重新获取偏向锁，否则升级为轻量级锁，CAS自旋获取锁，CAS到一定次数仍未得到锁则升级为重量级锁

#### ThreadLocal

线程局部变量，线程私有，实现线程安全的一种方式

#### synchronized/volatile

synchronized保证原子性、可见性，底层基于指令monitorenter与monitorexit

volatile保证内存可见性，将变量刷新到内存中，每次使用从内存获取，避免缓存不一致，禁止指令重排序