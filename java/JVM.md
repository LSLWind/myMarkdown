## JAVA内存管理

### JAVA运行时内存区域

* 程序计数器：线程私有，记录字节码指令地址
* 栈：线程私有，描述方法执行的内存模型，存放栈帧，包括局部变量表，方法出口等信息
* 堆：线程共享，存放对象实例与数组
* 方法区：线程共享，存放类信息，静态变量，常量池等
* 运行时常量池：方法区的一部分，存储编译器生成的class符号引用等

### 对象创建过程

遇到new 时先去常量池查找类符号引用，判断类是否被加载，未被加载则先执行类加载过程，之后为对象分配空间（内存大小可在类加载完成后完全确定），将内存空间初始化为0，对对象头设置必要的信息数据。

### 对象内存布局

**对象头**：包括运行时数据(Mark World)，如哈希码，偏向线程id，锁状态标志等；类型指针（指向.class的指针），虚拟机通过该指针确定对象是哪个类的实例。另外，如果对象是java数组，则对象头中还包括一块用于记录数组长度的数据（用于确定对象内存大小，数组无法通过元数据获取自身长度）

**实例数据**：代码中定义的字段内容，包括从父类中继承的与在子类中定义的

**对齐填充**：占位符，保证对象内存大小为8的整数倍

### 对象访问定义

对象是引用refernce类型，java程序需要通过栈上的refrence数据来操作堆上的具体对象，如何通过引用去访问对象由具体的虚拟机去实现，主流虚拟机有两种实现方式

**使用句柄访问**：在java堆中划分出一块内存作为句柄池，refrence中存储对象的句柄地址，句柄地址则包含指向对象实例数据（实际对象，堆中）与类型数据的地址（类，方法区中）这两部分信息

**直接指针访问**：refrence直接存储对象地址，对象中则必须包含能够访问类型数据的地址

## GC与内存分配

PC、栈等都是线程私有，栈帧中的内存在类结构确定下来时就是已知的，而堆与方法区则不同，程序只有在运行时才知道会创建那些对象，内存分配与回收都是动态的

### 确定对象是否存活

一般使用**可达性分析算法**来确定对象是否存活（也就是对象是否需要被回收），基本思想是：使用一系列的称为“GC Roots"的对象作为起始点，从该起始点向下搜索，走过的路径称为引用链，当从GC Root到某个对象不可达时（没有任何引用链）则判定该对象已死

[<img src="https://s2.ax1x.com/2020/02/13/1L0Sqs.png" alt="1L0Sqs.png" style="zoom:50%;" />](https://imgchr.com/i/1L0Sqs)

一般的引用指的是该引用类型中存储的数据是另一块内存的起始地址，java对引用类型进行了扩充，有四种类型：

* 强引用：普遍存在的，如Object o=new Object()，只要强引用还在则永远不被GC
* 软引用：描述还有用但并非必须的对象，在内存将要溢出之前进行回收，使用类SortReference来实现
* 弱引用：同样描述非必须对象，GC时回收该部分，由类WeakReference来实现
* 虚饮用：最弱的引用关系，无法通过虚引用来取得对象实例，唯一的目的就是该对象被GC时收到一个系统通知

### 垃圾收集算法

**标记-清除算法**：首先标记出所有需要回收的对象，在标记完成后进行统一回收，其缺点主要有两个：效率过低，标记与清除的过程的效率都不高；空间问题，会导致出现过多的内部碎片

**复制算法**：将内存分为容量相等的两块，每次只使用其中的一块，当这块用完时就将这块还存活的对象复制到另一块上面去，然后将这半块进行统一回收，缺点就是将内存分为了原来的一半，代价过大

现代虚拟机则并不会将内存等分，而是（新生代）分为一块较大的Eden空间和两块较小的Survivor空间（比例为8:1），每次使用Eden与一块Survivor，当回收时，将Eden与Survivor中还存活的对象分配到另一块Survivor中，如果这块Survivor内存不够，则这些对象通过分配担保机制进入老年代

**标记-整理算法**：同标记-清除算法，但标记完不是直接清除，而是让存活对象向一侧移动，清除掉端边界以外的内存即可

**分代收集算法**：现代虚拟机常用，根据对象存活周期将内存划分为几块，一般分为新生代与老年代，新生代对象经常死去，采用复制算法；老年代采用标记-清除或标记-整理算法

GC时系统必须停顿，保证系统引用关系没有动态变化，当GC时系统能够准确知道引用位置，HotSpot中使用一组名为OopMap的数据结构保存类加载时对象的引用位置，这样GC时就可以直接获得引用位置，完成GC Roots的枚举。

每一条指令执行完毕后，系统的引用关系都有可能发生变化，因此理论上来讲每条指令执行完毕后都会产生不同的OopMap，HotSpot中定义了一系列**安全点（Safepoint）**，只在安全点记录信息，因此程序只能在到达安全点时才能开始进行GC，对于多线程来说，采用**主动式中断思想**：系统需要GC时设置一个标志，各个线程在执行时主动去轮询该标志，当发现中断标志位是该线程就自己中断。同时系统会规划一整段**安全区（Safe Region)**,只要线程要进入安全区时就标记自己在安全区，因此GC时系统就不用检查该线程，当该线程离开安全区时则要检查系统是否已经完成了根节点枚举，如果没有则必须等待。

