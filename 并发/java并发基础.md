## 基本概念

java代码在javac编译后会变成Java字节码，字节码被类加载器加载到JVM里，JVM执行字节码，最终需要转化为汇编指令在CPU上执行，Java中所使用的并发机制依赖于JVM的实现和CPU的指令。

* 并发：指多个任务交替执行。
* 并行：多个任务在绝对时间上同时执行
* 同步：同步方法调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为。
* 异步：异步方法调用更像一个消息传递，一旦开始，方法调用就会立即返回，调用者就可以继续后续的操作。而异步方法通常会在另外一个线程中“真实”地执行。整个过程，不会阻碍调用者的工作。
* 临界区：临界区用来表示一种公共资源/共享数据，每次只能有一个线程进入临界区，其它线程必须等待

### 关键字volatile

 volatile用于声明变量(字段)使其成为共享变量。用关键字volatile声明的字段每次被使用时系统都会将该字段写回到内存，同时使缓存中的该字段失效(即该字段可能保存在缓存中，系统将该字段写回到内存并使缓存失效),这样，每当线程需要使用该字段时都需要从内存中读取该字段，保证了该字段的唯一可见性。**注意,volatile只保证了内存可见性，并不保证原子性，volatile适用于get/set，并不适用于getAndOperate。**

volatile的作用有两个：

1. 保证变量的内存可见性
2. 禁止指令重排序

**现代操作系统采用时分的方式调度运行的程序，系统为每个线程分配时间片，当时间片用完了就发生线程的调度**。多核处理器当然可以同时处理多个任务，但总内存还是只有那么一块。因此在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中数据读到处理器缓存里。

### 关键字synchronized

synchronized可用于声明方法，同时可以锁住对象。

- 对于普通同步方法，锁住当前实例对象。
- 对于静态同步方法，锁住当前类的Class对象。
- 对于同步方法块，锁住Synchonized括号里配置的对象。

**synchronized基于指令monitorenter和monitorexit实现，是java提供的基本上锁机制。**

每一个对象都有自己的监视器(monitor),获取锁就相当于拿到该对象的监视器进入同步方法/方法块，否则进入同步队列，陷入BLOCKED(阻塞)状态，当监视器退出后重新竞争锁。

```java
01 public class AccountingSync2 implements Runnable{
02     static AccountingSync2 instance=new AccountingSync2();
03     static int i=0;
04     public synchronized void increase(){
05         i++;
06     }
07     @Override
08     public void run() {
09         for(int j=0;j＜10000000;j++){
10             increase();
11         }
12     }
13     public static void main(String[] args) throws InterruptedException {
14         Thread t1=new Thread(instance);
15         Thread t2=new Thread(instance);
16         t1.start();t2.start();
17         t1.join();t2.join();
18         System.out.println(i);
19     }
20 }
```

上述代码中，synchronized关键字作用于一个实例方法。这就是说在进入increase()方法前，线程必须获得当前对象实例的锁。在本例中就是instance对象。

一种错误的同步方式如下：

```java
01 public class AccountingSyncBad implements Runnable{
02     static int i=0;
03     public synchronized void increase(){
04         i++;
05     }
06     @Override
07     public void run() {
08         for(int j=0;j＜10000000;j++){
09             increase();
10         }
11     }
12     public static void main(String[] args) throws InterruptedException {
13         Thread t1=new Thread(new AccountingSyncBad());
14         Thread t2=new Thread(new AccountingSyncBad());
15         t1.start();t2.start();
16         t1.join();t2.join();
17         System.out.println(i);
18     }
19 }
```

上述代码就犯了一个严重的错误。虽然在第3行的increase()方法中，申明这是一个同步方法。但很不幸的是，执行这段代码的两个线程都指向了不同的Runnable实例。由第13、14行可以看到，这两个线程的Runnable实例并不是同一个对象。因此，线程t1会在进入同步方法前加锁自己的Runnable实例，而线程t2也关注于自己的对象锁。换言之，这两个线程使用的是两把不同的锁。因此，线程安全是无法保证的。

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

