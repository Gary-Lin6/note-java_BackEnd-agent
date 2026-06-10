# 一.基本概念
## 1.java 线程
线程是什么？与进程区别在？见[[处理器管理#1.进程]]，[[处理器管理#1.线程]]

其中也有提到用户线程与系统线程区别。用户线程就是用户自己创建管理的线程，系统线程是系统自己创建管理的线程。用户线程的实现要基于系统线程，用户线程与系统线程间可能是一对一，多对多等形式存在。java线程就是java实现的线程，属于用户线程
### 1.1如何创建线程
```java
//1.直接继承Thread 类
class MyThread extends Thread {
    @Override
    public void run() {
        // 线程执行的代码
    }
}

MyThread t = new MyThread();

//2.实现runable接口
MyThread t = new MyThread(new Runnable(){
	@Override
    public void run() {
        // 线程执行的代码
    }
});
//runnable接口是函数式接口，可以用lambda表达式简化

//3.实现callable接口和futuretask
    FutureTask<Integer> futureTask = new FutureTask<>(new Callable(){
		    @Override
		    public Integer(泛型，这里以整数类型举例) call() throws Exception {
	        // 线程执行的代码, 这里返回一个整型结果
	        return 1;
	    }
    });
    Thread t = new Thread(futureTask);
    t.start();

    try {
        Integer result = futureTask.get();  // 获取线程执行结果
        System.out.println("Result: " + result);
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
//同样callable也是函数式接口，可以用lambda表达式简化。

//4.线程池
ExecutorService executor = Executors.newFixedThreadPool(10);  // 创建固定大小的线程池
    for (int i = 0; i < 100; i++) {
        executor.submit(new Runnable(){
	        @Override
		    public void run() {
		        // 线程执行的代码
		    }
        });  // 提交任务到线程池执行
    }
    executor.shutdown();  // 关闭线程池
```
>[!info] Runnable与Callable的区别

Runnable没返回值，Callable有返回值。
Callable的返回值要从传给线程的futuretask里调用get方法取。futuretask运行的内容就是传给futuretask的callable实现类。

### 1.2线程的6种状态
![[Pasted image 20260518194457.png]]
### 1.3线程常用方法

>[!info] 线程运行：start vs run

调用 `start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。
但是，直接执行 `run()` 方法，会把 `run()` 方法当成一个普通方法在调用该方法的线程去执行，所以这并不是多线程工作。
```java
Thread t1;
t1.start();  //由t1执行run
t1.run();    //由当前线程执行run
```

>[!info] sleep vs yield

sleep:调用sleep会使线程从running到timed_waiting等待态，直到sleep时间耗尽后重返runnable状态
yield：线程主动退出时间片，让操作系统调用其他线程。状态：running->ready
```java
Thread.sleep();  //执行这句的线程休眠
Thread.yield();  //执行这句的线程等待调度

//如果调用的是别人的sleep，yield方法呢？
//结论是：还是自己休眠，因为sleep和yield是静态方法，只影响调用者。
```

>[!info] join

**为什么要用join**
线程间可能是同步关系，需要等待别的线程完成后再继续
```java
Thread t1;
t1.join();   //等待t1结束，等待期间，调用线程阻塞：runable->waiting
```
join底层实现是使用wait。

>[!info] interrupt

**interrupt作用**
顾名思义，起打断作用，但实际上，真的是直接打断线程运行吗？

**interrupt原理**
仅给线程打断标记设为true，不会立刻停止线程，只是发信号，线程自己决定停不停。
配合：isInterrupted()：查看中断状态（打断标记），**不清除标记**
interrupted()：查看中断状态，**用完立刻清空标记**

**不同情况下的interrupt**
- 正常运行下打断：仅**打上中断标记**，线程**不会停止**，继续正常跑
- 阻塞状态下（sleep，join，wait）：**立刻抛出 InterruptedException**，并**自动清除中断标记**（标记变回 false）

>[!info] wait/notify（先看完线程安全问题）

wait/notify是在管程（monitor)基础上实现的。
当线程加锁成功：进入到synchronize内部时，可以使用wait（）让自己进入waitSet中。然后让出锁的使用权，让其他线程能抢夺锁。
直到某个线程抢锁成功后，可以使用notify/notifyAll，将在waitSet里的线程唤醒。被唤醒的线程不是马上抢到锁，而是进入EntryList中等待锁的释放。
- notify：随机唤醒一个waitSet里的线程
- notifyAll：唤醒waitSet里的所有线程

>[!Info] park&unpark

park，unpark是LockSupport类里的两个方法。使用park（），可以将线程暂停。让出cpu。使用unpark（t1），可以将park住的t1线程释放。
特点：先unpark要park的线程，后要park的线程不会被阻塞住。

**原理**
每个线程有自己的parker对象
parker里有三个变量：mutex，condition，counter。
counter值初始为0。
park：
- park时会检查counter值，如果为0，则进入condition阻塞住。
- 如果为1，则将counter值设为0，并继续运行
unpark（t1）：
- unpark会检查t1的parker对象里的counter，如果为1，则不管（说明多个unpark也最多设置condition为1）。
- 如果为0，则检查condition里是否有线程在阻塞。
	- 如果有：则唤醒。
	- 如果没有，则将counter设置为1.
## 2.线程安全问题
多线程要实现线程安全，要满足三个性质

- 原子性：提供互斥访问，同一时刻只能有一个线程对数据进行操作，在Java中使用了atomic包（这个包提供了一些支持原子操作的类，这些类可以在多线程环境下保证操作的原子性）和synchronized关键字来确保原子性；
- 可见性：一个线程对主内存的修改可以及时地被其他线程看到，在Java中使用了synchronized和volatile这两个关键字确保可见性；
- 有序性：一个线程观察其他线程中的指令执行顺序，由于指令重排序，该观察结果一般杂乱无序，在Java中使用了happens-before原则来确保有序性。

为满足这些性质，java为我们实现了synchronize和volatile关键字

### 2.1synchronize

#### 2.1.1synchronize怎么用

```java
//1.加在普通方法：
synchronized void method() { //业务代码 }
//锁的是调用该方法的实例对象

//2.加在静态方法上
synchronized static void method() { //业务代码 }
//锁的是类对象

//3.修饰代码块
synchronized(object) { //业务代码 }
//锁的是()里的object对象
```

#### 2.1.2synchronize底层原理
synchronize底层主要由markword+monitor实现

>[!info] 什么是Mark Word

java里，一切均为对象。而储存对象除了储存自身数据外，还有额外4个字节：markword。
![[Pasted image 20260518203011.png]]
它标识着当前对象的状态

>[!info] 什么是monitor

monitor：管程。每个对象都关联了一个monitor管程。
（java管程是由c实现的）[[处理器管理#3.2管程]]
![[Pasted image 20260518203424.png]]

>[!info] synchronize加重量级锁过程

当使用synchronize对一个对象加锁时，会检查该对象的MarkWord，

如果MarkWord指示为Normal状态，则该对象头的MarkWord会设计成Heavyweight Lock状态，其中ptr_to_heavyweight_monitor为指向该管程的指针。并且将该Monitor的Owner设置成成功使用synchronize的线程。

后续如果有其他线程也要synchronize这个对象，通过检查该对象的MarkWord，发现该对象上了重量锁，到管程一看，
- Owner不是自己，则进入EntryList，并由runnable->blocked
- 如果Owner是自己，则也可以继续执行下面代码（Monitor内部有个计数器，重入则计数+1）
**因此synchronize是可重入锁**

释放锁时，计数器-1，直到计数器为0，将EntryList里的线程唤醒，由操作系统重新调度。

#### 2.1.3synchronize优化
**为什么要优化**
早期synchronize加的都是重量级锁，前面我们说了synchronzie底层是由markword+monitor实现的，其中monitor的实现，依赖于操作系统内核。因此每次加锁解锁都要有用户态和内核态之间的转换，开销大。

但是，经过统计，发现大多数锁，要么全程无竞争，要么是同个线程反复用。因此java实现实现了锁升级机制：无锁->偏向锁->轻量级锁->重量级锁。以实现性能得提升

这不同类型锁是由MarkWord实现的：
无锁时：Normal状态，MarkWord如图
![[Pasted image 20260518205301.png]]

>[!info] 偏向锁

当第一个线程进入同步块时，锁会升级到**偏向锁**。此时会通过CAS操作（后面会讲）在对象头的Mark Word中记录该线程ID，锁标志位保持01，但偏向标志位变为1。
![[Pasted image 20260518205450.png]]
当有别的线程进入时，只用判断thread是否为自己，如果是，则直接进入，如果不是则要进行撤销偏向锁的工作：撤销过程需要等到安全点（所有线程都可以停止的状态），暂停持有偏向锁的线程，检查该线程是否还在执行同步代码，然后清除偏向标记，这个过程开销不小。
因此偏向锁后来也被停用甚至废除

>[!info] 轻量级锁

撤销完偏向锁后，锁升级成轻量级锁
这时JVM会在当前线程的栈帧中创建锁记录空间，把对象头的Mark Word复制进去，然后通过CAS操作尝试把对象头的Mark Word更新为指向锁记录的指针，锁标志位变为00。
![[Pasted image 20260518210139.png]]
如果CAS成功，说明获取轻量级锁成功。
如果CAS失败，说明有其他线程在竞争，则通过自旋来不断重试CAS操作，期望在短时间内持有锁的线程能释放锁

自旋次数：
- 早期：在JDK1.6之前自旋次数固定是10次，可以通过参数调整。
- DK1.6之后引入了自适应自旋：根据上次上锁情况自动调节上锁次数

如果自旋超过一定次数，或者一个线程持有锁，一个线程在自旋，还有个线程参与锁竞争时。那么锁就会膨胀为重量级锁。

**为什么要自旋**
重量级锁面对多个线程竞争时，会阻塞其他线程。而自旋能使线程不阻塞，适用于锁能快速释放的情景。



重量级锁：2.1.2已经讲过

#### 2.1.4synchronize与线程安全
synchronize实现了原子性：通过加锁，保证临界区操作线程间互斥
可见性：synchronize内对共享变量的修改对别的线程可见
有序性：synchronize区域内的代码不可重排列

### 2.2volatile
#### 2.2.1volatile是什么，有什么用
volatile 修饰变量，能保证线程每次取该变量都是从内存里面取，保证了可见性。
volatile所修饰的变量前后会加读写屏障，保证指令不会重排，保证了有序性。

#### 2.2.2保证可见性原理

**要讲明白这件事，先要了解，为什么会不可见。**

就和cpu与内存间有缓存一样，线程与内存间也有“缓存”（该缓存其实并不存在，是一个逻辑概念），对一个重复使用的数据，线程可能会把它存在自己线程空间中，方便下次使用。正因如此，如果该数据是一个共享数据x，假设有两个线程t1,t2。

如果t1修改了x为x1,他会修改本地和内存。但t2要读取x，直接从本地读取，发现还是x。造成t1对共享数据的修改对t2不可见。

**volatile怎么保证可见性？**
对于用volatile修饰的变量，修改和读取都要求到内存中读即可。

#### 2.2.3保证有序性原理

**同样的，为什么指令会重排？代码不是写好的吗？**

因为对于cpu，可能有些指令的重排，不会影响结果（单线程而言），且重排后，cpu效率更高（因为一条指令其实在cpu内可以分成多个部分，每条指令流水线式执行。通过对指令的重排，可能会使cpu吞吐量更大）。
所以某些时候，指令会重排。

**volatile怎么保证有序性**

JMM（Java 内存模型）定义了 4 种内存屏障（Memory Barrier），用于控制特定条件下的指令重排序和内存可见性：

| 屏障类型           | 指令示例                         | 说明                                                                                                                   |
| -------------- | ---------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **LoadLoad**   | `Load1; LoadLoad; Load2`     | 保证 `Load1` 的读取操作在 `Load2` 及其后续读取操作之前完成                                                                               |
| **StoreStore** | `Store1; StoreStore; Store2` | 保证 `Store1` 的写入操作对其他处理器可见（刷新到内存），先于 `Store2` 及其后续写入操作                                                                |
| **LoadStore**  | `Load1; LoadStore; Store2`   | 保证 `Load1` 的读取操作在 `Store2` 及其后续写入操作刷新到内存之前完成                                                                         |
| **StoreLoad**  | `Store1; StoreLoad; Load2`   | 保证 `Store1` 的写入操作对其他处理器可见，先于 `Load2` 及其后续读取操作。`StoreLoad` 屏障的开销是四种屏障中最大的，它同时具有其他三种屏障的效果，因此也称为 **全能屏障（Full Barrier）** |

对于对volatile变量的写操作:前后加写屏障
```java
StoreStore //保证volatile之前的操作不会跑到后面来,且前面的操作对volatile可见
volatile 写操作
StoreLoad //保证volatile之后的操作不会跑到前面来，且volatile的操作对后面的操作可见
```
对于volatile变量的读操作：后面加读屏障
```java
//可能与前面操作重排的，但因为是读操作，不会有影响，因为如果有影响，说明前面某操作与volatile修饰的变量有关，那么该操作前面也会加屏障，导致不会重排。
volatile 读操作
LoadLoad 屏障  //保证后面的操作不会到volatile前完成
LoadStore 屏障  //保证volatile读取的数据是后续数据还未更改的
```

## 3.ThreadLocal
ThreadLocal叫做线程变量，意思是ThreadLocal中填充的变量属于当前线程，该变量对其他线程而言是隔离的，也就是说该变量是当前线程独有的变量。ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。

好处：
- 线程隔离：ThreadLocal为每个线程提供了独立的变量副本，这意味着线程之间不会相互影响，可以安全地在多线程环境中使用这些变量而不必担心数据竞争或同步问题。
- 降低耦合度：在同一个线程内的多个函数或组件之间，使用ThreadLocal可以减少参数的传递，降低代码之间的耦合度，使代码更加清晰和模块化。
- 性能优势：由于ThreadLocal避免了线程间的同步开销，所以在大量线程并发执行时，相比传统的锁机制，它可以提供更好的性能。
### 3.1ThreadLocal与synchronize的区别
ThreadLocal< T >其实是与线程绑定的一个变量。ThreadLocal和Synchonized都用于解决多线程并发访问。

但是ThreadLocal与synchronized有本质的区别：

1. Synchronized用于线程间的数据共享，而ThreadLocal则用于线程间的数据隔离。

2. Synchronized是利用锁的机制，使变量或代码块在某一时该只能被一个线程访问。而ThreadLocal为每一个线程都提供了变量的副本,使得每个线程在某一时间访问到的并不是同一个对象，这样就隔离了多个线程对数据的数据共享。而Synchronized却正好相反，它用于在多个线程间通信时能够获得数据共享。

### 3.2原理
每个Thread里维护着一个ThreadLocalMap，这个map类似与Hashmap，当ThreadLocal调用set方法时，就会往这个Map里put了一个Entry，这个Entry的key就是ThreadLocal，value就是set的值。因此，当调用ThreadLocal的get方法时，就会拿出当前Thread里的ThreadLocalMap，根据ThreadLocal来把变量拿出来。
因为每个Thread的ThreadLocalMap不同，所以不同Thread放和取都是放在不同的Thread里，这也使得线程间取ThreadLocal的值不同，保证了线程隔离。
![[Pasted image 20260610213005.png|568]]
### 3.3ThreadLocal的内存泄漏问题
实际上 ThreadLocalMap 中使用的 key 为 ThreadLocal 的弱引用，弱引用的特点是，如果这个对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。

所以如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候会被清理掉的，这样一来 ThreadLocalMap中使用这个 ThreadLocal 的 key 也会被清理掉。但是，value 是强引用，不会被清理，这样一来就会出现 key 为 null 的 value。

因此，使用 ThreadLocal 的最佳实践是：使用完毕后显式调用 remove() 方法主动清理，尤其是在线程池环境下（线程不会结束，ThreadLocalMap 也就不会被 GC），不调用 remove() 几乎一定会导致内存泄漏

# 二.JUC（java.concurrent.util)
## 1.CAS（compareAndSet）
CAS 是一种乐观锁机制，它包含三个操作数：内存位置（V）、预期值（A）和新值（B）。CAS 操作的逻辑是，如果内存位置 V 的值等于预期值 A，则将其更新为新值 B，否则不做任何操作。整个过程是原子性的，通常由硬件指令支持，如在现代处理器上，cmpxchg 指令可以实现 CAS 操作。

### 1.1原子整数
包括AtomicIntger，AtomicBoolean，AtomicLong。
以AtomicIntger为例：通过cas保证整数操作如自增，自减，比较并设置时的原子性和线程安全性。

### 1.2原子引用
包括AtomicReference，AtomicMarkableReference,AtomicStampedReference
原子引用类，用于对对象引用进行原子操作。可以保证在多线程环境下，对对象的更新操作是原子性的，即要么全部成功，要么全部失败，不会出现数据不一致的情况。常用于实现无锁数据结构或需要对对象进行原子更新的场景。

**AtomicReference：ABA问题**
主线程用compareAndSet（）修改。
但它不能感知到共享变量是否被修改。只要别的线程改回成原来一样，那么主线程还是能cas。

为了感知到是否被修改，我们可以给变量加个版本号，每次修改会使版本号+1，这样就能感知到修改了。可以使用AtomicStampedReference类来解决。

但有时候并不关心引用变量修改了几次，只关心是否修改过，就可以使AtomicMarkableReference，根据mask是否是true/false便能判断是否修改。（mask是手动修改的）。

### 1.3原子数组
包括AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray
通过cas保证数组中的元素操作的原子性和多线程安全性。

### 1.4字段更新器
包括AtomicReferenceFieldUpdater,AtomicIntegerFieldUpdater,AtomicLongFieldUpdater。
通过cas保证对象的某个属性进行原子操作。需配合volatile使用。

### 1.5原子累加器
包括LongAdder，LongAccumulator
通过cas保证累加操作的原子性和多线程安全性。

**已经有原子整数类了，为什么还要原子累加器？**
因为原子累加器性能更高。

**原理**
```java
// 累加单元数组, 懒惰初始化 
transient volatile Cell[] cells;
// 基础值, 如果没有竞争, 则用 cas 累加这个域 
transient volatile long base;
// 在 cells 创建或扩容时, 置为 1, 表示加锁 
transient volatile int cellsBusy;

static final class Cell { 
	volatile long value; 
	Cell(long x) { 
		value = x; 
	} 
	
	// 防止缓存行伪共享 
	//@sun.misc.Contended 用来解决这个问题，它的原理是在使用此注解的对象或字段的前后各增加 128 字节大小的 padding，从而让 CPU 将对象预读至缓存时占用不同的缓存行，这样，不会造成对方缓存行的失效
	@sun.misc.Contended	
	// 最重要的方法, 用来 cas 方式进行累加, prev 表示旧值, next 表示新值 
	final boolean cas(long prev, long next) { 
		return UNSAFE.compareAndSwapLong(this, valueOffset, prev, next); 
	} 
	// 省略不重要代码 
}
```

**add方法**
![[Pasted image 20260610203024.png]]
**longAccumulate方法**
加锁：acs设置cellsBusy为1
cells数组未创建
![[Pasted image 20260610203325.png]]
cells数组创建，cell没创建（每个线程刚进入 longAccumulate 时，会尝试对应一个 cell）
![[Pasted image 20260610203829.png]]
cells数组创建，cell创建
![[Pasted image 20260610203948.png]]
**sum方法**
将所有累加单元和base值相加得到最终结果。
## 2.AQS
AbstractQueuedSynchronizer：是阻塞式锁和相关的同步器工具的框架。
属性：
- state：表示资源的状态（独享模式和共享模式）
	- 独享：只有一个线程能够访问资源
	- 共享：允许多个线程访问资源
- 等待队列：类似与Moniter的EntryList
- 条件变量：类似与Moniter的WaitSet，实现等待，唤醒机制，支持多条件变量。

子类要继承AQS父类，要实现：
- tryAquire：获取锁
- tryRelease：释放锁
- tryAcquireShared：共享模式获取锁
- tryReleaseShared：共享模式释放锁
- isHeldExeclusively：是否持有独占锁

自己实现锁implement lock：
内部类继承AQS
 - 重写tryAquire：
	- compareAndSetState(0,1):用cas方法把state改成1，表示获取锁。
	- setExclusiveOwner（Thread.currentThread()）.
- 重写 tryRelease：
	- setState（0)
	- setExclusiveOwner（null）
- 重写isHeldExeclusively：
	- state== 1？
### 2.1ReentrantLock
![[Pasted image 20260610153054.png]]
**特点**
- 可中断：lock.lockInterruptibly():如果有竞争，进入阻塞队列，可以被其他线程用interrupt方法打断阻塞。被动避免死等
- 可以设置超时时间：lock.tryLock(timeout,TimeUnit),等待设置的时间，如果超时了就不等了。同时它也可以被打断。
- 可以设置为公平锁：默认是不公平的，但可以修改。根据进入阻塞队列的顺序，先进阻塞队列的先获取到锁。
- 支持多个条件变量：await/signal。await前要先获取到锁。
- 可重入

**原理**
![[Pasted image 20260610153054.png|466]]

绿色箭头：实现接口。
红色：内部类。
紫色箭头：继承

这里只讲公平锁是怎么实现的：
公平锁重写了tryAquire，在获取锁之前，先检查AQS队列中是否有前驱节点，没有才去竞争

### 2.2ReentrantReadWriteLock
可重入的读写锁。读读操作可以并发，其他互斥。
使用：
```java
ReentrantReadWriteLock lock=new ReentrantReadWriteLock();
ReentrantReadWriteLock.ReadLock rl=lock.readLock();
ReentrantReadWriteLock.WriteLock rl=lock.writeLock();
读操作用读锁，写操作用写锁即可。
```
读写锁不支持条件变量。
重入时：读锁不能重入写锁。写锁可以重入读锁。

**原理**
表面上有两把锁，实际上是用的一个锁，但区别是state：写锁占state低16位。读写占state高16位。所以读写互斥就可以根据state高16位和低16位即可。

### 2.3StampedLock
读写锁。进一步优化读性能，在使用读锁，写锁时必须配合戳使用。
不支持条件变量和重入。

**原理**
加读写锁都会改变戳
用乐观读获取一个戳，再返回结果之前，验证戳有无效（是否被写锁改变），有效则返回，不返回则加读锁，读。



## 3.线程池
线程池的核心作用是复用线程、控制并发数，避免频繁创建销毁线程带来的性能开销。
![[Pasted image 20260610182704.png]]
线程池是与阻塞队列（任务队列）配合使用的。主线程将任务放入任务队列后。线程池会从中挑一些线程来执行任务队列中的任务。
执行过程如下。
![[Pasted image 20260610182956.png]]
**线程池参数**
- 核心线程数（corePoolSize）：默认是懒创建。当一个任务提交后，线程池发现当前线程中其他线程都在工作，且线程数没达到核心线程数时，会创建一个新线程来完成任务。且这些线程在完成任务后不会被回收。
- 最大线程数（maximumPoolSize）：最大线程数=核心线程数+救急线程数。救急线程在核心线程均已创建，且阻塞队列满了后，如果线程数还未达到最大线程数，就会创建救急线程来完成阻塞队列中的任务。当完成任务空闲后，会根据后面设置的参数销毁救急线程。
- **keepAliveTime**，**unit**：救急线程存活时间和单位。
- workQueue：阻塞队列
- threadFactory：线程创建工厂，用来给线程取名
- handler：拒绝策略，当线程数已达 `maximumPoolSize` 且工作队列已满时，对新提交任务的处理策略

**拒绝策略有哪些？**
- CallerRunsPolicy，使用线程池的调用者所在的线程去执行被拒绝的任务，除非线程池被停止或者线程池的任务队列已有空缺。
- AbortPolicy，直接抛出一个任务被线程池拒绝的异常。
- DiscardPolicy，不做任何处理，静默拒绝提交的任务。
- DiscardOldestPolicy，抛弃最老的任务，然后执行该任务。
- 自定义拒绝策略，通过实现接口可以自定义任务拒绝策略。

**核心线程数设置多少合理？**
CPU密集型：corePoolSize = CPU核数 + 1（避免过多线程竞争CPU）
IO密集型：corePoolSize = CPU核数 x 2（或更高，具体看IO等待时间）

>[!info] 怎么创建线程池：ThreadPoolExecutor

![[Pasted image 20260610185911.png]]

![[Pasted image 20260610165039.png]]
**为什么要用一个int来储存线程池状态和线程数量**
因为只用一次cas操作就可以同时修改线程池状态和线程数量

**怎么关闭线程池**
使用：shutdown，shotdownNow
- shutdown 调用后状态置为 SHUTDOWN，已提交但未开始执行的队列任务仍会按顺序继续执行直到全部完成；只对处于空闲等待状态的 worker 线程调用 interrupt()（即下方源码中的 interruptIdleWorkers()）。此后不能再往线程池中添加任何任务，否则将抛出 RejectedExecutionException 异常
- 而 shutdownNow 为STOP，并试图停止所有正在执行的线程，不再处理还在池队列中等待的任务，当然，它会返回那些未执行的任务。 它试图终止线程的方法是通过调用 Thread.interrupt() 方法来实现的，但是这种方法的作用有限，如果线程中没有sleep 、wait、Condition、定时锁等应用， interrupt()方法是无法中断当前的线程的。所以，ShutdownNow()并不代表线程池就一定立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。

以下是Executor（线程池工厂，里面有一些静态方法用于创建线程池）类中的一些线程池：
### 3.1newFixedThreadPool
```java
public static ExecutorService newFixedThreadPool(int nThreads) { 
	return new ThreadPoolExecutor(nThreads, nThreads, 
									0L, TimeUnit.MILLISECONDS,
								 new LinkedBlockingQueue<Runnable>()); 
}
```
特点：
- 核心线程数 == 最大线程数（没有救急线程被创建），因此也无需超时时间 
- 阻塞队列是无界的，可以放任意数量的任务 
- 适用于任务量已知，相对耗时的任务

### 3.2newCachedThreadPool
```java
public static ExecutorService newCachedThreadPool() { 
	return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
								 60L, TimeUnit.SECONDS,
								  new SynchronousQueue<Runnable>()); 
}
```
特点 
- 核心线程数是 0， 最大线程数是 Integer.MAX_VALUE，救急线程的空闲生存时间是 60s，意味着 
	- 全部都是救急线程（60s 后可以回收）
	- 救急线程可以无限创建
- 队列采用了 SynchronousQueue 实现特点是，它没有容量，没有线程来取是放不进去的

### 3.3newSingleThreadExecutor
```java
public static ExecutorService newSingleThreadExecutor() { 
	return new FinalizableDelegatedExecutorService 
			(new ThreadPoolExecutor(1, 1, 
									0L, TimeUnit.MILLISECONDS, 
									new LinkedBlockingQueue<Runnable>())); 
}
```
使用场景： 
- 希望多个任务排队执行。线程数固定为 1，任务数多于 1 时，会放入无界队列排队。任务执行完毕，这唯一的线程 也不会被释放。
与newFixedThreadPool(1)区别：
- 自己创建一个单线程串行执行任务，如果任务执行失败而终止那么没有任何补救措施，而线程池还会新建一 个线程，保证池的正常工作 
- Executors.newSingleThreadExecutor() 线程个数始终为1，不能修改
	- FinalizableDelegatedExecutorService 应用的是装饰器模式，只对外暴露了 ExecutorService 接口，因此不能调用 ThreadPoolExecutor 中特有的方法 
- Executors.newFixedThreadPool(1) 初始时为1，以后还可以修改
	- 对外暴露的是 ThreadPoolExecutor 对象，可以强转后调用 setCorePoolSize 等方法进行修改

### 3.4ScheduledThreadPool
可以设置定期的执行任务，它支持定时或周期性执行任务，比如每隔 10 秒钟执行一次任务，通过这个实现类设置定期执行任务的策略。

## 4.并发工具类
本质还是内部sync类继承AQS实现的
### 4.1Semaphore（信号量）
用来限制能同时访问共享资源的线程上限。理论可以看[[处理器管理#3.1信号量与PV操作]]
具体实现
- 将aqs的state设为限制共享资源的线程上限。
- Semaphore的tryAquire就是判断state-1后是否大于0，如果大于0，则通过cas-1，表示获取“锁”成功。如果小于等于0，说明资源耗尽，则进入阻塞队列。
- tryRelease就是cas将state+1，然后唤醒阻塞队列中的线程。

### 4.2 CountDownLatch（倒计时锁）
用来进行线程同步协作，等待所有线程完成倒计时。 
其中构造参数用来初始化等待计数值，await() 用来等待计数归零，countDown() 用来让计数减一。
使用await方法的线程，如果计时器不为0，则加入阻塞队列。
使用countDown的线程会使计时器-1。如果计时器减为0，则将阻塞队列中的线程释放


### 4.3CyclicBarrier
让一组线程互相等待，直到所有线程都到达某个屏障点后，再一起继续执行。与CountDownLatch不同的是，CyclicBarrier可以重复使用，当所有线程都通过屏障后，计数器会重置，可以再次用于下一轮的等待。适用于多个线程需要协同工作，在某个阶段完成后再一起进入下一个阶段的场景。

调用await方法，
- 如果线程调用await方法的线程数少于设定数，则会阻塞等待。
- 如果大于等于设定数，则所有阻塞等待线程都被释放，同时将计数重置。

## 5.并发集合类
### 5.1ConcurrentHash
线程安全的哈希映射表，用于在多线程环境下高效地存储和访问键值对。

>[!Info] HashMap死链问题（JDK7以前）

首先我们要了解JDK7以前，哈希表添加元素时，后添加的元素是添加在头节点的。
假设现在有两个线程，他们同时添加一个新元素造成扩容。对于1号节点挂的是a->b->null。且扩容后，a，b经过计算还是落在1号节点处。
1. 此时，1号线程在对1号节点迁移时，取到节点e：a->b->null。next：b->null。卡了。
2. 2号线程这时完成了扩容，扩容完后，1号节点的顺序是：b->a->null。
3. 这时1号线程恢复过来了，此时的e变成：a->null。next变成：b->a->null。
4. 但1号线程还要完成扩容啊。于是将e插入到新的哈希表中。此时1号节点变成：a->b->null，并且将next赋给e，于是e=b->a->null,next=a->null。
5. 发现变成了死循环了。
这就是死链问题。
究其原因，
- 是因为在多线程环境下使用了非线程安全的 map 集合
- JDK 8 虽然将扩容算法做了调整，不再将元素加入链表头（而是保持与扩容前一样的顺序），但仍不意味着能 够在多线程环境下能够安全扩容，还会出现其它问题（如扩容丢数据）

**实现原理JDK8**
使用细粒度的synchronize（只锁链表头节点）和cas保证线程安全。

思考：哈希表在什么时候会发生线程冲突？
首次创建，扩容，修改值时会发生线程冲突。这里逐步讲解怎么实现的

**修改值（put）**
```java
//相关属性
// 整个 ConcurrentHashMap 就是一个 Node[] 
static class Node<K,V> implements Map.Entry<K,V> {}

// 扩容时如果某个 bin 迁移完毕, 用 ForwardingNode 作为旧 table bin 的头结点 
static final class ForwardingNode<K,V> extends Node<K,V> {}
```
当调用put方法时，它会判断表是不是空，为空则调用initTable，用cas方法创建表。
随后判断表头节点为不为空，如果为空，则用cas方法创建头节点。
随后它会判断这个头节点是不是 ForwardingNode。如果是，它会帮忙扩容。
如果不是则将这个头节点锁住,然后进行更新操作（如果不存在就创建新节点）。

**首次创建**
```java
//相关属性
// 默认为 0 
// 当初始化时, 为 -1 
// 当扩容时, 为 -(1 + 扩容线程数) 
// 当初始化或扩容完成后，为 下一次的扩容的阈值大小 
private transient volatile int sizeCtl;

```
调用initTable():
通过cas的方式将sizeCtl从-1设置为初始长度。成功cas的创建table，其他没成功的重新进入循环，直到sizeCtl== 0。

**求size**
当put或remove值会改变size。
原理与LongAdder类似，用多个单元存储值，最终求和得结果，提高效率。

**transfer扩容流程**
先创建nextTable（容量为原先两倍）
然后以一个链表一个链表为单位进行搬迁
 - 如果链表头为空，说明已经处理完，将链表头设为forwardingNode，然后进入下一轮循环。
 - 如果链表头为forwardingNode,也说明处理完了，进入下一轮循环
 - 其他情况下则将链表头锁起来，进行搬迁。

**JDK7原理**
有一个segment数组。每个segment对应一把锁。


### 5.2LinkedBlockingQueue
线程安全队列。
==高明之处==
在于用了两把锁（一把锁锁dummy,一把锁锁尾节点）和 dummy 节点 
- 用一把锁，同一时刻，最多只允许有一个线程（生产者或消费者，二选一）执行 
- 用两把锁，同一时刻，可以允许两个线程同时（一个生产者与一个消费者）执行 消费者与消费者线程仍然串行 生产者与生产者线程仍然串行
```java
// 用于 put(阻塞) offer(非阻塞) 
private final ReentrantLock putLock = new ReentrantLock(); 
// 用户 take(阻塞) poll(非阻塞) 
private final ReentrantLock takeLock = new ReentrantLock();
```

### 5.3ConcurrentLinkedQueue
ConcurrentLinkedQueue 的设计与 LinkedBlockingQueue 非常像，
- 也是 两把【锁】，同一时刻，可以允许两个线程同时（一个生产者与一个消费者）执行
- dummy 节点的引入让两把【锁】将来锁住的是不同对象，避免竞争 
- 只是这【锁】使用了 cas 来实现

### 5.4CopyOnWriteArrayList
线程安全的列表，在对列表进行修改操作时，会创建一个新的底层数组，将修改操作应用到新数组上，而读操作仍然可以在旧数组上进行，从而实现了读写分离，提高了并发读的性能，适用于读多写少的场景。