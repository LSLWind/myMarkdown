## 基础

要保证线程安全，保证数据的并发一致性，不仅原子引用要线程安全，复合代码块同样要保证线程安全，即整个状态的一致性，而不是某个变量，某个对象的一致性。同理，对于同步来说，即使方法是线程安全的，调用方法的代码块仍可能不是线程安全的，仍需保证对调用方法的代码块的线程安全，例如

```java
if(!vector.contains(element))vector.add(element);
```

synchronized相当于java内置的互斥锁，



### 并发包concurrent

#### 接口Lock

Lock的用法为

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
}finally {
    lock.unlock();
}
```

Lock内部只有6个方法，其中获取锁lock()与释放锁unlock()最常用，都是非static void方法。

        并发包java.util.concurrent.locks提供了多种实现该接口的锁，但是其内部本身并没有使用synchronized上锁，而是使用CAS将进程加入同步队列，从而锁住代码块(或者说进入临界区/操作临界资源)（场景为多个(实现Runnable的)线程对象要竞争同一块代码块的操作），实现代码块的并发控制访问。

#### 队列同步器(同步器)AbstractQueuedSynchronizer(AQS)

​        有了接口，就要实现锁，我们通常使用的锁是基于队列同步器实现的，AbstractQueuedSynchronizer是用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作。简而言之，同步器是给锁的构建者准备的，通过继承该类，可以通过重写方法来实现自己想要的锁。

        队列同步器(同步器)AbstractQueuedSynchronizer内部已经帮我们写好了实现一个锁的基本方法，通过内部维护一个继承同步器AQS的类，调用该类的方法实现接口可以很轻松的自己写一个锁。同步器通过内部维护一个int(名为state)表示锁状态。同步器提供的一些基本方法可以帮我们实现接口Lock，主要有

* .getState()：获取当前同步状态。                                         
* ·setState(int newState)：设置当前同步状态。
* ·compareAndSetState(int expect,int update)：使用CAS设置当前状态，该方法能够保证状态
  设置的原子性。
* .acquire(1):上锁
* .release(1):释放锁

因此，实际的上锁(释放锁)操作还是通过AQS内部的方法来完成的。

在功能上，AQS可以完成两个操作，独占或者共享，独占即排它锁，共享即可共同读但同一时间只能有一个写。不同实现可能还会分公平锁与非公平锁，区别就是直接拿锁还是进入队列按照队列顺序拿锁。在具体实现中一般是公平锁拿锁的时候先排队在拿锁，而非公平锁则先拿一次锁，如果不成功再排队。

锁实现Lock接口，内部维护一个Sync,该类继承自AbstractQueuedSynchronizer，而AQS主要目的是维护队列及队列中的线程状态。

##### 独占式获取锁lock

使用lock()时：

```java
public void lock() { 
   sync.acquire(1);
 }
```

lock()函数由内部调用Sync的acquire(1)来实现

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

​       先通过tryAcquire(arg)尝试获取锁，如果获取不到则调用addWaiter(Node.EXCLUSIVE), arg)将当前线程变为一个"waiter"并将当前线程加入线程队列，然后通过acquireQueued一直从队列中请求获取锁。tryAcquire获取到锁则线程执行代码，否则acquireQueued将当前线程加入等待队列，acquireQueued中会一直自旋直到获取锁，返回中断标志位，如果为true表明线程已中断，则唤醒当前线程。

tryAcquire(int arg)尝试直接获取锁：

```java
protected boolean tryAcquire(int arg){
    throws UnsupportedOperationException();
}
```

​        AQS中的该方法留空了，因为AQS有两种功能，此处通过该方法自行设置锁是独占还是共享。（既然要给子类实现，应该用抽象方法，但是面向两种使用场景，需要给子类定义的方法都是抽象方法会导致子类无论如何都需要实现另外一种场景的抽象方法，显然，这对子类来说是不友好的。）

在ReentrantLock(可重入锁)中的FairSync（公平锁）中，tryAcquire实现:

```java
 protected final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();// 获取当前线程 
            int c = getState();  // 获取父类 AQS 中的标志位 
            if (c == 0) {
                if (!hasQueuedPredecessors() && 
                    // 如果队列中没有其他线程  说明没有线程正在占有锁！
                    compareAndSetState(0, acquires)) { 
                    // 修改一下状态位
                    setExclusiveOwnerThread(current);
                    // 将当前线程设置到 AQS 的一个变量中，说明这个线程拿走了锁（独占）。
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
             // 如果不为 0 意味着，锁已经被拿走了，但是，因为 ReentrantLock 是重入锁，
             // 是可以重复 lock,unlock 的，只要成对出现行。一次。这里还要再判断一次 获取锁的线程是不是当前请求锁的线程。
                int nextc = c + acquires;// 如果是的，累加在 state 字段上就可以了。
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);//设置状态
                return true;
            }
            return false;
        }