### 垃圾收集器

垃圾收集器是垃圾回收算法理论的实现，主要几款收集器为

**Serial收集器**：新生代、单线程收集器，使用GC线程去GC，停止系统所有工作线程（Stop The World），因此每隔一段时间系统就会停止一段时间去GC，使用复制算法

**ParNew收集器**：Serial收集器的多线程版本、新生代收集器，使用复制算法

**Parallel Scavenge收集器**：新生代、多线程收集器，使用复制算法，与ParNew的区别在于其追求吞吐量（吞吐量=运行用户代码时间/（运行用户代码时间+GC时间）），停顿越短代表响应速度越快，吞吐量越大代表CPU利用率越高

**Serial Old 收集器**：Serial收集器的老年代版本、单线程，使用标记-整理算法

**Parallel Old 收集器**：Parallel Scavenge收集器的老年代版本、多线程，使用标记-整理算法

**CMS 收集器**：老年代、多线程，使用标记-清除算法，整个过程分为四步：初始标记（需停顿，仅对GC Roots能直接关联到的对象进行标记）→并发标记（执行可达性分析，找到GC Roots引用链）→重新标记（需停顿，修正因并发标记期间系统运作导致的对象变动）→并发清除

**G1 收集器**：较新的老年代收集器、可并发、多线程，用于取代CMS，将整块的新生代与老年代划分为不连续的多个Region的集合进行收集，回收过程与CMS差不多，步骤为：初始标记→并发标记→最终标记→筛选回收

### 对象内存分配

**对象优先分配在Eden区**，也就是新生对象优先分配在新生代，当Eden区没有足够的空间时将进行一次MinorGC（Minor GC指新生代GC，对应的，Full GC指老年代GC），**大对象直接进入老年代**（大对象指需要大量连续内存空间的java对象），目的是避免在Eden与Survivor之间的来回大对象复制（Eden与Survivor共同组成新生代内存，复制算法回收新生代内存），**长期存活对象进入老年代**，虚拟机对每个对象定义了一个对象Age计数器，如果对象在Eden出生并经过一次Minor GC则移入Survivor空间，将Age置为1，每经过一次Minor GC则Age加一，当Age达到阈值（默认15）则进入老年代，同时执行动态年龄判定，如果Survivor空间中相同年龄的所有对象大小总和大于Survivor的一般则这些对象可直接进入老年代。

在发生Minor GC之前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那么Minor GC可以确保是安全的。如果不成立，则虚拟机会查看HandlePromotionFailure设置值是否允许担保失败。如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者HandlePromotionFailure设置不允许冒险，那这时也要改为进行一次Full GC。

