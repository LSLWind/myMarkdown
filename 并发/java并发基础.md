### 基本概念

java代码在javac编译后会变成Java字节码，字节码被类加载器加载到JVM里，JVM执行字节码，最终需要转化为汇编指令在CPU上执行，Java中所使用的并发机制依赖于JVM的实现和CPU的指令。

​    **现代操作系统采用时分的方式调度运行的程序，系统为每个线程分配时间片，当时间片用完了就发生线程的调度**。多核处理器当然可以同时处理多个任务，但总内存还是只有那么一块。因此在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中数据读到处理器缓存里。

* 并发：指多个任务交替执行，而多个任务之间有可能还是串行的。
* 并行：多个任务在绝对时间上同时执行
* 同步：同步方法调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为。
* 异步：异步方法调用更像一个消息传递，一旦开始，方法调用就会立即返回，调用者就可以继续后续的操作。而异步方法通常会在另外一个线程中“真实”地执行。整个过程，不会阻碍调用者的工作。

#### 关键字volatile

​    volatile用于声明变量(字段)使其成为共享变量。用关键字volatile声明的字段每次被使用时系统都会将该字段写回到内存，同时使缓存中的该字段失效(即该字段可能保存在缓存中，系统将该字段写回到内存并使缓存失效),这样，每当线程需要使用该字段时都需要从内存中读取该字段，保证了该字段的唯一可见性。**注意,volatile只保证了内存可见性，并不保证原子性，volatile适用于get/set，并不适用于getAndOperate。**

volatile的作用有两个：

1. 保证变量的内存可见性
2. 禁止指令重排序

#### 关键字synchronized

synchronized可用于声明方法，同时可以锁住对象。

- 对于普通同步方法，锁住当前实例对象。
- 对于静态同步方法，锁住当前类的Class对象。
- 对于同步方法块，锁住Synchonized括号里配置的对象。

**synchronized基于指令monitorenter和monitorexit实现，是java提供的基本上锁机制。**

每一个对象都有自己的监视器(monitor),获取锁就相当于拿到该对象的监视器进入同步方法/方法块，否则进入同步队列，陷入BLOCKED(阻塞)状态，当监视器退出后重新竞争锁。

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

### 锁

synchronized的锁机制更加细化，synchronized是可重入的

**偏向锁**
       Hotspot 的作者经过以往的研究发现大多数情况下锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程 ID，以后该线程在进入和退出同步块时不需要花费CAS操作来加锁和解锁，而只需简单的测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁，如果测试成功，表示线程已经获得了锁，如果测试失败，则需要再测试下 Mark Word中偏向锁的标识是否设置成 1（表示当前是偏向锁），如果没有设置，则使用 CAS 竞争锁，如果设置了，则尝试使用 CAS 将对象头的偏向锁指向当前线程。

   偏向锁的撤销：偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁。偏向锁的撤销，需要等待全局安全点（在这个时间点上没有字节码正在执行），它会首先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否活着，如果线程不处于活动状态，则将对象头设置成无锁状态，如果线程仍然活着，拥有偏向锁的栈会被执行，遍历偏向对象的锁记录，栈中的锁记录和对象头的Mark Word要么重新偏向于其他线程，要么恢复到无锁或者标记对象不适合作为偏向锁，最后唤醒暂停的线程。

**轻量级锁**
       轻量级锁加锁：线程在执行同步块之前， JVM会先在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录中，官方称为Displaced Mark Word。然后线程尝试使用 CAS 将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。如果有两条以上的线程争用同一个锁，那轻量级锁就不再有效，要膨胀为重量级锁，锁标志

的状态值变为”10”，Mark Word中存储的就是指向重量级（互斥量）的指针。

 轻量级锁解锁：轻量级解锁时，会使用原子的 CAS 操作来将Displaced Mark Word替换回到对象头，如果成功，则表示没有竞争发生。如果失败，表示当前锁存在竞争，锁就会膨胀成重量级锁。

​    **偏向锁，轻量级锁都是乐观锁，重量级锁是悲观锁**。

一个对象刚开始实例化的时候，没有任何线程来访问它的时候。它是可偏向的，意味着，它现在认为只可能有一个线程来访问它，所以当第一个线程来访问它的时候，它会偏向这个线程，此时，对象持有偏向锁。偏向第一个线程，这个线程在修改对象头成为偏向锁的时候使用CAS操作，并将对象头中的ThreadID改成自己的ID，之后再次访问这个对象时，只需要对比ID，不需要再使用CAS在进行操作。

