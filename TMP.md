## JVM

class文件，字节码，数据按照虚拟机规范照指定格式排列在class文件中的二进制流，包括版本，访问标志常量池，当前类，超级类，接口，字段，方法，属性等。查看class文件可使用专用的工具。java有自己专用的指令码（二进制），对应助记操作符（类似于汇编），jvm内存结构为

<img src="https://s1.ax1x.com/2020/04/17/JERXQA.png" alt="JERXQA.png" style="zoom:50%;" />

**javap**

查看class文件的工具，下载jdk时附带，使用方式为javap [参数] xxx.class (> [可指定的重定向文件])

**jitwatch**

java运行时编译的可视化工具，github开源项目

### JVM运行时数据区

源代码编译成字节码，虚拟机加载字节码转换成本地指令并执行。

**JVM运行时数据区**

线程共享：方法区（存储类信息，静态变量等），堆内存（存放实例对象，包括老年代与新生代）

线程独占：虚拟机栈（存储栈帧，包括局部变量表等），本地方法栈（同虚拟机栈，为执行本地方法而准备），程序计数器（记录当前线程执行字节码的位置，存储字节码指令地址）

**JVM运行**

简单的说，先将类加载进入方法区，之后JVM创建线程执行代码，在虚拟机栈、程序计数器内存区域中创建线程独占的空间（也就是说线程空间包含自己的栈，程序计数器），之后执行字节码指令（也就是执行方法的过程，此时栈帧不断变化，栈帧包含本地变量表，操作数栈等多个运行方法所需要的数据信息），每当调用其它方法时就根据方法描述创建新栈帧，方法参数从操作数栈中弹出来压入虚拟机栈，虚拟机执行虚拟机最上层的栈帧，通过栈完成方法调用与指令执行的过程

### 缓存一致性与指令重排

**缓存一致性**

现代CPU使用多级缓存结构（cache，L1-L3），对于缓存来说，在多CPU读取同样的数据进行缓存，并进行数据运算（更新）后仍需写入主存，写入主存的以哪个CPU为准成了一个问题。

为解决这个问题要实现缓存同步协议，在单个CPU对缓存中的数据进行改动时要通知给其它CPU，也就是说CPU要控制自己的读写的同时还要监听其它CPU发出的通知

**指令重排**

CPU性能优化时的一种手段，指令重排会破坏指令执行的顺序，在多线程执行时会破坏代码逻辑（指令重排会遵守as-if-serial语义，即单线程程序执行结果不会被改变）

cpu缓存与指令重排在多线程执行时会导致问题，如缓存数据与主内存数据并不同步，CPU间的缓存数据也不同步，指令重排会破坏多线程编程时的代码逻辑。处理器提供了两个内存屏障指令用于解决上述问题

写内存屏障（Store Memory Barrier）：让写入缓存中的最新数据更新写入主内存，让其他线程可见（强制写入主内存，此时不会指令重排），避免指令重排问题

读内存屏障（Load Memory Barrier）：强制缓存数据失效，从主内存中加载数据，必变缓存一致性问题。

## JMM

为屏蔽各个硬件平台的差异性，jvm有自己的内存模型，内存模型描述程序的可能行为，程序的运行结果可由内存模型预测，虚拟机规范中的内存模型规定了线程操作的定义，同步的规则等信息，内存模型决定了在程序的每个点上可以读取到什么值

<img src="https://s1.ax1x.com/2020/04/17/JEWb7V.png" alt="JEWb7V.png" style="zoom:50%;" />

<img src="https://s1.ax1x.com/2020/04/17/JEfAhD.png" alt="JEfAhD.png" style="zoom:50%;" />

**final在JMM中的处理**

<img src="https://s1.ax1x.com/2020/04/17/JEoiWT.png" alt="JEoiWT.png" style="zoom:50%;" />

**double与long的特殊处理**

虚拟机规范中，写64位别的double与long操作分为两次32位写操作，因此对double于long的操作必须是原子操作（大部分商业虚拟机已经实现），否则可能出现赋值错误的诡异情况出现

