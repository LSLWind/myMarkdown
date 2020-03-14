## 基础

要保证线程安全，保证数据的并发一致性，不仅原子引用要线程安全，复合代码块同样要保证线程安全，即整个状态的一致性，而不是某个变量，某个对象的一致性。同理，对于同步来说，即使方法是线程安全的，调用方法的代码块仍可能不是线程安全的，仍需保证对调用方法的代码块的线程安全，例如

```java
if(!vector.contains(element))vector.add(element);
```

上述代码就有可能导致add两个element

### 接口Lock

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

### 队列同步器(同步器)AbstractQueuedSynchronizer(AQS)

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

### 独占式获取锁lock

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

### 独占式释放锁unlock()

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

### 共享式同步状态获取与释放

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

### 总结

AQS是用于构建锁的基本工具，因此支持共享/独占式，公平/非公平锁结构，而具体实现留给子类（tryAcquire()与tryRelease()留空，让子类实现获取锁的CAS方式，自己实现加入等待队列，不断自旋尝试获取锁的过程），因此有了多种多样的锁，适用于不同的状况之下

总结一下AQS获取锁过程（以独占式为例）：

1. AQS内部维护状态标志位state，state=0时表示当前线程可进入临界区，内部维护线程队列，通过CAS原子操作控制state与线程队列
2. 当前线程尝试获取锁，即先判断state是否为0，不为0表示获取锁失败，将当前线程构建为node并CAS加入队尾，先一次CAS快速尝试，否则自旋CAS先判断tail是否为null，否则初始化队列，加入一个哨兵节点，然后CAS将当前node加入队尾
3. 当前线程在等待队列中不断CAS判断自己前驱是否为队头，如果前驱为队头则CAS尝试重置state状态位（即CAS获取锁），获取锁成功置自己为head
4. 释放锁时，先尝试置state位0，然后唤醒head的后继节点，使队列中下一个节点能够自旋获取锁
5. 头结点拥有同步状态
6. 内部维护线程队列时使用了挂起/唤醒线程的操作，用到了LockSupport类

### 使用AQS实现锁Demo

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

## 重入锁ReentrantLock

```java
01 public class ReenterLock implements Runnable{
02     public static ReentrantLock lock=new ReentrantLock();
03     public static int i=0;
04     @Override
05     public void run() {
06         for(int j=0;j＜10000000;j++){
07             lock.lock();
08             try{
09                 i++;
10             }finally{
11                 lock.unlock();
12             }
13         }
14     }
15     public static void main(String[] args) throws InterruptedException {
16         ReenterLock tl=new ReenterLock();
17         Thread t1=new Thread(tl);
18         Thread t2=new Thread(tl);
19         t1.start();t2.start();
20         t1.join();t2.join();
21         System.out.println(i);
22     }
23 }
```

与synchronized相比，重入锁有着显示的操作过程。开发人员必须手动指定何时加锁，何时释放锁。也正因为这样，重入锁对逻辑控制的灵活性要远远好于synchronized。

就重入锁的实现来看，它主要集中在Java层面。在重入锁的实现中，主要包含三个要素：

第一，是原子状态。原子状态使用CAS操作（在第4章进行详细讨论）来存储当前锁的状态，判断锁是否已经被别的线程持有。

第二，是等待队列。所有没有请求到锁的线程，会进入等待队列进行等待。待有线程释放锁后，系统就能从等待队列中唤醒一个线程，继续工作。

第三，是阻塞原语park()和unpark()，用来挂起和恢复线程。没有得到锁的线程将会被挂起。

### 中断响应

对于synchronized来说，如果一个线程在等待锁，那么结果只有两种情况，要么它获得这把锁继续执行，要么它就保持等待。而使用重入锁，则提供另外一种可能，那就是线程可以被中断。也就是在等待锁的过程中，程序可以根据需要取消对锁的请求。中断会释放当前线程的所有锁，在适当情况下可以避免死锁

```java
01 public class IntLock implements Runnable {
02     public static ReentrantLock lock1 = new ReentrantLock();
03     public static ReentrantLock lock2 = new ReentrantLock();
04     int lock;
05     /**
06      * 控制加锁顺序，方便构造死锁
07      * @param lock
08      */
09     public IntLock(int lock) {
10         this.lock = lock;
11     }
12
13     @Override
14     public void run() {
15         try {
16             if (lock == 1) {
17                 lock1.lockInterruptibly();
18                 try{
19                     Thread.sleep(500);
20                 }catch(InterruptedException e){}
21                 lock2.lockInterruptibly();
22             } else {
23                 lock2.lockInterruptibly();
24                 try{
25                     Thread.sleep(500);
26                 }catch(InterruptedException e){}
27                 lock1.lockInterruptibly();
28             }
29
30         } catch (InterruptedException e) {
31             e.printStackTrace();
32         } finally {
33             if (lock1.isHeldByCurrentThread())
34                 lock1.unlock();
35             if (lock2.isHeldByCurrentThread())
36                 lock2.unlock();
37             System.out.println(Thread.currentThread().getId()+":线程退出");
38         }
39     }
40
41     public static void main(String[] args) throws InterruptedException {
42         IntLock r1 = new IntLock(1);
43         IntLock r2 = new IntLock(2);
44         Thread t1 = new Thread(r1);
45         Thread t2 = new Thread(r2);
46         t1.start();t2.start();
47         Thread.sleep(1000);
48         //中断其中一个线程
49         t2.interrupt();
50     }
51 }
```