新生代使用复制收集算法，但为了内存利用率，只使用其中一个Survivor空间来作为轮换备份，因此当出现大量对象在Minor GC后仍然存活的情况（最极端的情况就是内存回收后新生代中所有对象都存活），就需要老年代进行分配担保，把Survivor无法容纳的对象直接进入老年代。与生活中的贷款担保类似，老年代要进行这样的担保，前提是老年代本身还有容纳这些对象的剩余空间，一共有多少对象会活下来在实际完成内存回收之前是无法明确知道的，所以只好取之前每一次回收晋升到老年代对象容量的平均大小值作为经验值，与老年代的剩余空间进行比较，决定是否进行Full GC来让老年代腾出更多空间。

取平均值进行比较其实仍然是一种动态概率的手段，也就是说，如果某次Minor GC存活后的对象突增，远远高于平均值的话，依然会导致担保失败（Handle Promotion Failure）。如果出现了HandlePromotionFailure失败，那就只好在失败后重新发起一次Full GC。虽然担保失败时绕的圈子是最大的，但大部分情况下都还是会将HandlePromotionFailure开关打开，避免Full GC过于频繁。

## OOM异常

### java堆溢出

java堆用于存储对象实例，只要不断创建对象并保证GC Roots到对象之间没有可达路径以避免垃圾回收，当对象数量到达最大堆容量后就会产生java堆溢出

```java
public class HeapOOM{
	static class OOMObject{
	}
	public static void main(String[] args){
	    List<OOMObject>list=new ArrayList<OOMObject>();
	    while(true){
	        list.add(new OOMObject());
	    }
	}
}
```

解决思路：

* 可能是内存泄漏， 内存泄漏指由于疏忽或错误造成程序未能释放已经不再使用的内存，内存泄漏是由于某些内存未被自动回收导致的，此时应该借助工具查看确定泄漏对象以及GC Roots引用链信息，定位泄漏位置
* 如果是真的java堆溢出则应该调整JVM堆的最大参数，或者检查代码，分析内存布局，减少对象内存 

### 虚拟机栈/本地方法栈溢出

线程请求的栈深度大于虚拟机允许的最大深度则抛出StackOverflowError(如方法无限递归)；如果虚拟机栈扩展时无法申请到足够的空间则抛出OutOfMemoryError

栈溢出经常碰到

```java
public class JacaVMStackSOF{
	private int stackLength=1;
	public void stackLeak(){
		stackLength++;
		stackLeak();
	}
	public static void main(String[] args){
		JacaVMStackSOF oom=new JacaVMStackSOF();
		try{
			oom.stackLeak();
		}catch(Throwable e){
			e.printStackTrace();
            throw e;
		}
	}
}
```

建立过多的线程同样可能造成栈溢出，每个线程分配的栈大小与栈容量参数有关，进程内存=堆+方法区+栈*线程数，如果是过多线程导致的栈溢出可以减少线程数量或者减少栈容量来换取更多的线程

## 类文件结构

Java虚拟机不和包括Java在内的任何语言绑定，它只与“Class文件”这种特定的二进制文件格式所关联，Class文件中包含了Java虚拟机指令集和符号表以及若干其他辅助信息。基于安全方面的考虑，Java虚拟机规范要求在Class文件中使用许多强制性的语法和结构化约束，但任一门功能性语言都可以表示为一个能被Java虚拟机所接受的有效的Class文件。作为一个通用的、机器无关的执行平台，任何其他语言的实现者都可以将Java虚拟机作为语言的产品交付媒介。例如，使用Java编译器可以把Java代码编译为存储字节码的Class文件，使用JRuby等其他语言的编译器一样可以把程序代码编译成Class文件，虚拟机并不关心Class的来源是何种语言。

任何一个Class文件都对应着唯一一个类或接口的定义信息，但反过来说，类或接口并不一定都得定义在文件里（譬如类或接口也可以通过类加载器直接生成）。

### class文件结构