## 线程

### 线程状态

5态模型

* New：尚未启动线程（未代用start()之前）
* Runnable：可运行线程，等待CPU调度（调用start()之后）
* Blocked：线程阻塞，处于同步代码块中或方法中被阻塞（没有拿到锁）
* Waiting（Timed Waiting）：线程等待，需要被唤醒或者等待指定时间被唤醒（睡眠或者wait()）
* Terminated：线程终止，执行完毕或抛出异常

<img src="https://i.loli.net/2020/03/31/YmdcK7tzvSr51i4.png" alt="image.png" style="zoom: 50%;" />

### 线程终止

**stop方法**

不建议用，使用stop()方法会直接终止线程执行，同时释放所有持有的锁，因此会破坏同步（如只执行到一半代码时被强制stop，锁被释放了，同步会被破坏），stop已被标记为过时

**interrupt**

使用中断会终止线程执行并重置线程的中断状态位，出错会抛出InterruptedException异常，便于逻辑控制。

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

**使用自定义标志位**

在代码逻辑中增加一个控制标志位来控制线程的终止，初始时置为true，想要终止时置为false，常用于循环控制

### 线程通信

线程通信可实现线程之间的协同，主要分为文件共享、网络共享、共享变量与jdk提供的api

**文件共享/网络共享**

以文件为数据存储媒介通过读写文件实现数据交互

**变量共享**

以主存为数据存储媒介通过读写内存中的变量完成数据交互

<img src="https://i.loli.net/2020/03/31/aesqd5hKElwJUQH.png" alt="image.png" style="zoom:50%;" />

**线程协作api**

**生产者消费者模型**

<img src="https://i.loli.net/2020/03/31/bjA8T6xvFMKWgG1.png" alt="image.png" style="zoom:50%;" />

该模型通过调用wait()/notify()/notifyAll()方法实现

wait()使当前线程等待（加入等待集合中）并释放当前持有的对象锁，这两个方法只能由同一对象锁的持有者线程调用，实际上就是对某一对象的线程同步操作，由该对象调用wait/notify()，同时在调用notify()之前要先使用wait()，顺序不对会导致死锁

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

**park/unpark机制**

一种令牌许可机制，线程调用park则等待“许可”，unpark方法为指定线程提供“许可”，不要求先后顺序，只要有许可就可以直接执行。这两个方法位于concurrent包中的LockSupport类中。

```java
    public static void parkUnparkTest() throws Exception{
        Thread consumerThread=new Thread(()->{
            System.out.println("1.进入等待");
            LockSupport.park();
            System.out.println("3.等到许可");
        });
        consumerThread.start();
        Thread.sleep(1000L);
        LockSupport.unpark(consumerThread);//通知许可
        System.out.println("2.通知完毕");
    }
```

park/unpark不会释放拿到的锁，在同步方法中拿到同一个锁时会导致死锁

==在线程通信中值得注意的地方在于对检查条件不应该使用if条件判断，而应该使用while条件，因为处于等待状态的线程可能会受到伪唤醒（底层实现代码所致），如果不在循环中检查等待条件，程序可能会在没有满足结束条件的情况下退出（while会一直循环直到条件判断成立）==

```java
        //wait
        synchronized (obj){
            while(<条件判断>){
                obj.wait();
            }
        }
        //park
        while(<条件判断>){
            LockSupport.park();
        }
```

### 线程封闭

数据被封闭在线程中，不需要进行同步

**ThreadLocal**

位于concurrent包中，泛型类，是一个线程级别的变量，每个线程都有一个ThreadLocal，相当于每个线程都拥有一个自己独立的变量，保证并发下该变量的绝对安全。使用static修饰的静态ThreadLocal变量会为每个线程创建一个副本，该副本只对创建他的线程可见，实现数据对线程的封闭

```java
   public static ThreadLocal<String> threadLocal=new ThreadLocal<>();
   public static void threadLocalTest() throws Exception{
        threadLocal.set("主线程设置123");
        System.out.println(threadLocal.get());
        new Thread(()->{
            threadLocal.set("线程设置456");
            System.out.println(threadLocal.get());
        }).start();
        Thread.sleep(1000L);
        System.out.println(threadLocal.get());
    }
```