线程t1和t2启动后，t1先占用lock1，再占用lock2；t2先占用lock2，再请求lock1。因此，很容易形成t1和t2之间的相互等待。在这里，对锁的请求，统一使用lockInterruptibly()方法。这是一个可以对中断进行响应的锁申请动作，即在等待锁的过程中，可以响应中断。

在代码第47行，主线程main处于休眠，此时，这两个线程处于死锁的状态，在代码第49行，由于t2线程被中断，故t2会放弃对lock1的申请，同时释放已获得lock2。这个操作导致t1线程可以顺利得到lock2而继续执行下去。

### 锁申请等待限时

除了等待外部通知之外，要避免死锁还有另外一种方法，那就是限时等待。依然以约朋友打球为例，如果朋友迟迟不来，又无法联系到他。那么，在等待1～2个小时后，我想大部分人都会扫兴离去。对线程来说也是这样。通常，我们无法判断为什么一个线程迟迟拿不到锁。也许是因为死锁了，也许是因为产生了饥饿。但如果给定一个等待时间，让线程自动放弃，那么对系统来说是有意义的。我们可以使用tryLock()方法进行一次限时的等待。

下面这段代码展示了限时等待锁的使用。

```java
01 public class TimeLock implements Runnable{
02     public static ReentrantLock lock=new ReentrantLock();
03     @Override
04     public void run() {
05         try {
06             if(lock.tryLock(5, TimeUnit.SECONDS)){
07                 Thread.sleep(6000);
08             }else{
09                 System.out.println("get lock failed");
10             }
11         } catch (InterruptedException e) {
12             e.printStackTrace();
13         }finally{if(lock.isHeldByCurrentThread()) lock.unlock();}
14     }
15     public static void main(String[] args) {
16         TimeLock tl=new TimeLock();
17         Thread t1=new Thread(tl);
18         Thread t2=new Thread(tl);
19         t1.start();
20         t2.start();
21     }
22 }
```

在这里，tryLock()方法接收两个参数，一个表示等待时长，另外一个表示计时单位。这里的单位设置为秒，时长为5，表示线程在这个锁请求中，最多等待5秒。如果超过5秒还没有得到锁，就会返回false。如果成功获得锁，则返回true。

在本例中，由于占用锁的线程会持有锁长达6秒，故另一个线程无法在5秒的等待时间内获得锁，因此，请求锁会失败。

ReentrantLock.tryLock()方法也可以不带参数直接运行。在这种情况下，当前线程会尝试获得锁，如果锁并未被其他线程占用，则申请锁会成功，并立即返回true。如果锁被其他线程占用，则当前线程不会进行等待，而是立即返回false。这种模式不会引起线程等待，因此也不会产生死锁。下面演示了这种使用方式：

```java
01 public class TryLock implements Runnable {
02     public static ReentrantLock lock1 = new ReentrantLock();
03     public static ReentrantLock lock2 = new ReentrantLock();
04     int lock;
05
06     public TryLock(int lock) {
07         this.lock = lock;
08     }
09
10     @Override
11     public void run() {
12         if (lock == 1) {
13             while (true) {
14                 if (lock1.tryLock()) {
15                     try {
16                         try {
17                             Thread.sleep(500);
18                         } catch (InterruptedException e) {
19                         }
20                         if (lock2.tryLock()) {
21                             try {
22                                 System.out.println(Thread.currentThread()
23                                         .getId() + ":My Job done");
24                                 return;
25                             } finally {
26                                 lock2.unlock();
27                             }
28                         }
29                     } finally {
30                         lock1.unlock();
31                     }
32                 }
33             }
34         } else {
35             while (true) {
36                 if (lock2.tryLock()) {
37                     try {
38                         try {
39                             Thread.sleep(500);
40                         } catch (InterruptedException e) {
41                         }
42                         if (lock1.tryLock()) {
43                             try {
44                                 System.out.println(Thread.currentThread()
45                                         .getId() + ":My Job done");
46                                 return;
47                             } finally {
48                                 lock1.unlock();
49                             }
50                         }
51                     } finally {
52                         lock2.unlock();
53                     }
54                 }
55             }
56         }
57     }
58
59     public static void main(String[] args) throws InterruptedException {
60         TryLock r1 = new TryLock(1);
61         TryLock r2 = new TryLock(2);
62         Thread t1 = new Thread(r1);
63         Thread t2 = new Thread(r2);
64         t1.start();
65         t2.start();
66     }
67 }
```

上述代码中，采用了非常容易死锁的加锁顺序。也就是先让t1获得lock1，再让t2获得lock2，接着做反向请求，让t1申请lock2，t2申请lock1。在一般情况下，这会导致t1和t2相互等待，从而引起死锁。