```

​        思路很明确，先看是否有线程持有锁，如果没有(state=0)，那么通过cas设置state，并将当前线程置位持有锁的线程，否则看看是否是当前线程持有锁，如果是则将state加上acquires表示已重入锁，否则获取锁失败。

        可以看到，AQS内部维护的state表示实际上的“锁”，通过cas操作保证原子性，通过加上acquire保证锁的可重入性(各种实现中,acquires是方法acquire()传进来的，为方便释放锁，通常为1)。

如果获取锁失败，则要将当前线程构造成waiter加入等待队列中

addWaiter(Node mode)的代码为：

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);//将当前线程以模式mode绑定到Node
    //先使用快速入列法来尝试一下,如果失败,则进行更加完备的入列算法.
    Node pred = tail; //列尾指针
    if (pred != null) {
        node.prev = pred; //步骤1:该节点的前趋指针指向tail
        if (compareAndSetTail(pred, node)){ //步骤二:cas将尾指针指向该节点
            pred.next = node;//步骤三:如果成功,让旧列尾节点的next指针指向该节点.
            return node;
        }
    }
    //cas失败,或在pred == null时调用enq
    enq(node);
    return node;
}
```

​        用当前线程去构造一个 Node 对象，mode 是一个表示 Node 类型的字段，仅仅表示这个节点是独占的，还是共享的，或者说，AQS 的这个队列中，哪些节点是独占的，哪些是共享的。队列是一个双向链表。

如果一次cas不成功，则调用enq尝试进行cas循环操作来达到设置尾指针的目的。

```java
private Node enq(final Node node) {
    for (;;) { //cas无锁算法的标准for循环,不停的尝试
        Node t = tail;
        if (t == null) { //初始化
            if (compareAndSetHead(new Node())) 
              //需要注意的是head是一个哨兵的作用,并不代表某个要获取锁的线程节点
                tail = head;
        } else {
            //和addWaiter中一致,不过有了外侧的无限循环,不停的尝试,自旋锁
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

​        思路也很明确，将当前线程绑定到一个Node上并设置mode，然后通过cas操作将当前线程设置为tail加入队列。如果尾部为null，表示第一次插入，则先插入一个“哨兵”，再来cas。因此保证整个队列的结构为头结点为哨兵，head之后才为实际的线程节点。

head，tail是AQS中维护的头尾节点。

private transient volatile Node head;
private transient volatile Node tail;
private volatile long state;
mode只有两种模式，独占或者共享，在Node中分别是null或者new Node()（这种区别方式还是头一次见）。

Node的结构为：

```java
static final class Node {
        static final int CANCELLED =  1;//1 ：当前节点被取消或者中断
        static final int SIGNAL    = -1;//-1：下一个节点需要被释放
        static final int CONDITION = -2;//-2：当前节点正处在等待队列中
        static final Node SHARED = new Node();//共享模式
        static final Node EXCLUSIVE = null;//独占模式

    volatile int waitStatus;//当前节点的状态
     
    volatile Node prev;//前一个节点
     
    volatile Node next;//下一个节点
     
    volatile Thread thread;//节点绑定的线程
     
    Node nextWaiter;//节点的等待模式，共享或独占
     
    final boolean isShared() {
        return nextWaiter == SHARED;
    }
     
    //返回前一个节点
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }
    //构造方法
    Node() {    // Used to establish initial head or SHARED marker
        
    }
    //指定构造方法
    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }
     
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }

}
```

Node是AQS内部维护的私有队列（双向链表）的节点，通过内部维护的head,tail表示队列（双向链表）。

将没能获取到锁的线程加入等待队列并不停自选的acquireQueued(final Node node, int arg)代码为：

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                //一直循环，要么获取锁，要么挂起
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
             // 如果当前的节点是 head 说明他是队列中第一个“有效的”节点（哨兵），因此尝试获取。
                    setHead(node);//设置头结点
                    p.next = null; 
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) && 
                // 否则，检查前一个节点的状态，看当前获取锁失败的线程是否需要挂起。
                    parkAndCheckInterrupt()) 
               // 如果需要，借助 JUC 包下的 LockSopport 类的静态方法 Park 挂起当前线程，直到被唤醒。
                    interrupted = true;
            }
        } finally {
            if (failed) // 如果有异常 
                cancelAcquire(node);// 取消请求，对应到队列操作，就是将当前节点从队列中移除。
        }
    }
```