### 线程池

**线程池管理器创建**

创建并管理线程池

**工作线程**

线程池中的线程，没有任务时处于等待状态，可循环执行任务

**任务接口**

每个任务必须实现的接口以供工作线程调度任务的执行，规定了任务的入口，任务执行完的收尾工作等

**任务队列**

存放没有处理的任务，提供一种缓冲机制

<img src="https://i.loli.net/2020/03/31/urPKn6XpBG45OLm.png" alt="image.png" style="zoom:50%;" />

**线程池API**

**Executor**

最上层接口，定义任务执行的方法execute

**ExecutorService**

继承自Executor接口，拓展Callable、Future、关闭方法等

**ScheduledExecutorService**

继承自ExecutorService，增加定时任务相关的方法

**ThreadPoolExecutor**

基础的线程池实现，基本的三个参数为，核心线程数量，最大线程数量，超出核心线程数的线程存活时间，第四个参数为单位，一般为TimeUnit.SECONDS

**ScheduledThreadPoolExecutor**

继承自ThreadPoolExecutor，实现ScheduledExecutorService接口中的定时任务方法

**Executors工具类**

帮助创建线程池的类

* newFixedThreadPool(int nThreads)：创建一个固定大小，无界队列的线程池，等价于`new ThreadPoolExecuotr(5,5,0L,TimeUnit.MILLISECONDS)`
* newCachedThreadPool()：创建一个大小无界的缓冲线程池，任务队列是一个同步队列，有空闲线程就执行，没有就创建一个线程执行，线程数随任务的多少变化，并不会控制线程数量大小，内部实现为`new ThreadPoolExecutor(0,Integer.MAX_VALUE,60L,TimeUnit.SECONDS)`，核心线程数为0，最大线程很大，最大存活时间60s，有空闲线程就执行，没有就创建执行，空闲60s自动销毁。

* newSingleThreadExecutor()：只有一个线程俩执行无界任务队列，确保任务按顺序逐个执行
* newScheduledThreadPool(int corePoolSize)：定时执行任务的线程池，等价于`new ScheduledThreadPoolExecutor(){...},3000,TimeUnit.MILLISECONDS，`，定时执行任务，本质为延时队列，有两种执行方式

**线程池终止**

* shutdown()：等待执行结束后全部关闭
* shutdownNow()：不管有没有任务都立刻终止，返回一个List<Runnable\>，即没有执行完的任务

### 线程池原理

