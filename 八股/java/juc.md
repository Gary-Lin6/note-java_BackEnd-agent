
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