​        根据上面所说的AQS的队列结构，head并不为真正的需要出队列的线程节点，而是一个“哨兵”，因此只要节点node的前驱为head就表示当前节点可以尝试获取锁（出队列），否则就代表获取锁失败(不能出队列)，看看是否需要挂起。如果挂起了，就代表进行中断，返回（true）之后再来一次selfInterrupt()唤醒中断的线程，否则没挂起直接返回（false）。

        可以看到AQS内部大量使用CAS操作控制state，head，tail，通过维护state状态值（锁）管理线程出入队列，队列是双向链表，对象是内部维护的Node，Node与当前线程绑定，在逻辑上完成了锁的功能（代码块的并发控制）。对线程的挂起及唤醒操作是通过使用 UNSAFE 类调用 JNI 方法实现的。当然，还提供了挂起指定时间后唤醒的 API。同时AQS内部也使用了java.util.concurrent.locks中的LockSupport，LockSupport中只有static方法，与当前线程绑定，可调用park()与unpark()阻塞/唤醒线程。

##### 独占式释放锁unlock()

释放锁unlock()的过程为：

内部还是调用了release(1)，而release(1)则内部调用tryRelease(1)，tryRelease()同样需要子类自己去实现。

在ReentrantLock(可重入锁)中的FairSync（公平锁）中，tryRelease实现:

```java
protected final boolean tryRelease(int releases){
	int c=getState()-releases;//自减可重入
	boolean free=false;
	//省略一小段
	if(c==0){
		free=true;
		setExclusiveOwnerThread(null);
	}
	setState(c);
	return free;
}
```

可以看到可重入锁，需要将状态减为0时才表示当前线程释放锁

##### 共享式同步状态获取与释放

共享式即多个线程可同时访问公共资源，如读操作

```java
   public final void acquireShared(long arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
```

尝试获取共享锁，如果失败，则自旋尝试获取同步状态，同理tryAcquireShared(arg)交给子类自己实现，这里实现将线程加入等待队列并不断自旋尝试获取锁的过程。

```java
  private void doAcquireShared(long arg) {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    long r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

##### 总结

AQS是用于构建锁的基本工具，因此支持共享/独占式，公平/非公平锁结构，而具体实现留给子类（tryAcquire()与tryRelease()留空，让子类实现获取锁的CAS方式，自己实现加入等待队列，不断自旋尝试获取锁的过程），因此有了多种多样的锁，适用于不同的状况之下

总结一下AQS获取锁过程（以独占式为例）：

1. AQS内部维护状态标志位state，state=0时表示当前线程可进入临界区，内部维护线程队列，通过CAS原子操作控制state与线程队列
2. 当前线程尝试获取锁，即先判断state是否为0，不为0表示获取锁失败，将当前线程构建为node并CAS加入队尾，先一次CAS快速尝试，否则自旋CAS先判断tail是否为null，否则初始化队列，加入一个哨兵节点，然后CAS将当前node加入队尾
3. 当前线程在等待队列中不断CAS判断自己前驱是否为队头，如果前驱为队头则CAS尝试重置state状态位（即CAS获取锁），获取锁成功置自己为head
4. 释放锁时，先尝试置state位0，然后唤醒head的后继节点，使队列中下一个节点能够自旋获取锁
5. 头结点拥有同步状态
6. 内部维护线程队列时使用了挂起/唤醒线程的操作，用到了LockSupport类

##### 使用AQS实现锁Demo

```java
package concurrent;


import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.AbstractQueuedSynchronizer;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

/**
 * 实现一个自定义共享锁
 * 即自定义锁标志与如何控制锁，AQS底部维护线程队列与state，如果线程获取锁失败自动加入线程队列并自旋尝试获取锁
 */
public class TwinsLock extends AbstractQueuedSynchronizer implements Lock {
    private final Sync sync=new Sync(2);
    private static class Sync extends AbstractQueuedSynchronizer{
        Sync(int count){
            if(count<=0)throw new IllegalArgumentException("count必须大于0");
            setState(count);//设置状态位
        }

        //CAS尝试获取锁
        @Override
        public int tryAcquireShared(int reduceCount){
            for(;;){
                int current=getState();//获取状态位
                int newCount=current-reduceCount;
                if(newCount<0||compareAndSetState(current,newCount)){
                    return newCount;
                }
            }
        }
        //CAS释放锁
        @Override
        public boolean tryReleaseShared(int returnCount){
            for(;;){
                int current=getState();
                int newCount=current+returnCount;
                if(newCount>2)throw new IllegalArgumentException("必须先lock后unlock");
                if(compareAndSetState(current,newCount)){
                    return true;
                }
            }
        }
    }