![JE6VpR.png](https://s1.ax1x.com/2020/04/17/JE6VpR.png)

超出核心线程数量的线程数会在最大存活时间后自动销毁

### 线程安全

**竞态条件与临界区**

临界区：并发线程中与共享变量有关的程序段叫做临界区，即一次只能有一个线程执行的程序段

竞态条件：多个线程访问的相同资源，一次最多只能有一个线程对其进行操作

一段代码是线程安全的则它不包含竞态条件，只有多个线程更新共享资源时才会发生竞态条件

栈封闭（如ThreadLocal）不会在线程之间共享变量因此是线程安全的

原子操作：要么全做，要么全不做的一系列不可分割的过程，不可中断

**CAS操作**

比较并替换，维护两个值，旧值与新值，每次将旧值与实际值比较，如果符合则实际替换成新值。一般提供的方法为cas(旧值，当前值，新值)

问题：循环+CAS，消耗资源；只能针对单个变量；ABA问题

ABA问题：看似变量没变化，实际上变化了，解决方案是每次修改时附加版本号 

## 锁

**自旋锁**：不放弃cpu，不断循环cas对数据尝试进行更新

**悲观锁**：假定会发生并发冲突，同步所有对数据的相关操作，从读取数据就开始上锁

**乐观锁**：假定没有冲突，修改数据时如果发现和之前获取的数据不一致则读取最新数据，修改后重新尝试修改

**排它锁**：写锁，一次最多有一个线程获取一个写锁，其它线程不能再加锁

**共享锁**：读锁，可以多个线程获取，但不能再加写锁

**可重入锁**：同一个线程可以多次获取同一把锁

### synchronized

基于对象监视器实现，每个对象都与一个监视器相关联，一次只有一个线程可以锁定监视器，其它线程则阻塞。同步代码快，能保证内存可见性，禁止指令重排序。

* 普通方法锁住当前实例对象
* 静态同步方法锁住当前类的Class对象
* 同步方法块锁住括号内的对象

**锁粗化**：jit编译时的一种优化，对于重复获取锁的代码块进行优化，将同步代码块放入一把锁中，即将锁的范围扩大

**锁消除**：jti编译时的一种优化，对于没有并发冲突的同步代码进行锁消除

synchronized锁存储在java对象头中，对象头包含mark word，存储对象的hashCode和锁信息等，其信息会随着锁标志位的变化而变化

**锁升级**

synchronized锁会从无锁状态->偏向锁->轻量级锁->重量级锁这四种状态变化

**偏向锁**：当第一个线程获取锁时，对象头记住该线程id，以后该线程进出同步代码块时都不用再cas获取锁，其它线程获取锁时会检查对象头中是否有偏向锁，有则尝试cas获取锁，而原先拿到锁的线程在合适的时候释放锁，之后锁就会升级。即使用一种只有等到竞争出现时才释放锁的机制，只有当其它线程尝试获取锁时持有偏向锁的线程才会释放锁。偏向标记只有第一次有用，出现过争用之后就没用了。

**轻量级锁**：出现多线程争用情况，线程1cas获取锁成功而线程2失败，此时线程2并不阻塞而是不断自旋尝试获取锁，自旋次数太多则锁升级为重量级锁

**重量级锁**：多线程争用时失败线程不会再自旋而是直接阻塞

![image.png](https://i.loli.net/2020/04/20/Mtn1WLXG3uZOo5B.png)

## concurrent包

### 基础接口

**Lock**

基本锁接口

**ReadWriteLock**

基本读写锁接口

### 基础实现

**ReentrantLock**

可重入锁，有公平/非公平两种模式，但重入的时候加锁(lock)与解锁(unlock)的次数应该一致，否则不会释放锁

**ReentReadWriteLock**

ReadWriteLock接口实现，维护一对关联锁，一个只用于读，一个只用于写，写锁排他，读锁共享，构造该对象之后可以分别获取读写锁

```java
ReadWriteLock readWriteLock=new ReentrantReadWriteLock();
Lock r=readWriteLock.readLock();//读锁
Lock w=readWriteLock.writeLock();//写锁
```

锁降级：将写锁降级为读锁

### Condition

配合Lock使用，实现wait()/notify()的功能（底层使用park/unpark机制），提供更加精细的同步控制，注释提供的实例为

```java

   class BoundedBuffer {
     final Lock lock = new ReentrantLock();
     final Condition notFull  = lock.newCondition(); //条件，逻辑上代表不满
     final Condition notEmpty = lock.newCondition(); //条件，逻辑上代表不空
  
     final Object[] items = new Object[100];
     int putptr, takeptr, count;
     //写入数据
     public void put(Object x) throws InterruptedException {
       lock.lock();
       try {
         while (count == items.length)
           notFull.await();//已经满了，不空等待
          //否则写入数据
         items[putptr] = x;
         if (++putptr == items.length) putptr = 0;//指针循环
         ++count;
         notEmpty.signal();//释放不空信号
       } finally {
         lock.unlock();
       }
     }
     //读取数据
     public Object take() throws InterruptedException {
       lock.lock();
       try {
         while (count == 0)
           notEmpty.await();//没有数据，不空等待
         Object x = items[takeptr];
         if (++takeptr == items.length) takeptr = 0;
         --count;
         notFull.signal();//释放不满信号
         return x;
       } finally {
         lock.unlock();
       }
     }
   }
```

### 数据类型原子操作

在包atomic下提供一系列对基本数据类型的包装类实现对基本数据类型的原子操作，基于Lock下的Unsafe类使用CAS完成原子操作，Unsafe的不安全性在于其功能庞大，难以使用，另一方面在于java的跨平台性，有些平台并不支持某些操作

<img src="https://s1.ax1x.com/2020/04/17/JZAMiq.png" alt="JZAMiq.png" style="zoom:50%;" />

上述API底层使用Unsafe的CAS操作，jdk8提供了专门的计数器类提高效率，基本思路为多个线程分配多个值操作，对上提供虚拟的值来进行访问，减少并发操作。基本的维护一个数组cell[]，将一个value分为多个让多个线程操作，最终值使用sum返回所有值的和，适用于写多读少的场景。

### 基于并发包实现Lock

```java
public class LockDemo implements Lock {
    //判断锁的状态或者说拥有者
    volatile AtomicReference<Thread> owner=new AtomicReference<>();
    //保存正在等待的线程
    volatile LinkedBlockingQueue<Thread> waiters=new LinkedBlockingQueue<>();

    @Override
    public boolean tryLock(){
        return owner.compareAndSet(null,Thread.currentThread());
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return false;
    }

    @Override
    public void unlock() {
        //释放拥有者
        if(owner.compareAndSet(Thread.currentThread(),null)){
            //通知等待者 释放线程
            for (Thread next : waiters) {
                LockSupport.unpark(next);
            }
        }
    }

    @Override
    public Condition newCondition() {
        return null;
    }

    @Override
    public void lock(){
        boolean addQ=true;
        while (!tryLock()){
            if(addQ){
                //如果没拿到锁，加入到等待集合
                waiters.offer(Thread.currentThread());
                addQ=false;
            }else {
                //阻塞当前线程
                LockSupport.park();//伪唤醒，非unpark唤醒，因此用while
            }
        }
        waiters.remove(Thread.currentThread());
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {

    }
}
```

### AQS

Abstract Queue Synchronizer 抽象队列同步器，提供了对资源占用，释放，线程等待与唤醒等接口和具体实现，可用在各种需要控制资源争用的场景中

![](https://i.loli.net/2020/04/20/4JMS2InYu5Ad8KB.png)

内部维护一个状态位state，首个线程进入时CAS置状态位为1，失败代表已被使用，CAS加入等待队列，不断CAS判断自己是否在队头，在队头则CAS尝试获取状态位。

AQS资源占用的过程为：

![JtkU8P.png](https://s1.ax1x.com/2020/04/22/JtkU8P.png)

### Semaphore

信号量，控制多个线程争取许可，信号量可以使多个线程同时使用一个资源，即只要有许可即可访问

* acquire：获取一个许可，没有则等待
* release：释放一个㐓
* availablePermits：获取可用的许可数目

信号量内部也使用AQS实现，内部维护许可数量，保证同一时间最多有有限个线程访问同步代码块

典型场景：

**代码并发处理限流**

即控制访问资源的线程数量

```java
public class SemaphoreDemo {
    public static void main(String[] args){
        SemaphoreDemo semaphoreDemo=new SemaphoreDemo();
        int N=8;//有8个人访问
        Semaphore semaphore=new Semaphore(5);//许可数量
        for(int i=0;i<N;i++){
            String vipNo="vip-00"+i;
            new Thread(()->{
                try {
                    semaphore.acquire();//获取令牌
                    semaphoreDemo.service(vipNo);
                    semaphore.release();//释放许可
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }).start();
        }
    }
    public void service(String vipNo) throws InterruptedException{
        System.out.println("访问"+vipNo);
        Thread.sleep(new Random().nextInt(3000));
        System.out.println("结束访问"+vipNo);

    }
}
```

### CountDownLatch

工具类，也称为倒计数器，创建对象时，传入指定数值作为线程参与的数量

* await：方法等待计数器值变为0，在这之前线程进入等待状态
* coutdown：计数器数值减一直到为0

经常用于等待其他线程执行到某一节点再继续执行当前线程代码，内部仍使用AQS的state作为标识，只要state不为0就仍挂起，直到变为0才唤醒在await上挂起的线程

```java
public class CountDownDemo {
    public static void main(String[] args)throws InterruptedException{
        CountDownLatch countDownLatch=new CountDownLatch(10);//创建，计数
        for(int i=0;i<9;i++){
            new Thread(()->{
                try{
                    Thread.sleep(2000L);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
                countDownLatch.countDown();
                System.out.println("执行完毕，计数器减1,还需等待"+countDownLatch.getCount());
            }).start();
        }
        countDownLatch.countDown();
        countDownLatch.await();//等待计数器为0
        System.out.println("结束");
    }
}
```

### CyclicBarrier

又称线程栅栏，创建对象时，指定栅栏线程数量

* await：等指定数量的线程都处于等待状态时，继续执行后续代码
* barrierAction：线程数量到了指定量之后，自动触发执行指定任务

CyclicBarrier对象可多次触发执行，只有当到达指定线程数量时才会唤醒线程，否则就将在await上的线程挂起

典型场景：

* 等多个线程数据统计完之后进行数据汇总

```java
public class CyclicBarrierDemo {
    public static void main(String[] args)throws InterruptedException{
        LinkedBlockingQueue<String> sqls=new LinkedBlockingQueue<>();
        CyclicBarrier barrier=new CyclicBarrier(10,()->{
            //每满足10次数据库操作，触发一次批量执行
            System.out.println("触发");
        });
        for(int i=0;i<10;i++){
            new Thread(()->{
                try {
                    sqls.add("data-"+Thread.currentThread());
                    Thread.sleep(1000L);//模拟数据库操作
                    barrier.await();//等待屏障打开，4个线程都完毕后才打开，如同栅栏一样
                    System.out.println("插入完毕");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

## concurrent包中的并发容器类

### ConcurrentHashMap

**jdk1.7中的实现**

内部不再直接使用Entry而是使用Segment，即分段锁机制，Segment的初始数量固定，Segment内部维护一个数组table，table类型为HashEntry

* put：根据key找到一个Segment，然后锁住Segment，之后在Segment中根据key向table中插入数据（相当于Segment中又维护了一个HashMap）
* get：根据key拿到Segment，不用上锁直接拿即可

**jdk1.8中的实现**

取消Segment，使用Node作为元素（保存K,V与附属信息）存储，内部维护Node链表的数组table

* put：根据key找到table中对应的位置，如果该位置为null则通过CAS尝试初始化Node链表，之后锁住链表的头部，然后直接在该链表中put数据
* get：根据key拿到table中的链表，然后从链表中拿数据即可

### ConcurrentSkipListMap

跳表，最底层是一个链表，链表插入有序，但该链表有索引，该索引index包含链表节点，插入Node时随机创建一个索引（如插入数据为偶数则创建索引）。索引Index也是一个链表，入口为一个HeadIndex（顶级索引），插入数据时先查找索引，之后根据索引确定范围再在底部链表寻找。

对于大数据量，跳表也可以创建多级索引，在二级索引基础上，加入插入底层链表的节点需要生成该节点的索引，该索引需要先判断该索引的级别（如以该节点数据二进制连续为1的数量生成级别），即生成多级别索引，顶级索引指向最高级索引链表的头部

```java
class Node{
    String key;
    String value;
    Node next;
}
class Index{
    Node node;
    Index right;
    Index down;
}
class SkipList{
	Node head;
}
```

![JUm3VK.png](https://s1.ax1x.com/2020/04/22/JUm3VK.png)

### CopyOnWriteArrayList

内部仍是Object数组，可以保证添加/删除数据的时候保证其它正在读数据的线程不受干扰（可以一边读数据一边增删数据）

* add：操作前先直接锁住，然后复制数组数据，将数据添加至复制数组，之后维护的数组变为复制数组

### CopyOnWriteArraySet

线程安全的Set，基于CopyOnWriteArrayList实现

### ArrayBlockingQueue

队列阻塞



​	