Class文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在Class文件之中，中间没有添加任何分隔符，这使得整个Class文件中存储的内容几乎全部是程序运行的必要数据，没有空隙存在。当遇到需要占用8位字节以上空间的数据项时，则会按照高位在前的方式分割成若干个8位字节进行存储。

根据Java虚拟机规范的规定，Class文件格式采用一种类似于C语言结构体的伪结构来存储数据，这种伪结构中只有两种数据类型：无符号数和表

**无符号数**属于基本的数据类型，以u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节和8个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值。

**表**是由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性地以"_info"结尾。表用于描述有层次关系的复合结构的数据，整个Class文件本质上就是一张表

无论是无符号数还是表，当需要描述同一类型但数量不定的多个数据时，经常会使用一个前置的容量计数器加若干个连续的数据项的形式，这时称这一系列连续的某一类型的数据为某一类型的集合。

**魔数**

每个Class文件的头4个字节称为**魔数（Magic Number）**，它的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。很多文件存储标准中都使用魔数来进行身份识别，譬如图片格式，如gif或者jpeg等在文件头中都存有魔数。

**版本号**

紧接着魔数的4个字节存储的是Class文件的版本号：第5和第6个字节是**次版本号（Minor Version）**，第7和第8个字节是**主版本号（Major Version）**。Java的版本号是从45开始的，JDK 1.1之后的每个JDK大版本发布主版本号向上加1（JDK 1.0\~1.1使用了45.0~45.3的版本号），高版本的JDK能向下兼容以前版本的Class文件，但不能运行以后版本的Class文件，即使文件格式并未发生任何变化，虚拟机也必须拒绝执行超过其版本号的Class文件。

紧接着主次版本号之后的是常量池入口，常量池可以理解为Class文件之中的资源仓库，它是Class文件结构中与其他项目关联最多的数据类型，也是占用Class文件空间最大的数据项目之一，同时它还是在Class文件中第一个出现的表类型数据项目。

**常量池**

紧接着主次版本号之后的是**常量池入口**，常量池可以理解为Class文件之中的资源仓库，它是Class文件结构中与其他项目关联最多的数据类型，也是占用Class文件空间最大的数据项目之一，同时它还是在Class文件中第一个出现的表类型数据项目。

由于常量池中常量的数量是不固定的，所以在常量池的入口需要放置一项u2类型的数据，代表常量池容量计数值（constant_pool_count）。与Java中语言习惯不一样的是，这个容量计数是从1而不是0开始的。在Class文件格式规范制定之时，设计者将第0项常量空出来是有特殊考虑的，这样做的目的在于满足后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的含义，这种情况就可以把索引值置为0来表示。

Class文件结构中只有常量池的容量计数是从1开始，对于其他集合类型，包括接口索引集合、字段表集合、方法表集合等的容量计数都与一般习惯相同，是从0开始的。

常量池中主要存放两大类常量：字面量（Literal）和符号引用（Symbolic References）。字面量比较接近于Java语言层面的常量概念，如文本字符串、声明为final的常量值等。而符号引用则属于编译原理方面的概念，包括了下面三类常量：

* 类和接口的全限定名（Fully Qualified Name）
* 字段的名称和描述符（Descriptor）
* 方法的名称和描述符

Java代码在进行Javac编译的时候，并不像C和C++那样有“连接”这一步骤，而是在虚拟机加载Class文件的时候进行动态连接。也就是说，在Class文件中不会保存各个方法、字段的最终内存布局信息，因此这些字段、方法的符号引用不经过运行期转换的话无法得到真正的内存入口地址，也就无法直接被虚拟机使用。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析、翻译到具体的内存地址之中。

**访问标志**

在常量池结束之后，紧接着的两个字节代表**访问标志（access_flags）**，这个标志用于识别一些类或者接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是类的话，是否被声明为final等。access_flags中一共有16个标志位可以使用，当前只定义了其中8个，没有使用到的标志位要求一律为0。

**类索引、父类索引与接口索引集合**