    @Override
    public void lock(){
        sync.acquireShared(1);
    }
    @Override
    public void unlock(){
        sync.releaseShared(1);
    }


    @Override
    public void lockInterruptibly() throws InterruptedException {

    }
    @Override
    public boolean tryLock() {
        return false;
    }
    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return false;
    }
    @Override
    public Condition newCondition() {
        return null;
    }
}

```



```java
package concurrent;

import org.junit.Test;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;

public class TwinsLockTest {
    @Test
    public void test(){

        final Lock lock=new TwinsLock();

        for(int i=0;i<10;i++){
            Thread thread = new Thread() {
                @Override
                public void run() {
                    try {
                        lock.lock();
                        TimeUnit.SECONDS.sleep(1);
                        System.out.println(Thread.currentThread().getName());
                        TimeUnit.SECONDS.sleep(1);
                    } catch (Exception e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                }
            };

            thread.setName("线程"+i);
            thread.setDaemon(true);
            thread.start();
        }

        for (int i=0;i<10;i++){
            try{
                TimeUnit.SECONDS.sleep(2);
                System.out.println();
            }catch (Exception e){
                e.printStackTrace();
            }
        }

    }

}

```

### 辅助类CountDownLatch

位于java本身提供的并发包concurrent中

```java
package MyStudy;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

/**
 * CountDownLatch,允许一个或多个线程等待直到在其他线程中执行完成并返回继续执行的同步辅助类。
 * 其内部维护一个计数器，构造器初始传值，当前线程使用CountDownLatch对象的await()方法时会进入阻塞状态直到计数器变为0
 * 内部只有五个方法
 * void await()：使当前线程等到锁存器计数到零，除非线程是 interrupted 。
 * boolean await(long timeout, TimeUnit unit)：使当前线程等待直到锁存器计数到零为止，除非线程为 interrupted或指定的等待时间过去。
 * void countDown()：减少锁存器的计数，如果计数达到零，释放所有等待的线程。
 * long getCount()：返回当前计数。
 * String toString()：返回一个标识此锁存器的字符串及其状态。
 * 该类可用于控制线程间的执行顺序(如几个线程同时开始执行，或先后执行)
 */
public class CountDownLatchPractice {
    private static CountDownLatch start=new CountDownLatch(1);//计数

    public static void main(String[] args){
        Thread t=new Thread(new Test(),"Test");
        t.start();

        try{
            System.out.println(System.currentTimeMillis());
            TimeUnit.SECONDS.sleep(1);
            start.countDown();//计数器减一,main先执行,Test才能执行
        }catch (InterruptedException e){
            System.out.println(Thread.currentThread().getName()+"-中断");
        }
    }


