# 简单说一下并发

## 进程和线程

- 资源单位：进程是资源分配的基本单位，线程是资源调度的基本单位。
- 资源所属：线程依赖于进程，线程只有运行资源（程序计数器、寄存器、栈），系统资源（内存、IO、cpu）共享所属的进程。
- cpu切换：进程切换涉及系统资源，线程切换只涉及运行资源。
- 通信方式：进程通信依赖进程通信，同一进程下的线程共享全局变量等数据。
- 崩溃后果：线程崩溃导致进程崩溃，进程崩溃不影响其他进程。

## 进程通信

- 无名管道（pipe）：半双工的通信方式，只能有亲缘关系的进程通信。
  - 优点：简单方便。
  - 缺点：只能父子进程通信、缓冲区有限。
- 有名管道（FIFO）：半双工的通信方式，允许无亲缘关系的进程通信。
  - 优点：解决了只能父子进程通信的问题。
  - 缺点：缓冲区有限。
- 信号（Signal）：一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。例如kill -9。
  - 优点：解决了缓冲区有限的问题。
  - 缺点：传递信息少。
- 消息队列（Message Queue）：是消息的链表，存放在内核中并由消息队列标识符标识。
  - 优点：解决了传递信息少的问题。
  - 缺点：信息复制耗时。
- 信号量（Semaphore）：一个计数器，用来控制多个进程对共享资源的访问。
  - 优点：进程、线程的同步手段。
  - 缺点：信号量有限。
- 共享内存（Shared Memory）：映射一段能被其他进程访问的内存。
  - 优点：解决了信息复制耗时、信号量有限的问题。
  - 缺点：单一主机内共享。
- 套接字（Socket）：可用于不同主机间的进程通信。
  - 优点：解决了单一主机内共享的问题。
  - 缺点：不可靠通信、网络异常。
- UNIX域套接字（UNIX Domain Socket）：同一主机使用文件进行进程间通信。Docker、MySQL均用到。
  - 优点：可靠通信，容易扩展为套接字模式。
  - 缺点：速度比共享内存慢。

## 线程安全的机制

共享内存是实现线程协作的基础，共享内存有两个问题：竞态条件、内存可见性。

- synchronized：隐式锁，不尝试获取锁，不响应中断。
- 显式锁：Lock接口，ReentrantLock实现类，显式锁，尝试获取锁（可限时），响应中断。
- volatile：解决内存可见性和禁止指令重排优化，set操作不能依赖于get操作。
- 原子变量：基础是CAS，适用于计数。
- 写时复制：共享对象是只读的，写的时候使用锁新创建一个对象。
- ThreadLocal：每个线程对同一个变量都有自己的副本，可避免方法传参。

### java内存模型

规定了一个线程所做的变化何时以及如何变成对其他线程可见。

- 原子性：原子性变量操作有read、load、use、assign、store、write、lock、unlock。
- 可见性：当一个线程修改了共享变量的值，其他线程能够立即得知这个修改。依赖主内存作为传递媒介的方式来实现。
- 有序性：如果在本线程内观察，所有操作都是有序的；如果在一个线程中观察另一个线程，所有操作都是无序的。

允许虚拟机将没有被volatile修饰的64位数据读写操作划分为2次32位的操作来进行，即允许虚拟机实现选择可以不保证64位数据类型的read、load、store、write这4个操作的原子性。

jvm提供了更高层次的字节码指令monitorenter和monitorexit来隐式的使用lock、unlock，这两个字节码也正是synchronized关键字的实现。

- 活性失败：多线性并发时，如果A线程修改了共享变量，此时B线程感知不到此共享变量的变化。
- 快速失败：对集合进行迭代时如果有其他线程对集合进行添加删除操作，迭代会快速报错。
- 安全失败：对集合迭代时对原集合进行一份拷贝，对拷贝的新元素进行迭代，修改操作下次迭代生效。

## 线程的协作机制

- wait/notify：与synchronized配合使用，每个对象都有一把锁、一个锁等待队列（获取不到锁放入等待队列）、一个条件等待队列（调用wait等待notify的条件通知）。
- 显式条件：与显式锁配合使用，await/signal，支持多个条件队列。
- 线程的中断：取消或关闭线程的一种方式。
- 协作工具类：信号量类Semaphore用于限制对资源的并发访问数，倒计时门栓CountDownLatch用于不同角色线程间的同步，循环栅栏CyclicBarrier用于同一角色线程间的同步。
- 阻塞队列：封装了锁和条件。
- Future/FutureTask：异步调用，封装了主线程和执行线程关于执行状态和结果的同步。

### valatile

主存速度跟不上CPU速度，CPU会先把主存数据读到CPU的高速缓存。多个CPU操作会导致缓存不一致的问题，有以下两个方式解决。

- 总线加LOCK#锁：只允许一个CPU使用这个变量的内存。
- 缓存一致性协议：当CPU写操作时发现变量是共享的，发出信号通知其他CPU将该变量的缓存行置为无效。

### Lock

- lock、unlock：阻塞获取锁。
- lockInterruptibly：可响应中断，被其他线程中断，抛出InterruptedException。
- tryLock、tryLock(time, unit)：非阻塞获取锁，可限时，等待期间响应中断。

### ReentrantLock

继承Lock接口，可重入的独占锁，依赖CAS、LockSupport可实现ReentrantLock。

- CAS：比较并交换，乐观锁，自旋等待，aba问题。
- LockSupport：park方法使当前线程放弃CPU，进入等待状态，unpark使指定的线程恢复可运行状态。

### aqs