但是使用tryLock()后，这种情况就大大改善了。由于线程不会傻傻地等待，而是不停地尝试，因此，只要执行足够长的时间，线程总是会得到所有需要的资源，从而正常执行（这里以线程同时获得lock1和lock2两把锁，作为其可以正常执行的条件）。在同时获得lock1和lock2后，线程就打印出标志着任务完成的信息“My Job done”。

### 公平锁

在大多数情况下，锁的申请都是非公平的。也就是说，线程1首先请求了锁A，接着线程2也请求了锁A。那么当锁A可用时，是线程1可以获得锁还是线程2可以获得锁呢？这是不一定的。系统只是会从这个锁的等待队列中随机挑选一个。因此不能保证其公平性。这就好比买票不排队，大家都乱哄哄得围在售票窗口前，售票员忙得焦头烂额，也顾不及谁先谁后，随便找个人出票就完事了。而公平的锁，则不是这样，它会按照时间的先后顺序，保证先到者先得，后到者后得。**公平锁的一大特点是：它不会产生饥饿现象。**只要你排队，最终还是可以等到资源的。如果我们使用synchronized关键字进行锁控制，那么产生的锁就是非公平的。而重入锁允许我们对其公平性进行设置。它有一个如下的构造函数：

```java
public ReentrantLock(boolean fair)
```

当参数fair为true时，表示锁是公平的。公平锁看起来很优美，但是要实现公平锁必然要求系统维护一个有序队列，因此公平锁的实现成本比较高，性能相对也非常低下，因此，默认情况下，锁是非公平的。如果没有特别的需求，也不需要使用公平锁。公平锁和非公平锁在线程调度表现上也是非常不一样的。下面的代码可以很好地突出公平锁的特点：

```java
01 public class FairLock implements Runnable {
02     public static ReentrantLock fairLock = new ReentrantLock(true);
03
04     @Override
05     public void run() {
06         while(true){
07         try{
08             fairLock.lock();
09             System.out.println(Thread.currentThread().getName()+" 获得锁");
10         }finally{
11             fairLock.unlock();
12         }
13         }
14     }
15
16     public static void main(String[] args) throws InterruptedException {
17         FairLock r1 = new FairLock();
18         Thread t1=new Thread(r1,"Thread_t1");
19         Thread t2=new Thread(r1,"Thread_t2");
20         t1.start();t2.start();
21     }
22 }
```

### 常用方法

对上面ReentrantLock的几个重要方法整理如下。

- lock()：获得锁，如果锁已经被占用，则等待。
- lockInterruptibly()：获得锁，但优先响应中断。
- tryLock()：尝试获得锁，如果成功，返回true，失败返回false。该方法不等待，立即返回。
- tryLock(long time, TimeUnit unit)：在给定时间内尝试获得锁。
- unlock()：释放锁。

## Condition条件

如果大家理解了Object.wait()和Object.notify()方法的话，那么就能很容易地理解Condition对象了。它和wait()和notify()方法的作用是大致相同的。但是wait()和notify()方法是和synchronized关键字合作使用的，而Condtion是与重入锁相关联的。通过Lock接口（重入锁就实现了这一接口）的Condition newCondition()方法可以生成一个与当前重入锁绑定的Condition实例。利用Condition对象，我们就可以让线程在合适的时间等待，或者在某一个特定的时刻得到通知，继续执行。

Condition接口提供的基本方法如下：

```java
void await() throws InterruptedException;
void awaitUninterruptibly();
long awaitNanos(long nanosTimeout) throws InterruptedException;
boolean await(long time, TimeUnit unit) throws InterruptedException;
boolean awaitUntil(Date deadline) throws InterruptedException;
void signal();
void signalAll();
```

以上方法的含义如下：

- await()方法会使当前线程等待，同时释放当前锁，当其他线程中使用signal()或者signalAll()方法时，线程会重新获得锁并继续执行。或者当线程被中断时，也能跳出等待。这和Object.wait()方法很相似。
- awaitUninterruptibly()方法与await()方法基本相同，但是它并不会在等待过程中响应中断。
- singal()方法用于唤醒一个在等待中的线程。相对的singalAll()方法会唤醒所有在等待中的线程。这和Obejct.notify()方法很类似。

```java
01 public class ReenterLockCondition implements Runnable{
02     public static ReentrantLock lock=new ReentrantLock();
03     public static Condition condition = lock.newCondition();
04     @Override
05     public void run() {
06         try {
07             lock.lock();
08             condition.await();
09             System.out.println("Thread is going on");
10         } catch (InterruptedException e) {
11             e.printStackTrace();
12         }finally{
13             lock.unlock();
14         }
15     }
16     public static void main(String[] args) throws InterruptedException {
17         ReenterLockCondition tl=new ReenterLockCondition();
18         Thread t1=new Thread(tl);
19         t1.start();
20         Thread.sleep(2000);
21         //通知线程t1继续执行
22         lock.lock();
23         condition.signal();
24         lock.unlock();
25     }
26 }
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