    static class Test implements Runnable{
        @Override
        public void run(){
            try{
                start.await();//当前线程会被阻塞直到内部计数器为0
                System.out.println(System.currentTimeMillis());
            }catch (InterruptedException e){
                System.out.println(Thread.currentThread().getName()+"-中断");
            }
        }
    }
}
```



#### 接口Callable/Future/FutureTask

接口Callable作用与Runnable相同，用于给线程提供执行方法，不过Runnable的run()没有返回值，而Callable中的call()有返回值V并且可以抛出异常，一个泛型接口，传进来的类型就是可以返回的类型，因为线程异步执行，无法直接从其他线程中获取返回结果，因此新增Future，新增5个方法监视目标线程调用call的情况

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);//取消任务
    boolean isCancelled();//表示任务是否被取消
    boolean isDone();//表示任务是否已经完成
    V get() throws InterruptedException, ExecutionException;//获取call()返回值
    V get(long timeout, TimeUnit unit)//同上，超时返回null
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

FutureTask是Future的一个实现，实现了Runnable接口与Future接口，因此可直接传入Thread（）中执行。

使用时，首先一个类实现Callable<V\>，交给线程池executor执行，返回结果变为Future<T\>，通过get()获取异步执行结果。也可以直接使用FutureTask，交给线程池（因为FutureTask实现了Runnable，因此也可以直接交给Thread）。

第一种：Callable+Future:（需要交给线程池执行，返回Future）

```java
import java.util.concurrent.*;
public class FutureExample {
    static class MyCallable implements Callable<String> {
        @Override
        public String call() throws Exception {
            System.out.println("do something in callable");
            Thread.sleep(5000);
            return "Ok";
        }
    }
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        Future<String> future = executorService.submit(new MyCallable());
        System.out.println("do something in main");
        Thread.sleep(1000);
        String result = future.get();
        System.out.println("result: " + result);
    }
}
```

第二种：FutureTask

```java
import java.util.concurrent.*
public class FutureTaskWithExecutorService {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        FutureTask<String> futureTask = new FutureTask<>(() -> { 
            System.out.println("do something in callable");
            Thread.sleep(5000);
            return "Ok";
        });
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.submit(futureTask);
        System.out.println("do something in main");
        Thread.sleep(1000);
        String result = futureTask.get();
        System.out.println("result: " + result);
        /*xxxxxxxxx-------或者直接交给Thread--------xxxxxxxxxxxxxx
        new Thread(futureTask).start();
        System.out.println("do something in main");
        Thread.sleep(1000);
        String result = futureTask.get();
        System.out.println("result: " + result);*/
    }
}
```

#### 线程池框架Executor

java中的线程池框架为Executor，工作定义为Runnable/Callable/Future/FutureTask，工作执行为ExecutorService/ThreadPoolExecutor/ScheduledThreadPoolExecutor

线程池的原理为：

1.将Runnble或Callable划分为任务，Runnable execute()执行，无返回值，Callable submit()提交，有返回值，判断当前线程数是否小于核心线程数corePool，小于则创建新的线程执行任务，否则加入任务队列runnableTaskQueue

2.如果等待队列已满，则尝试创建新的线程执行，线程数量不能超过最大线程数量

3.如果超过最大线程数量，则拒绝任务，调用饱和算法，或抛出异常，或直接丢弃等

Executor将执行线程封装成Worker，如果任务小于corePool则直接创建线程执行，Worker执行完一个任务后不断(循环)从等待队列中拿任务来执行

任务队列runnableTaskQueue的主要类型有：

* ArrayBlockingQueue：基于数组的有界阻塞队列，FIFO
* LinkedBlockingQueue：基于链表的阻塞队列，FIFO
* SynchronousQueue：不存储元素的阻塞队列
* PriorityBlockingQueue：具有优先级的无限阻塞队列

创建线程池类型（线程池实现）主要有：ThreadPollExecutor与ScheduledThreadPollExecutor，Executor提供静态方法（由工厂类Executors创建线程池）返回不同类型的线程池：

* FixedThreadPool：创建具有固定大小数目的线程池
* SingleThreadExecutor：只使用单个线程的线程池
* CachedThreadExecutor：大小无界的线程池，适合执行很多短期异步执行任务
* ScheduledThreadPoolExecutor：包含多个线程的ScheduledThreadPoolExecutor
* SingleThreadScheduledExecutor：只包含一个线程的ScheduledThreadPoolExecutor

线程池关闭调用shutdown或者shutdownNow，都是遍历工作线程比中断，shutdown类型为void，shutdownNow返回List<Runnable\>

即具体的过程为：

1. Executors工厂方法返回线程池ExecutorService
2. 使用execute()提交Runnable或者使用submit()提交Callable，返回Future或者使用FutureTask
3. 关闭线程池

```java
    private static volatile int count=0;
    public static void main(String[] args){
        //创建Callable
        Callable<Integer>[] callables=new Callable[1000];
        for(int i=0;i<1000;i++){
            callables[i]=new Callable<Integer>() {
                @Override
                public Integer call() throws Exception {
                    synchronized (this){
                        count++;
                        System.out.println(Thread.currentThread().getName());
                    }
                    return count;
                }
            };
        }
        //创建线程池
        ExecutorService executor=Executors.newFixedThreadPool(5);
        for(int i=0;i<1000;i++){
            //提交任务给线程池执行
            Future<Integer> future=executor.submit(callables[i]);
            try{
                System.out.println(i+" "+future.get());
            }catch (Exception e){
                e.printStackTrace();
            }
            i++;
            FutureTask<Integer> futureTask=new FutureTask<>(callables[i]);
            executor.submit(futureTask);
            try {
                System.out.println(i+" "+futureTask.get());
            }catch (Exception e){
                e.printStackTrace();
            }
        }
		//线程池关闭
        executor.shutdown();
    }
```

 ![img](https://pic2.zhimg.com/v2-9ca281dbd00fe6697a398e4ab115a1bd_b.jpg) 

 ![img](https://pic3.zhimg.com/v2-9793d8ea54f9a9059268e6a0de457326_b.jpg) 

#### 