**类索引（this_class）**和**父类索引（super_class）**都是一个u2类型的数据，而**接口索引集合**（interfaces）是一组u2类型的数据的集合，Class文件中由这三项数据来确定这个类的继承关系。类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名。由于Java语言不允许多重继承，所以父类索引只有一个，除了java.lang.Object之外，所有的Java类都有父类，因此除了java.lang.Object外，所有Java类的父类索引都不为0。接口索引集合就用来描述这个类实现了哪些接口，这些被实现的接口将按implements语句（如果这个类本身是一个接口，则应当是extends语句）后的接口顺序从左到右排列在接口索引集合中。

类索引、父类索引和接口索引集合都按顺序排列在访问标志之后，类索引和父类索引用两个u2类型的索引值表示，它们各自指向一个类型为CONSTANT_Class_info的类描述符常量，通过CONSTANT_Class_info类型的常量中的索引值可以找到定义在CONSTANT_Utf8_info类型的常量中的全限定名字符串。对于接口索引集合，入口的第一项——u2类型的数据为接口计数器（interfaces_count），表示索引表的容量。如果该类没有实现任何接口，则该计数器值为0，后面接口的索引表不再占用任何字节。

**字段表集合**

**字段表（field_info）**用于描述接口或者类中声明的变量。字段（field）包括类级变量以及实例级变量，但不包括在方法内部声明的局部变量。我们可以想一想在Java中描述一个字段可以包含什么信息？可以包括的信息有：字段的作用域（public、private、protected修饰符）、是实例变量还是类变量（static修饰符）、可变性（final）、并发可见性（volatile修饰符，是否强制从主内存读写）、可否被序列化（transient修饰符）、字段数据类型（基本类型、对象、数组）、字段名称。上述这些信息中，各个修饰符都是布尔值，要么有某个修饰符，要么没有，很适合使用标志位来表示。而字段叫什么名字、字段被定义为什么数据类型，这些都是无法固定的，只能引用常量池中的常量来描述。

**方法表集合**

Class文件存储格式中对方法的描述与对字段的描述几乎采用了完全一致的方式，**方法表**的结构如同字段表一样，依次包括了访问标志（access_flags）、名称索引（name_index）、描述符索引（descriptor_index）、属性表集合（attributes）几项。

方法里的Java代码，经过编译器编译成字节码指令后，存放在方法属性表集合中一个名为"Code"的属性里面

**属性表集合**

在Class文件、字段表、方法表都可以携带自己的属性表集合，以用于描述某些场景专有的信息。

与Class文件中其他的数据项目要求严格的顺序、长度和内容不同，属性表集合的限制稍微宽松了一些，不再要求各个属性表具有严格顺序，并且只要不与已有属性名重复，任何人实现的编译器都可以向属性表中写入自己定义的属性信息，Java虚拟机运行时会忽略掉它不认识的属性。为了能正确解析Class文件，《Java虚拟机规范（第2版）》中预定义了9项虚拟机实现应当能识别的属性，而在最新的《Java虚拟机规范（Java SE 7）》版中，预定义属性已经增加到21项

### 字节码指令

Java虚拟机的指令由一个字节长度的、代表着某种特定操作含义的数字（称为操作码，Opcode）以及跟随其后的零至多个代表此操作所需参数（称为操作数，Operands）而构成。由于Java虚拟机采用面向操作数栈而不是寄存器的架构，所以大多数的指令都不包含操作数，只有一个操作码。

字节码指令集是一种具有鲜明特点、优劣势都很突出的指令集架构，由于限制了Java虚拟机操作码的长度为一个字节（即0~255），这意味着指令集的操作码总数不可能超过256条；又由于Class文件格式放弃了编译后代码的操作数长度对齐，这就意味着虚拟机处理那些超过一个字节数据的时候，不得不在运行时从字节中重建出具体数据的结构，如果要将一个16位长度的无符号整数使用两个无符号字节存储起来（将它们命名为byte1和byte2），那它们的值应该是这样的：

