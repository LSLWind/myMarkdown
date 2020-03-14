#### cas:不加锁实现原子操作(乐观锁)

​    cas即compare and swap，一种乐观锁策略，对于变量x，它的实现过程是，有3个操作数，内存值V，旧的预期值E，要修改的新值U，当且仅当预期值E和内存值V相同时，才将内存值V修改为U，否则什么都不做。

​    **原子操作是并发基本操作，保证字段的唯一修改性。cas常用于并发包中，对于共享字段x，假如要执行x++这一操作，x++必须保证原子性。一种办法是，x++写入方法中，对该方法使用synchronized上锁，肯定能保证x++的原子性，显而易见这种方式会比较耗时，另一种办法就是使用cas。**

```java
AtomicInteger atomicI = new AtomicInteger(0);//包装原子类
for (;;) {//循环CAS
    int i = atomicI.get();//获取当前内存值
    boolean success = atomicI.compareAndSet(i, ++i);//尝试cas
    if (success) {
        break;
    }
}
```

#### cas中的ABA问题

​    ABA问题比较抽象，线程A,B同时拿到了字段x,A执行cas(x,x+1),又执行cas(x,x-1),此时B以为x没变化，但实际上x变化了，如果单纯的是换值，这么做当然没问题，但是如果操控的是带有状态的结构，就可能引发问题。

下面这篇文章写得非常好，***引用于:https://cloud.tencent.com/developer/article/1098132***

> ​    先描述ABA。假设两个线程T1和T2访问同一个变量V，当T1访问变量V时，读取到V的值为A；此时线程T1被抢占了，T2开始执行，T2先将变量V的值从A变成了B，然后又将变量V从B变回了A；此时T1又抢占了主动权，继续执行，它发现变量V的值还是A，以为没有发生变化，所以就继续执行了。这个过程中，变量V从A变为B，再由B变为A就被形象地称为ABA问题了。
>
> ​    上面的描述看上去并不会导致什么问题。T1中的判断V的值是A就不应该有问题的，无论是开始的A，还是ABA后面的A，判断的结果应该是一样的才对。
>
> ​    不容易看出问题的主要还是因为：“值是一样的”等同于“没有发生变化”（就算被改回去了，那也是变化）的认知。毕竟在大多数程序代码中，我们只需要知道值是不是一样的，并不关心它在之前的过程中有没有发生变化；所以，当我需要知道之前的过程中“有没有发生变化”的时候，ABA就是问题了。
>
> **现实ABA问题**
>
> ​     警匪剧看多了人应该可以快速反应到发生了什么。应用到ABA问题，首先，这里的A和B并不表示被掉的包这个实物，而是掉包过程中的状态的变化。假设一个装有10000W箱子（别管它有多大）放在一个房间里，10分钟后再进去拿出来赎人去。但是，有个贼在这10分钟内进去（别管他是怎么进去的）用一个同样大小的空箱子，把我的箱子掉包了。当我再进去看的时候，发现箱子还在，自然也就以为没有问题了的，就继续拿着桌子上的箱子去赎人了（别管重量对不对）。现在只要知道这里有问题就行了，拿着没钱的箱子去赎人还没有问题么？
>
> ​    这里的变量V就是桌子上是否有箱子的状态。A，是桌子上有箱子的状态；B是箱子在掉包过程中，离开桌子，桌子上没有箱子的状态；最后一个A也是桌子上有箱子的状态。但是箱子里面的东西是什么就不不知道了。
>
> **程序世界的ABA问题**
>
> 在运用CAS做Lock-Free操作中有一个经典的ABA问题：
>
> ​     线程1准备用CAS将变量的值由A替换为B，在此之前，线程2将变量的值由A替换为C，又由C替换为A，然后线程1执行CAS时发现变量的值仍然为A，所以CAS成功。但实际上这时的现场已经和最初不同了，尽管CAS成功，但可能存在潜藏的问题，例如下面的例子：
>
> ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9hc2sucWNsb3VkaW1nLmNvbS9odHRwLXNhdmUveWVoZS0xNDM0MTc2L2Z4ZDJwdnc2MXkucG5nP2ltYWdlVmlldzIvMi93LzE2MjA?x-oss-process=image/format,png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)
>
> 现有一个用单向链表实现的堆栈，栈顶为A，这时线程T1已经知道A.next为B，然后希望用CAS将栈顶替换为B：
>
> head.compareAndSet(A,B);
>
> 在T1执行上面这条指令之前，线程T2介入，将A、B出栈，再pushD、C、A，此时堆栈结构如下图，而对象B此时处于游离状态：
>
> ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9hc2sucWNsb3VkaW1nLmNvbS9odHRwLXNhdmUveWVoZS0xNDM0MTc2L3I0ZzlnaGU0amYucG5nP2ltYWdlVmlldzIvMi93LzE2MjA?x-oss-process=image/format,png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)
>
> 此时轮到线程T1执行CAS操作，检测发现栈顶仍为A，所以CAS成功，栈顶变为B，但实际上B.next为null，所以此时的情况变为：
>
> ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9hc2sucWNsb3VkaW1nLmNvbS9odHRwLXNhdmUveWVoZS0xNDM0MTc2L2NmM3hhdG92M3gucG5nP2ltYWdlVmlldzIvMi93LzE2MjA?x-oss-process=image/format,png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)
>
> 其中堆栈中只有B一个元素，C和D组成的链表不再存在于堆栈中，平白无故就把C、D丢掉了。
>
> 以上就是由于ABA问题带来的隐患，各种乐观锁的实现中通常都会用版本戳version来对记录或对象标记，避免并发操作带来的问题，在Java中，AtomicStampedReference<E>也实现了这个作用，它通过包装[E,Integer]的元组来对对象标记版本戳stamp，从而避免ABA问题

​    当然，一般来说使用Actomic原子包即可，如果出现问题，可能是ABA，那么就使用AtomicStampedReference<E\>,实现方式是内部增加一个版本号(实现起来应该比较复杂了)。

```
public boolean compareAndSet(
    V expectedReference, // 预期引用
    V newReference, // 更新后的引用
    int expectedStamp, // 预期标志
    int newStamp // 更新后的标志
)
```

### 