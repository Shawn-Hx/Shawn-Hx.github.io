---
title: Java 内存模型
tags:
- JVM
- Notes
---

> “并发处理的广泛应用是使得Amdahl定律代替摩尔定律成为计算机性能发展源动力的根本原因，也是人类“压榨”计算机运算能力的最有力武器。“
> 
> ——周志明《深入理解Java虚拟机：JVM高级特性与最佳实践（第2版）》

<!--more-->

以下内容整理自周志明《深入理解Java虚拟机：JVM高级特性与最佳实践（第2版）》

## 内存模型 ##

在**物理计算机**中，为了解决缓存一致性问题，需要各个处理器访问缓存时都遵循一些协议，在读写时要根据协议来进行操作，这类协议有MSI、MESI、MOSI、Synapse、Firefly、Dragon Protocol等。

内存模型可以理解为在特定的操作协议下，对特定的内存或高速缓存进行读写访问的过程抽象。

## Java内存模型 ##

Java内存模型（Java Memory Model，JMM）试图屏蔽各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。

> 注：这里和以下所说的Java内存模型都特指在JDK1.2之后建立起来并在JDK1.5中完备过的内存模型。

## 主内存与工作内存 ##

Java内存模型的主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节。

**变量（Variables）**：与Java编程中的变量有所区别，它包括实例字段、静态字段和构成数组对象的元素，不包括局部变量与方法参数，因为后者是线程私有的。

> 注：如果局部变量是一个reference类型，它引用的对象在Java堆中可被各个线程共享，但是reference本身在Java栈的局部变量表中，它是线程私有的。

**主内存（Main Memory）**：所有的变量都存储在主内存中，主内存与物理硬件的主内存可以互相类比，但此处只是虚拟机内存的一部分。

**工作内存（Working Memory）**：每条线程有自己的工作内存，工作内存可与处理器高速缓存类比，线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，不能直接读写主内存中的变量。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成。

> 注：关于主内存副本拷贝，对象的引用、对象中某个在线程中访问到的字段是有可能存在拷贝的，但不会有虚拟机实现成把整个对象拷贝一次。

## 内存间交互操作 ##

关于主内存与工作内存之间具体的交互协议，Java内存模型定义了以下8中操作来完成：

- **lock（锁定）**
	
	作用于主内存变量，把一个变量标识为一条线程独占的状态。
- **unlock（解锁）**
	
	作用于主内存变量，把一个处于锁定状态的变量释放，变量释放后才可以被其他线程锁定。
- **read（读取）**
	
	作用于主内存变量，把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用。
- **load（载入）**
	
	作用于工作内存变量，把read操作从主内存得到的变量值放入工作内存的变量副本中。
- **use（使用）**
	
	作用于工作内存变量，把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量值的字节码指令时将会执行这个操作。
- **assign（赋值）**
	
	作用于工作内存变量，把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- **store（存储）**
	
	作用于工作内存变量，把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用。
- **write（写入）**
	
	作用于主内存变量，把store操作从工作内存中得到的变量值放入主内存的变量中。

把一个变量从主内存复制到工作内存，要顺序地执行read和load操作，把一个变量从工作内存同步回主内存，要顺序地执行store和write操作。Java内存模型只要求上述两个操作必须*按顺序执行*，而没有保证是*连续执行*。

Java内存模型还规定了在执行上述8种基本操作时必须满足如下规则：

- 不允许read和load、store和write操作之一单独出现
- 不允许一个线程丢弃它最近的assign操作，即变量在工作内存中改变了之后必须把该变化同步回主内存
- 不允许一个线程无原因地（没有发生过任何assign操作）把数据从其工作内存同步回主内存中
- 一个新的变量只能在主内存中“诞生”，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量
- 一个变量在同一个时刻只允许一条线程对其进行lock操作，但lock操作可以被同一条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁
- 如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行load或assign操作初始化变量的值
- 如果一个变量事先没有被lock操作锁定，那就不允许对它执行unlock操作，也不允许去unlock一个被其他线程锁定住的变量
- 对一个变量执行unlock操作之前，必须先把此变量同步回主内存中

这8种内存访问操作以及上述规则限定，再加上volatile的一些特殊规定，就已经完全确定了Java程序中哪些内存访问操作在并发下是安全的。

## 特殊规则——volatile变量 ##

关键字**volatile**可以说是Java虚拟机提供的最轻量级的同步机制。Java内存模型对volatile专门定义了一些特殊的访问规则。

### volatile 关键字的作用 ###

#### 1. 保证变量对所有线程的可见性 ####

- 可见性：当一条线程修改了这个变量的值，新值对于其他线程来说是可以立即得知的
- volatile变量只能保证可见性，Java中的运算并非原子操作，会导致volatile变量的运算在并发下是不安全的