一旦有第二个线程访问这个对象，因为偏向锁不会主动释放，所以第二个线程可以看到对象时偏向状态，这时表明在这个对象上已经存在竞争了，检查原来持有该对象锁的线程是否依然存活，如果挂了，则可以将对象变为无锁状态，然后重新偏向新的线程，如果原来的线程依然存活，则马上执行那个线程的操作栈，检查该对象的使用情况，如果仍然需要持有偏向锁，则偏向锁升级为轻量级锁，（偏向锁就是这个时候升级为轻量级锁的）。如果不存在使用了，则可以将对象回复成无锁状态，然后重新偏向。

   轻量级锁认为竞争存在，但是竞争的程度很轻，一般两个线程对于同一个锁的操作都会错开，或者说稍微等待一下（自旋），另一个线程就会释放锁。 但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁膨胀为重量级锁，重量级锁使除了拥有锁的线程以外的线程都阻塞，防止CPU空转。

https://pic1.zhimg.com/v2-8f405804cd55a26b34d59fefc002dc08_r.jpg

**判断对象头的Mark Word，判断锁标志位，判断是否持有偏向锁（偏向锁在Mark Word中设置ThreadID,其它线程可以看到），没有则判断持有偏向锁的线程是否存活，如果持有偏向锁的线程仍然存活且仍需要偏向锁则锁升级为轻量级锁（否则将对象回复成无锁状态并重新偏向），当轻量级锁CAS自旋竞争到一定次数则升级为重量级锁(线程竞争不自旋，阻塞，不消耗CPU)**

### Thread

#### 线程优先级

java线程通过priority(内置int变量)控制优先级，通过方法setPriority(int)修改，但操作系统可以完全不理会优先级的设定，因此不能通过控制priority来控制java线程的执行顺序与执行时间。

#### 线程状态

- NEW：线程被构建，但未调用start()方法
- RUNNABLE：运行状态
- BLOCKED:阻塞，需要拿到锁
- WAITING：等待，需要被唤醒，调用wait()方法即进入
- TIME_WAITING：超时等待，可在超时时自动返回,即正在等待执行中
- TERMINATED：终止，当前线程执行完毕

通过调用getState()返回线程当前状态，返回值为Thread.State,一个枚举类，继java.lang.Enum<Thread.State>。

#### Daemon线程

​    支持性线程，也可看作是守护线程，即只是为了支持其他线程工作而启动的线程，调用setDaemon(boolean)方法设置，当进程中没有除Daemon线程外的其它任何线程时,jvm将退出，Daemon线程也立即终止，因此Daemon不能依靠finally确保执行关闭。

#### 中断Interrupted

中断用于终止当前线程，或通过控制终止其它线程

​    调用interrupt()设置中断(当前线程中断),调用isInterrupted()判断当前线程是否中断，如果中断，终止或抛出InterruptedException则返回假 ，Thread.interrupted()用于测试当前线程是否中断。

可通过设置interrupt()或内部boolean变量安全的终止线程。

```java
package chapter4_threads;

public class Shutdown {
    public static void main(String[] args){
        Runner a=new Runner("A");
        Runner b=new Runner("B");

        a.start();
        b.start();
        SleepUtils.second(2);
        //通过设置interrupt()终止线程
        a.interrupt();
        SleepUtils.second(2);
        //通过内置boolean变量终止线程
        b.shutdown();
    }

}
class Runner extends Thread{
    private long i=0;//计数
    private volatile boolean on=true;//用于控制

    public Runner(String name) {
        super(name);
    }

    @Override
    public void run(){
        while (on&&!Thread.currentThread().isInterrupted()){
            i++;
        }
        System.out.println(Thread.currentThread().getName()+":"+i);
    }

    public void shutdown(){
        on=false;
    }
}
```

#### 等待/通知机制

​      线程A,B通过对象O进行通信。A的执行需要某些条件，执行到某一步，条件满足就继续执行，否则先暂停直到条件满足，如何感受到条件变化?通过中间条件O进行控制，A拿到O的锁，条件不满足则调用wait()方法进入等待队列同时释放O的锁，B拿到锁后进行执行，到某一步调用notify()/notifyAll()，到释放锁后就通知等待队列中的某个线程/所有线程从等待队列进入到同步队列，重新竞争锁并从wait()方法之后继续执行。线程同步用到该机制。