（byte1＜＜8）|byte2

这种操作在某种程度上会导致解释执行字节码时损失一些性能。但这样做的优势也非常明显，放弃了操作数长度对齐，就意味着可以省略很多填充和间隔符号；用一个字节来代表操作码，也是为了尽可能获得短小精干的编译代码。这种追求尽可能小数据量、高传输效率的设计是由Java语言设计之初面向网络、智能家电的技术背景所决定的，并一直沿用至今。

如果不考虑异常处理的话，那么Java虚拟机的解释器可以使用下面这个伪代码当做最基本的执行模型来理解，这个执行模型虽然很简单，但依然可以有效地工作：

```
do{
自动计算PC寄存器的值加1；
根据PC寄存器的指示位置，从字节码流中取出操作码；
if（字节码存在操作数）从字节码流中取出操作数；
执行操作码所定义的操作；
}while（字节码流长度＞0）；
```

在Java虚拟机的指令集中，大多数的指令都包含了其操作所对应的数据类型信息。例如，iload指令用于从局部变量表中加载int型的数据到操作数栈中，而fload指令加载的则是float类型的数据。这两条指令的操作在虚拟机内部可能会是由同一段代码来实现的，但在Class文件中它们必须拥有各自独立的操作码。

对于大部分与数据类型相关的字节码指令，它们的操作码助记符中都有特殊的字符来表明专门为哪种数据类型服务：i代表对int类型的数据操作，l代表long,s代表short,b代表byte,c代表char,f代表float,d代表double,a代表reference。也有一些指令的助记符中没有明确地指明操作类型的字母，如arraylength指令，它没有代表数据类型的特殊字符，但操作数永远只能是一个数组类型的对象。还有另外一些指令，如无条件跳转指令goto则是与数据类型无关的。

由于Java虚拟机的操作码长度只有一个字节，所以包含了数据类型的操作码就为指令集的设计带来了很大的压力：如果每一种与数据类型相关的指令都支持Java虚拟机所有运行时数据类型的话，那指令的数量恐怕就会超出一个字节所能表示的数量范围了。因此，Java虚拟机的指令集对于特定的操作只提供了有限的类型相关指令去支持它，换句话说，指令集将会故意被设计成非完全独立的（Java虚拟机规范中把这种特性称为"Not Orthogonal"，即并非每种数据类型和每一种操作都有对应的指令）。有一些单独的指令可以在必要的时候用来将一些不支持的类型转换为可被支持的类型。

大部分的指令都没有支持整数类型byte、char和short，甚至没有任何指令支持boolean类型。编译器会在编译期或运行期将byte和short类型的数据带符号扩展（Sign-Extend）为相应的int类型数据，将boolean和char类型数据零位扩展（Zero-Extend）为相应的int类型数据。与之类似，在处理boolean、byte、short和char类型的数组时，也会转换为使用对应的int类型的字节码指令来处理。因此，大多数对于boolean、byte、short和char类型数据的操作，实际上都是使用相应的int类型作为运算类型（Computational Type）。

## 虚拟机类加载机制

虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。

与那些在编译时需要进行连接工作的语言不同，在Java语言里面，类型的加载、连接和初始化过程都是在程序运行期间完成的，这种策略虽然会令类加载时稍微增加一些性能开销，但是会为Java应用程序提供高度的灵活性，Java里天生可以动态扩展的语言特性就是依赖运行期动态加载和动态连接这个特点实现的。例如，如果编写一个面向接口的应用程序，可以等到运行时再指定其实际的实现类；用户可以通过Java预定义的和自定义类加载器，让一个本地的应用程序可以在运行时从网络或其他地方加载一个二进制流作为程序代码的一部分，这种组装应用程序的方式目前已广泛应用于Java程序之中。从最基础的Applet、JSP到相对复杂的OSGi技术，都使用了Java语言运行期类加载的特性。

### 类加载

类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading）7个阶段。其中验证、准备、解析3个部分统称为连接（Linking）