volatile变量的运算在并发下不安全的例子：
```java
public class VolatileTest {
    public static volatile int race = 0;
    
    public static void increase() {
        race++;
    }
    
    private static final int THREADS_COUNT = 20;
    
    public static void main(String[] args) {
        Thread[] threads = new Thread[THREADS_COUNT];
        for (int i = 0; i < THREADS_COUNT; i++) {
            // JDK1.8后可用Lambda表达式
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int j = 0; j < 10000; j++) {
                        increase();
                    }
                }
            });
            threads[i].start();
        }
        // 如果使用IDEA，这里的1需要改为2，因为多了一个Monitor Ctrl-Break线程
        while (Thread.activeCount() > 1) {
            Thread.yield();
        }
        System.out.println(race);
    }
}
```
在这段代码中发起了20个线程，每个线程对race变量进行10000次自增操作，但运行程序输出的结果都是一个小于200000的数字。由于自增运算`race++`在Class文件中由4条字节码指令构成，当`getstatic`指令把race的值取到操作栈顶时，`volatile`关键字保证了race的值在此时是正确的，但是执行`iconst_1`、`iadd`指令时，其他线程可能已经把race的值加大了，操作栈顶的值变成了过期的数据，所以`putstatic`指令执行后就会把较小的race值同步回主内存中。

#### 2. 禁止指令重排序优化 ####

普通变量仅仅保证在该方法执行过程中所有依赖赋值结果的地方都能获取到正确的结果，而不能保证变量赋值操作的顺序与程序代码中的执行顺序一致。

指令重排序干扰程序并发执行的例子：
```java
Map configOptions;
char[] configText;
volatile boolean initialized = false;

// 假设以下代码在线程A中执行
// 模拟读取配置信息，当读取完成后将initialized设置为true以通知其他线程配置可用
configOptions = new HashMap();
configText = readConfigFile(fileName);
processConfigOptions(configText, configOptions);
initialized = true;

// 假设以下代码在线程B中执行
// 等待initialized，代表线程A已经把配置信息初始化完成
while (!initialized) {
    sleep();
}
// 使用线程A中初始化好的配置信息
doSomethingWithConfig();
```

在上例中，如果定义initialized变量时没有使用**`volatile`**修饰，就可能会由于指令重排序的优化，导致位于线程A中最后一句的代码`initialized = true`被提前执行，这样在线程B中使用配置信息的代码就可能出现错误。

**`volatile`**关键字禁止指令重排序的例子：
```java
public class Singleton {
    private volatile static Singleton instance;

    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
代码是一段标准的DCL单例代码，观察加入和未加入**`volatile`**关键字所生成汇编代码的差别，关键变化在于有**`volatile`**修饰的变量赋值后多执行了一个`lock addl $0x0, (%esp)`操作，这个操作相当于一个**内存屏障**（Memory Barrier，指重排序时不能把后面的指令重排序到内存屏障之前的位置），指令中把ESP寄存器的值加0是一个空操作，IA32手册规定`lock`前缀的作用是使得本CPU的Cache写入内存，该写入动作也会引起别的CPU或者别的内核无效化其Cache。通过这样一个操作，可以让volatile变量的修改对其他CPU立即可见。

**`volatile`**能禁止指令重排序是因为CPU在指令重排时需要能正确处理指令依赖情况以保障程序能得出正确的执行结果，而`lock addl $0x0, (%esp)`指令把修改同步到内存时，意味着所有之前的操作都已经执行完成，这样便形成了“指令重排序无法越过内存屏障”的效果。

### volatile 变量的特殊规则 ###
假定T表示一个线程，V和W分别表示两个volatile类型变量，那么在进行read、load、use、assign、store和write操作时需要满足如下规则：

- 只有当线程T对变量V执行的前一个动作是load的时候，线程T才能对变量V执行use动作；并且，只有当线程T对变量V执行的后一个动作是use的时候，线程T才能对变量V执行load动作。

- 只有当线程T对变量V执行的前一个操作是assign的时候，线程T才能对变量V执行store操作；并且，只有当线程T对变量V执行的后一个动作是store的时候，线程T才能对变量V执行assign动作。

- 假定动作A是线程T对变量V实施的use或assign动作，假定动作F是和动作A相关联的load或store动作，假定动作P是和动作F相应的对变量V的read或write动作；类似地，假定动作B是线程T对变量W实施的use或assign动作，假定动作G是和动作B相关联的load或store动作，假定动作Q是和动作G相应的对变量W的read和write操作，如果A先于B，那么P先于Q。

### volatile 使用总结 ###

**`volatile`**关键字的**运用场景**：

- 运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值
- 变量不需要与其他的状态变量共同参与不变约束

使用例子：
```java
volatile boolean shutdownRequested;

public void shutdown() {
    shutdownRequested = true;
}