![](https://pic1.zhimg.com/v2-8f405804cd55a26b34d59fefc002dc08_r.jpg)

**判断对象头的Mark Word，判断锁标志位，判断是否持有偏向锁（偏向锁在Mark Word中设置ThreadID,其它线程可以看到），没有则判断持有偏向锁的线程是否存活，如果持有偏向锁的线程仍然存活且仍需要偏向锁则锁升级为轻量级锁（否则将对象回复成无锁状态并重新偏向），当轻量级锁CAS自旋竞争到一定次数则升级为重量级锁(线程竞争不自旋，阻塞，不消耗CPU)**

## Thread

### 创建线程

常规有两种方式创建线程，继承Thread类并重写run()方法，或者实现Runnable接口然后作为参数传入Thread中，获取Thread实例对象。然后调用start()方法执行线程。

```java
Thread t1=new Thread(){
    @Override
    public void run(){
        System.out.println("Hello, I am t1");
    }
};
t1.start();
```

### 终止线程

Thread提供了stop()方法来终止线程，但是其原理是直接释放对象持有的锁并强制终止，在这种情况下，很有可能导致数据的不一致性。

为了安全的终止线程，可以在线程对象中设置一个boolean变量用于控制线程什么时候终止，只要保证终止前线程的所有操作都是原子操作，那么就可以避免数据的不一致性（最常见的一种情况，写到一半突然没了锁，导致写被覆盖或者写混乱）

### 线程状态与优先级

线程状态有：

- NEW：线程被构建，但未调用start()方法
- RUNNABLE：运行状态
- BLOCKED:阻塞，需要拿到锁
- WAITING：等待，需要被唤醒，调用wait()方法即进入
- TIME_WAITING：超时等待，可在超时时自动返回,即正在等待执行中
- TERMINATED：终止，当前线程执行完毕

通过调用getState()返回线程当前状态，返回值为Thread.State,一个枚举类，即java.lang.Enum<Thread.State>。

**优先级**

java线程通过priority(内置int变量)控制优先级，通过方法setPriority(int)修改，但操作系统可以完全不理会优先级的设定，因此不能通过控制priority来控制java线程的执行顺序与执行时间。

在Java中，使用1到10表示线程优先级。一般可以使用内置的三个静态标量表示：

```
public final static int MIN_PRIORITY = 1;
public final static int NORM_PRIORITY = 5;
public final static int MAX_PRIORITY = 10;
```

### 守护线程Daemon

支持性线程，也可看作是守护线程，即只是为了支持其他线程工作而启动的线程，调用setDaemon(boolean)方法设置，当进程中没有除Daemon线程外的其它任何线程时,jvm将退出，Daemon线程也立即终止，因此Daemon不能依靠finally确保执行关闭。

```java
01 public class DaemonDemo {
02     public static class DaemonT extends Thread{
03         public void run(){
04             while(true){
05                 System.out.println("I am alive");
06                 try {
07                     Thread.sleep(1000);
08                 } catch (InterruptedException e) {
09                     e.printStackTrace();
10                 }
11             }
12         }
13     }
14     public static void main(String[] args) throws InterruptedException {
15         Thread t=new DaemonT();
16         t.setDaemon(true);
17         t.start();
18
19         Thread.sleep(2000);
20     }
21 }
```

在这个例子中，由于t被设置为守护线程，系统中只有主线程main为用户线程，因此在main线程休眠2秒后退出时，整个程序也随之结束。但如果不把线程t设置为守护线程，main线程结束后，t线程还会不停地打印，永远不会结束。

### 中断Interrupted

中断用于终止当前线程，或通过控制终止其它线程，中断不会立即使线程退出，通过设置中断标志位实现中断，Thread中有三个关于中断的方法

```java
public void Thread.interrupt()               // 中断线程
public boolean Thread.isInterrupted()        // 判断是否被中断
public static boolean Thread.interrupted()   // 判断是否被中断，并清除当前中断状态
```

调用interrupt()设置中断(当前线程中断),调用isInterrupted()判断当前线程是否中断，如果中断，终止或抛出InterruptedException则返回假 ，Thread.interrupted()用于测试当前线程是否中断。

对于中断来讲，设置中断的同时必须保证有中断处理的逻辑，因为其内部仅仅是控制中断标志位来控制线程执行，如果没有中断处理逻辑，那么设置中断是没用的

```java
public static void main(String[] args) throws InterruptedException {
    Thread t1=new Thread(){
        @Override
        public void run(){
            while(true){
                Thread.yield();
            }
        }
    };
    t1.start();
    Thread.sleep(2000);
    t1.interrupt();
}
```

在这里，虽然对t1进行了中断，但是在t1中并没有中断处理的逻辑，因此，即使t1线程被置上了中断状态，但是这个中断不会发生任何作用。

```java
Thread t1=new Thread(){
    @Override
    public void run(){
        while(true){
            if(Thread.currentThread().isInterrupted()){
                System.out.println("Interruted!");
                break;
            }
            Thread.yield();
        }
    }
};
```

通过设置interrupt()或内部boolean变量都可以安全的终止线程。

```java
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

### 等待/通知机制

线程A,B通过对象O进行通信。A的执行需要某些条件，执行到某一步，条件满足就继续执行，否则先暂停直到条件满足，如何感受到条件变化?通过中间条件O进行控制，A拿到O的锁，条件不满足则调用wait()方法进入等待队列同时释放O的锁，B拿到锁后进行执行，到某一步调用notify()/notifyAll()，到释放锁后就通知等待队列中的某个线程/所有线程从等待队列进入到同步队列，重新竞争锁并从wait()方法之后继续执行。线程同步用到该机制。

典型的经典范式为消费者/生产者模式

**消费者:                                                                       生产者:**

**synchronized(O){                                                    synchronized(O){**

  **while(条件不满足){                                                      改变条件**

​      **O.wait();                                                                   O.notify()/O.notifyAll();**      

  **}                                                                               }**    

如果一个线程调用了object.wait()，那么它就会进入object对象的等待队列。这个等待队列中，可能会有多个线程，因为系统运行多个线程同时等待某一个对象。当object.notify()被调用时，它就会从这个等待队列中，随机选择一个线程，并将其唤醒。

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

### join与yeild

多时候，一个线程的输入可能非常依赖于另外一个或者多个线程的输出，此时，这个线程就需要等待依赖线程执行完毕，才能继续执行。JDK提供了join()操作来实现这个功能

```java
public final void join() throws InterruptedException
public final synchronized void join(long millis) throws InterruptedException
```

第一个join()方法表示无限等待，它会一直阻塞当前线程，直到目标线程执行完毕。第二个方法给出了一个最大等待时间，如果超过给定时间目标线程还在执行，当前线程也会因为“等不及了”，而继续往下执行。在进程A中使用进程B的join方法即表示进程A会等待进程B执行完毕才继续执行，join是加入的意思，这里就表示要进程B加入到进程A中来

```java
public class JoinMain {
    public volatile static int i=0;
    public static class AddThread extends Thread{
        @Override
        public void run() {
            for(i=0;i＜10000000;i++);
        }
    }
    public static void main(String[] args) throws InterruptedException {
        AddThread at=new AddThread();
        at.start();
        at.join();//主进程会等待AddThread进程结束才继续执行
        System.out.println(i);
    }
}
```

join()的本质是让调用线程wait()在当前线程对象实例上。下面是JDK中join()实现的核心代码片段：

```java
while (isAlive()) {
    wait(0);
}
```

可以看到，它让调用线程在当前线程对象上进行等待。当线程执行完成后，被等待的线程会在退出前调用notifyAll()通知所有的等待线程继续执行。因此，值得注意的一点是：不要在应用程序中，在Thread对象实例上使用类似wait()或者notify()等方法，因为这很有可能会影响系统API的工作，或者被系统API所影响。

**谦让Thread.yield()**

```java
public static native void yield();
```

这是一个静态方法，一旦执行，它会使当前线程让出CPU。但要注意，让出CPU并不表示当前线程不执行了。当前线程在让出CPU后，还会进行CPU资源的争夺，但是是否能够再次被分配到，就不一定了。

如果你觉得一个线程不那么重要，或者优先级非常低，而且又害怕它会占用太多的CPU资源，那么可以在适当的时候调用Thread.yield()，给予其他重要线程更多的工作机会。

### 进程组

在一个系统中，如果线程数量很多，而且功能分配比较明确，就可以将相同功能的线程放置在一个线程组里。

```java
01 public class ThreadGroupName implements Runnable {
02     public static void main(String[] args) {
03         ThreadGroup tg = new ThreadGroup("PrintGroup");
04         Thread t1 = new Thread(tg, new ThreadGroupName(), "T1");
05         Thread t2 = new Thread(tg, new ThreadGroupName(), "T2");
06         t1.start();
07         t2.start();
08         System.out.println(tg.activeCount());
09         tg.list();
10     }
11
12     @Override
13     public void run() {
14         String groupAndName=Thread.currentThread().getThreadGroup().getName()
15                 + "-" + Thread.currentThread().getName();
16         while (true) {
17             System.out.println("I am " + groupAndName);
18             try {
19                 Thread.sleep(3000);
20             } catch (InterruptedException e) {
21                 e.printStackTrace();
22             }
23         }
24     }
25 }
```

activeCount()可以获得活动线程的总数，但由于线程是动态的，因此这个值只是一个估计值，无法确定精确，list()方法可以打印这个线程组中所有的线程信息，对调试有一定帮助。代码中第4、5两行创建了两个线程，使用Thread的构造函数，指定线程所属的线程组，将线程和线程组关联起来。

线程组还有一个值得注意的方法stop()，它会停止线程组中所有的线程。这看起来是一个很方便的功能，但是它会遇到和Thread.stop()相同的问题，因此使用时也需要格外谨慎。



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

## 并发常见错误

### 并发下的ArrayList

ArrayList是线程不安全容器，问题就出在ArrayList的内部是线上，当多个线程并发执行调用同一个ArrayList对象时就有可能导致数据被覆盖，要么结果不对，要么抛出异常

```java
public class ArrayListMultiThread {
    static ArrayList＜Integer＞ al = new ArrayList＜Integer＞(10);
    public static class AddThread implements Runnable {
        @Override
        public void run() {
            for (int i = 0; i ＜ 1000000; i++) {
                al.add(i);
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(new AddThread());
        Thread t2=new Thread(new AddThread());
        t1.start();
        t2.start();
        t1.join();t2.join();
        System.out.println(al.size());
    }
}
```

问题并不出在调用add()方法上（逻辑上这好像是没错的），而是出现在add()方法内部，ArrayList的底部实现上

线程安全的List容器可以使用Vector

### 并发下的HashMap

```java
public class HashMapMultiThread {

    static Map＜String,String＞ map = new HashMap＜String,String＞();

    public static class AddThread implements Runnable {
        int start=0;
        public AddThread(int start){
            this.start=start;
        }
        @Override
        public void run() {
            for (int i = start; i ＜ 100000; i+=2) {
                map.put(Integer.toString(i), Integer.toBinaryString(i));
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(new HashMapMultiThread.AddThread(0));
        Thread t2=new Thread(new HashMapMultiThread.AddThread(1));
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(map.size());
    }
}
```

在1.7中由于put使用头插法会导致链表结构被破坏，导致死循环