<img src="https://i.loli.net/2020/02/19/5brCV3KhYai6QuP.png" alt="image.png" style="zoom: 67%;" />

**加载**

Java虚拟机规范中并没有进行强制约束什么情况下需要开始类加载过程的第一个阶段，这点可以交给虚拟机的具体实现来自由把握。但是对于初始化阶段，虚拟机规范则是严格规定了有且只有5种情况必须立即对类进行“初始化”（而加载、验证、准备自然需要在此之前开始）：

1）遇到new、getstatic、putstatic或invokestatic这4条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或设置一个类的静态字段（被final修饰、已在编译期把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。

2）使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。

3）当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。

4）当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。

5）当使用JDK 1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

对于这5种会触发类进行初始化的场景，虚拟机规范中使用了一个很强烈的限定语：“有且只有”，这5种场景中的行为称为对一个类进行主动引用。除此之外，所有引用类的方式都不会触发初始化，称为被动引用。

在加载阶段，虚拟机需要完成以下3件事情：

1）通过一个类的全限定名来获取定义此类的二进制字节流。

2）将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。

3）在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

相对于类加载过程的其他阶段，一个**非数组类**的加载阶段（准确地说，是加载阶段中获取类的二进制字节流的动作）是开发人员可控性最强的，因为加载阶段既可以使用系统提供的引导类加载器来完成，也可以由用户自定义的类加载器去完成，开发人员可以通过定义自己的类加载器去控制字节流的获取方式（即重写一个类加载器的loadClass()方法）。

对于**数组类**而言，情况就有所不同，数组类本身不通过类加载器创建，它是由Java虚拟机直接创建的。但数组类与类加载器仍然有很密切的关系，因为数组类的元素类型（Element Type，指的是数组去掉所有维度的类型）最终是要靠类加载器去创建，一个数组类（下面简称为C）创建过程就遵循以下规则：

如果数组的组件类型（Component Type，指的是数组去掉一个维度的类型）是引用类型，那就递归采用本节中定义的加载过程去加载这个组件类型，数组C将在加载该组件类型的类加载器的类名称空间上被标识。如果数组的组件类型不是引用类型（例如int[]数组），Java虚拟机将会把数组C标记为与引导类加载器关联。

加载阶段完成后，虚拟机外部的二进制字节流就按照虚拟机所需的格式存储在方法区之中，方法区中的数据存储格式由虚拟机实现自行定义，虚拟机规范未规定此区域的具体数据结构。然后在内存中实例化一个java.lang.Class类的对象（并没有明确规定是在Java堆中，对于HotSpot虚拟机而言，Class对象比较特殊，它虽然是对象，但是存放在方法区里面），这个对象将作为程序访问方法区中的这些类型数据的外部接口。

**验证**

验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

Java语言本身是相对安全的语言（依然是相对于C/C++来说），使用纯粹的Java代码无法做到诸如访问数组边界以外的数据、将一个对象转型为它并未实现的类型、跳转到不存在的代码行之类的事情，如果这样做了，编译器将拒绝编译。但前面已经说过，Class文件并不一定要求用Java源码编译而来，可以使用任何途径产生，甚至包括用十六进制编辑器直接编写来产生Class文件。在字节码语言层面上，上述Java代码无法做到的事情都是可以实现的，至少语义上是可以表达出来的。虚拟机如果不检查输入的字节流，对其完全信任的话，很可能会因为载入了有害的字节流而导致系统崩溃，所以验证是虚拟机对自身保护的一项重要工作。

整体上看，验证阶段大致上会完成下面4个阶段的检验动作：文件格式验证、元数据验证、字节码验证、符号引用验证。

1.文件格式验证

第一阶段要验证字节流是否符合Class文件格式的规范，并且能被当前版本的虚拟机处理。该验证阶段的主要目的是保证输入的字节流能正确地解析并存储于方法区之内，格式上符合描述一个Java类型信息的要求。这阶段的验证是基于二进制字节流进行的，只有通过了这个阶段的验证后，字节流才会进入内存的方法区中进行存储，所以后面的3个验证阶段全部是基于方法区的存储结构进行的，不会再直接操作字节流。