典型的经典范式为消费者/生产者模式

**消费者:                       生产者:**

**synchronized(O){                    synchronized(O){**

  **while(条件不满足){                   改变条件**

​    **O.wait();                         O.notify()/O.notifyAll();**      

  **}                                }**    

  **statement...**

**}**

```java
public class WaitNotify {
    public static Object o=new Object();
    public static boolean control=false;
    public static void main(String[] args){
        Thread a=new Thread(new A(),"A");
        Thread b=new Thread(new B(),"B");

        a.start();
        SleepUtils.second(2);
        b.start();
    }

    static class A implements Runnable{
        private int i=0;
        @Override
        public void run(){
            //拿到o的锁,A等待,条件满足时通知
            synchronized (o){
                try {
                    i++;
                    //A线程拿到o的锁并等待通知，感受到通知就返回
                    while(!control){
                        o.wait();//达不到条件就等待,调用完wait()方法就会释放锁,进入等待队列，感受到通知进入同步队列，重新竞争锁，拿到锁之后从wait()向下继续执行
                        i++;
                    }
                }catch (InterruptedException e){
                    System.out.println(Thread.currentThread().getName()+"-中断");
                }
            }
            System.out.println("A-"+i);
        }
    }

    static class B implements Runnable{
        @Override
        public void run(){
            synchronized (o){
                //B拿到o的锁，进行通知
                o.notify();
                control=true;
            }
        }
    }
}
```

#### 线程间通信-管道输入/输出流

线程间通过管道PipedWriter/PipedReader传递信息，媒介为内存，使用时必须进行连接

out.connect(in);即PipedWriter对象必须调用connect()方法连接PipedReader对象

线程A中使用PipedWriter对象向管道中写入，线程B中使用PipedReader对象从管道中读取数据，完成线程间的通信

```java
import java.io.IOException;
import java.io.PipedReader;
import java.io.PipedWriter;

/**
 * 线程间通过管道PipedWriter/PipedReader传递信息，媒介为内存，使用时必须进行连接
 * out.connect(in);
 */
public class Piped {

    public static void main(String[] args){
        PipedReader in=new PipedReader();
        PipedWriter out=new PipedWriter();

        try{
            //管道必须进行连接，否则抛出异常
            out.connect(in);
            Thread printThread=new Thread(new Print(in),"printThread");
            printThread.start();

            int receive=0;
            while((receive=System.in.read())!=-1){
                out.write(receive);//写入管道
            }

            out.close();//关闭
        }catch (IOException e){
            System.out.println(Thread.currentThread().getName()+"IO异常");
        }
    }
    
    static class Print implements Runnable{
        private PipedReader in;
        public Print(PipedReader in){
            this.in=in;
        }

        public void run(){
            int receive=0;
            try{
                while((receive=in.read())!=-1){//从管道中接收
                    System.out.println((char)receive);
                }
            }catch (IOException e){
                System.out.println(Thread.currentThread().getName()+"IO异常");
            }

        }
    }
}
```

#### Thread对象的join()方法

join()方法，在线程A中使用B.join()则A先等待，直到B线程结束才返回从A继续开始执行

#### 线程局部变量ThreadLocal<T\>

ThreadLocal,一个泛型类，提供线程局部变量 ,ThreadLocal实例通常是希望将状态与线程关联的类中的私有静态字段（例如，用户ID或事务ID）。

```java
package chapter4_threads;

import java.util.concurrent.TimeUnit;

/**
 * ThreadLocal,一个泛型类，提供线程局部变量
 * ThreadLocal实例通常是希望将状态与线程关联的类中的私有静态字段（例如，用户ID或事务ID）。
 */
public class Threadlocal {
    static ThreadLocal<Long> TIME_THREADLOCAL=new ThreadLocal<>();
    public static void main(String[] args){
        begin();
        try{
            TimeUnit.SECONDS.sleep(1);
        }catch (InterruptedException e){}

        System.out.println("经过时间:"+end());
    }

    //设置开始时间
    private static void begin(){
        TIME_THREADLOCAL.set(System.currentTimeMillis());
    }
    //返回经过的时间差
    private static long end(){
        return System.currentTimeMillis()-TIME_THREADLOCAL.get();
    }
}
```