public void doWork() {
    while (!shutdownRequested) {
        // do stuff
    }
}
```

在某些情况下，**`volatile`**的同步机制的性能确实要优于锁（使用**`synchronized`**关键字或`java.util.concurrent`包里面的锁），但是由于虚拟机对锁实行的许多消除和优化，使得我们很难量化地认为**`volatile`**就会比**`synchronized`**快多少。大多数场景下**`volatile`**的总开销要比锁低。我们在**`volatile`**与锁之中选择的唯一依据仅仅是**`volatile`**的语义能否满足使用场景的需求。

## 特殊规则——long和double型变量 ##

**long和double的非原子性协定（Nonatomic Treatment of double and long Variables）**

Java内存模型要求8个操作都具有**原子性**，但是对于64位的数据类型——long和double，在模型中特别定义了一条相对宽松的规定：允许虚拟机将没有被volatile修饰的64位数据的读写操作划分为两次32位的操作来进行，即虚拟机实现可以选择不保证64位数据类型的load、store、read和write这4个操作的原子性。

Java内存模型虽然允许虚拟机不把long和double变量的读写实现成原子操作，但**“强烈建议”**虚拟机把这些操作实现为具有原子性的操作。

在实际开发中，各种平台下的商用虚拟机几乎都选择把64位数据的读写操作作为原子操作来对待，因此编写代码时一般不需要把用到的long和double变量专门声明为volatile。

## Java内存模型特征 ##

Java内存模型是围绕着在并发过程中如何处理原子性、可见性和有序性这3个特征来建立的。

### 原子性 (Atomcity) ###

由Java内存模型直接保证的原子性变量操作包括read、load、assign、use、store和write，可以大致认为基本数据类型的访问和读写是具备原子性的。

如果应用场景需要一个更大范围的原子性保证，Java内存模型还提供了lock和unlock操作来满足这种需求，虚拟机未把lock和unlock操作直接开放给用户使用，但是却提供了更高层次的字节码指令`monitorenter`和`monitorexit`来隐式地使用这两个操作，这两个字节码指令反映到Java代码中就是同步块**synchronized**关键字，在**synchronized**块之间的操作也具备原子性。

### 可见性 (Visibility) ###

可见性是指当一个线程修改了共享变量的值，其他线程能够立即得知这个修改。

Java内存模型通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值这种依赖主内存作为传递媒介的方式实现可见性。
普通变量和volatile变量都是如此，它们的区别是，volatile变量保证了**多线程**操作时变量的可见性，普通变量不能保证这一点。

除了**volatile**之外，**synchronized**和**final**关键字也能实现可见性。

- **volatile** 实现可见性

	规则：见 [volatile变量的特殊规则]({{ post.url }}#volatile-变量的特殊规则)
- **synchronized** 实现可见性

	规则：对一个变量执行unlock操作之前，必须先把此对象同步回主内存中（执行store、write操作）。
- **final** 实现可见性

	被final修饰的字段在构造器中一旦初始化完成，并且构造器没有把“this”引用传递出去，那在其他线程就能看见final字段的值。

### 有序性 (Ordering) ###

Java程序中天然的有序性：如果在本线程内观察，所有的操作都是有序的；如果在一个线程中观察另一个线程，所有的操作都是无序的。

Java语言提供了**volatile**和**synchronized**关键字保证线程间的有序性。

- **volatile** 实现有序性

	volatile本身包含了禁止指令重排序的语义
- **synchronized** 实现有序性

	规则：一个变量在同一时刻只允许一条线程对其进行lock操作，这决定了持有同一个锁的两个同步块只能串行地进入。
	
## 先行发生原则 ##

Java语言中有一个**“先行发生”（happens-before）的原则**。这个原则是判断数据是否存在竞争、线程是否安全的主要依据。

先行发生是Java内存模型中定义的两项操作之间的偏序关系，如果说操作A先行发生于B，其实就是说在发生B操作之前，操作A产生的影响能被操作B观察到，“影响”包括修改了内存中共享变量的值、发送了消息、调用了方法等。

Java内存模型下有一些天然的先行发生关系，无需任何同步协助就已经存在。如果两个操作之间的关系不在此列，并且无法通过下列规则推导出来的话，它们就没有顺序性保障，虚拟机可以对它们随意进行重排序。

- **程序次序规则 (Program Order Rule)**
	
	**在一个线程内**，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作。准确地说应该是控制流顺序。

- **管程锁定规则 (Monitor Lock Rule)**

	一个unlock操作先行发生于后面对**同一个锁**的lock操作，“后面”是指时间上的先后顺序。

- **volatile变量规则 (Volatile Variable Rule)**

	对一个volatile变量的写操作先行发生于后面对这个变量的读操作，“后面”同样是指时间上的先后顺序。

- **线程启动规则 (Thread Start Rule)**

	Thread对象的`start()`方法先行发生于此线程的每一个动作。

- **线程终止规则 (Thread Termination Rule)**

	线程中的所有操作都先行发生于对此线程的终止检测，可以通过`Thread.join()`方法结束、`Thread.isAlive()`的返回值等手段检测到线程已经终止执行。

- **线程中断规则 (Thread Interruption Rule)**

	对线程`interrupt()`方法的调动先行发生于被中断线程的代码检测到终端事件的发生，可以通过`Thread.interrupted()`方法检测到是否有终端现象。

- **对象终结规则 (Finalizer Rule)**

	一个对象的初始化完成先行发生于它的`finalize()`方法的开始。

- **传递性 (Transitivity)**

	如果操作A先行发生于操作B，操作B先行发生于操作C，那么操作A先行发生于操作C。
	

**事件先后顺序与先行发生原则之间基本没有太大的关系，在衡量并发安全问题时，不要受到事件顺序的干扰，一切必须以先行发生原则为准。**