2.元数据验证

第二阶段是对字节码描述的信息进行语义分析，以保证其描述的信息符合Java语言规范的要求。第二阶段的主要目的是对类的元数据信息进行语义校验，保证不存在不符合Java语言规范的元数据信息。

3.字节码验证

第三阶段是整个验证过程中最复杂的一个阶段，主要目的是通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。在第二阶段对元数据信息中的数据类型做完校验后，这个阶段将对类的方法体进行校验分析，保证被校验类的方法在运行时不会做出危害虚拟机安全的事件。

4.符号引用验证

最后一个阶段的校验发生在虚拟机将符号引用转化为直接引用的时候，这个转化动作将在连接的第三阶段——解析阶段中发生。符号引用验证可以看做是对类自身以外（常量池中的各种符号引用）的信息进行匹配性校验。符号引用验证的目的是确保解析动作能正常执行，如果无法通过符号引用验证，那么将会抛出一个java.lang.IncompatibleClassChangeError异常的子类，如java.lang.IllegalAccessError、java.lang.NoSuchFieldError、java.lang.NoSuchMethodError等。

对于虚拟机的类加载机制来说，验证阶段是一个非常重要的、但不是一定必要（因为对程序运行期没有影响）的阶段。如果所运行的全部代码（包括自己编写的及第三方包中的代码）都已经被反复使用和验证过，那么在实施阶段就可以考虑使用-Xverify:none参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。

**准备**

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。这个阶段中有两个容易产生混淆的概念需要强调一下，首先，这时候进行内存分配的仅包括类变量（被static修饰的变量），而不包括实例变量，实例变量将会在对象实例化时随着对象一起分配在Java堆中。其次，这里所说的初始值“通常情况”下是数据类型的零值，假设一个类变量的定义为：

```java
public static int value=123；
```

那变量value在准备阶段过后的初始值为0而不是123，因为这时候尚未开始执行任何Java方法，而把value赋值为123的putstatic指令是程序被编译后，存放于类构造器＜clinit＞()方法之中，所以把value赋值为123的动作将在初始化阶段才会执行。

**解析**

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，符号引用在在Class文件中以CONSTANT_Class_info、CONSTANT_Fieldref_info、CONSTANT_Methodref_info等类型的常量出现。

符号引用（Symbolic References）：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经加载到内存中。各种虚拟机实现的内存布局可以各不相同，但是它们能接受的符号引用必须都是一致的，因为符号引用的字面量形式明确定义在Java虚拟机规范的Class文件格式中。

直接引用（Direct References）：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。直接引用是和虚拟机实现的内存布局相关的，同一个符号引用在不同虚拟机实例上翻译出来的直接引用一般不会相同。如果有了直接引用，那引用的目标必定已经在内存中存在。

**初始化**

类初始化阶段是类加载过程的最后一步，前面的类加载过程中，除了在加载阶段用户应用程序可以通过自定义类加载器参与之外，其余动作完全由虚拟机主导和控制。到了初始化阶段，才真正开始执行类中定义的Java程序代码（或者说是字节码）。

在准备阶段，变量已经赋过一次系统要求的初始值，而在初始化阶段，则根据程序员通过程序制定的主观计划去初始化类变量和其他资源，或者可以从另外一个角度来表达：初始化阶段是执行类构造器＜clinit＞()方法的过程。

### 类加载器

虚拟机设计团队把类加载阶段中的“通过一个类的全限定名来获取描述此类的二进制字节流”这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类。实现这个动作的代码模块称为“类加载器”。

类加载器虽然只用于实现类的加载动作，但它在Java程序中起到的作用却远远不限于类加载阶段。对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间。这句话可以表达得更通俗一些：比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义，否则，即使这两个类来源于同一个Class文件，被同一个虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等。