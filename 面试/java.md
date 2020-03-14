**接口与抽象类的区别**

接口只定义接口方法，抽象类除了定义抽象方法，也可以有属性与其它实现方法

接口比抽象类更加灵活，可以实现多个接口，但只可以继承一个类

**如果hashMap的key是一个自定义的类，怎么办**

重写equals与hashcode

**为什么重写equals还要重写hashcode**

equals用于比较两个对象是否相同，hashCode是一个对象的散列码，用于进行hash查找key，在hashmap的put中，hashcode用于定位对象在哪条链表中，equals用于判断对象是否相同，如果不重写hashcode那么有可能逻辑上相同的两个对象定位到了不同的链表中导致了重复插入

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