AbstractQueuedSynchronizer，抽象的队列同步器，为了复用并发工具（ReentrantReadWriteLock、Semaphore、CountDownLatch、CyclicBarrier）的代码。

- 共享资源状态：封装了一个共享资源状态（是否被锁和持有数量），提供查询和设置状态的方法，setState、getState。
- 当前持有线程：用于实现锁时，保存锁的当前持有线程，提供查询和设置线程的方法，setExclusiveOwnerThread、getExclusiveOwnerThread。
- 线程等待队列：维护一个先进先出的线程等待队列，借助CAS实现了无阻塞算法进行更新。

- lock方法：如果未被锁定，则使用CAS进行锁定；如果已被当前线程锁定，则增加锁定次数；如果被其他线程锁定，加入等待队列，被唤醒后检查自己是否是第一个等待的线程，是则锁定。
- FairSync、NonfairSync参数：FairSync整体性能比较低，会让活跃线程得不到锁，进入等待状态，引起频繁的上下文切换。

## 容器工具类

- ReentrantReadWriteLock：读写锁，内部使用同一个整数变量表示锁的状态，16位给读锁，16位给写锁，便于进行CAS操作，只有一个锁等待队列，写锁的获取确保没有其他线程持有，读锁的获取逐个唤醒排队的读锁。
- Semaphore：信号量，acquire/release，限制对资源的并发访问数，不重入。
- CountDownLatch：倒计时门栓，门栓一开始关闭，所有线程等待倒计时变为0，门栓打开后就不能关上了。
- CyclicBarrier：循环栅栏，所有线程到达栅栏后等待，等待一起通过，是循环的，适用于并行迭代计算，每个线程计算一部分，一起计算完后交换数据和计算结果，再进行下一次迭代。

## 线程

### 线程的实现

- 内核线程（1：1）：用户态、内核态来回切换。
- 用户线程（1：N）：自己实现线程调度、处理器映射。
- 混合（M：N）；结合两者优势。

Java 1.2之前的线程为用户线程：屏蔽底层系统的线程差异。

### 线程调度的方式

- 协同式：执行时间线程控制。
- 抢占式：系统分配执行时间。

### 线程的创建方式

- 继承Thread类。
- 实现Runnable接口。
- 通过ExecutorService和Callable实现有返回值的线程。
- 基于线程池。

### 线程的生命周期

- New：新建。
- Runnable：就绪。
- Running：运行。
- Blocked：阻塞。
- Dead：死亡。

### 线程的基本方法

- sleep：当前线程睡眠。
- wait：线程等待。
- notify：线程唤醒。
- yield：线程让步。
- interrupt：线程中断。
- join：线程加入。
- setDeamon：后台守护线程。

### 线程的上下文切换

- 挂起一个进程，将这个进程在CPU中的状态存储于内存的PCB（进程控制块）中。
- 在PCB中检索下一个进程的上下文并将其在CPU的寄存器中恢复。
- 跳转到程序计数器所指向的位置并恢复该进程。

## 线程池

线程复用、线程资源管理、控制并发数。

### 线程池核心组件

- 线程池管理器：创建并管理线程池。
- 工作线程：线程池中执行具体任务的线程。
- 任务接口：定义工作线程的调度和执行策略。
- 任务队列：存放待处理的任务。

```java
public ThreadPoolExecutor(int corePoolSize, int maxinumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maxinumPoolSize, keepAliveTime, unit, workQueue, Executors.defaultThreadFactory(), defalutHandler);
}
```

- corePoolSize：核心线程数量。
- maxinumPoolSize：最大线程数量。
- keepAliveTime：当前线程超过核心线程时，空闲线程的存活时间。
- unit：keepAliveTime的时间单位。
- workQueue：任务队列，被提交但未被执行的任务。
- threadFactory：线程工厂，用于创建线程。
- handler：任务拒绝策略。

### 线程池工作流程

线程池刚被创建时，只向系统申请一个用于执行线程队列和管理线程池的线程资源。调用execute()添加一个任务时，流程如下。

- 运行的线程数少于corePoolSize，线程池立刻创建线程并执行任务。
- 运行的线程数大于等于corePoolSize，任务放入阻塞队列。
- 阻塞队列已满，运行的线程数少于maxinumPoolSize，线程池创建非核心线程并执行任务。
- 阻塞队列已满，运行的线程数大于等于maxinumPoolSize，线程池拒绝执行任务并抛出RejectExecutionException异常。
- 任务执行完毕后，从线程池队列中移除，线程池从阻塞队列中取下一个任务继续执行。
- 线程处于空闲状态（时间超过keepAliveTime），且运行的线程数大于corePoolSize，此任务被认定为空闲线程并停止，保证线程池维持corePoolSize大小。

### 线程池的拒绝策略

- AbortPolicy：直接抛出异常。
- CallerRunsPolicy：被丢弃的任务未关闭，则继续执行。
- DiscardOldestPolicy：移除线程队列中最早的一个线程任务，并尝试提交当前任务。
- DiscardPolicy：丢弃当前任务。
- 自定义拒绝策略：实现RejectedExecutionHandler接口。

### 常用线程池

- newCacheThreadPool：可缓存的线程池。
- newFixedThreadPool：固定大小的线程池。
- newScheduledThreadPool：可做任务调度的线程池。
- newSingleThreadPool：单个线程的线程池。
- newWorkStealingPool：足够大小的线程池。

## 锁

- 重量级锁：基于操作系统的互斥量（Mutex）实现，导致用户态、内核态直接切换。
- 轻量级锁：没有多线程竞争的前提下，提高线程交替执行同步块时的性能。
- 偏向锁：同个锁被同个线程多次获取，提高某个线程交替执行同步块时的性能。
