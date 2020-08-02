# 简单说一下并发

## 进程和线程

- 资源单位：进程是资源分配的基本单位，线程是资源调度的基本单位。
- 资源所属：线程依赖于进程，线程只有运行资源（程序计数器、寄存器、栈），系统资源（内存、IO、cpu）共享所属的进程。
- cpu切换：进程切换涉及系统资源，线程切换只涉及运行资源。
- 通信方式：进程通信依赖进程通信，同一进程下的线程共享全局变量等数据。
- 崩溃后果：线程崩溃导致进程崩溃，进程崩溃不影响其他进程。

### 进程通信

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

### 线程的实现

- 内核线程（1：1）：用户态、内核态来回切换。
- 用户线程（1：N）：自己实现线程调度、处理器映射。
- 混合（M：N）；结合两者优势。

Java 1.2之前的线程为用户线程，为了屏蔽底层系统的线程差异。

### 线程调度的方式

- 协同式：执行时间线程控制。
- 抢占式：系统分配执行时间。

### 线程的上下文切换

- 挂起一个进程，将这个进程在CPU中的状态存储于内存的PCB（进程控制块）中。
- 在PCB中检索下一个进程的上下文并将其在CPU的寄存器中恢复。
- 跳转到程序计数器所指向的位置并恢复该进程。

### 线程的创建方式

- 继承Thread类：Thread类实现了Runnable接口并定义了操作线程的一些方法。重写run方法作为执行逻辑，调用start方法启动线程。
- 实现Runnable接口：创建Thread类的实例并传入实现Runnable接口的类实例，调用start方法启动线程。
- 提交有返回值的线程池：提交有返回值的Callable线程任务。
- 提交无返回值的线程池：提交Runnable线程任务。

### 线程的生命周期

- New：新建，new方法。
- Runnable：就绪，start方法。
- Running：运行，run方法，调用yield进入就绪。
- Blocked：阻塞，sleep、suspend方法，调用resume进入就绪。
- Dead：死亡，stop方法。

### 线程的基本方法

- wait：线程等待，释放对象锁，等待notify唤醒，属于Object类。
- sleep：当前线程睡眠，不释放对象锁，属于Thread类。
- yield：线程让步。
- interrupt：线程中断。
- join：线程加入。
- notify：线程唤醒。
- setDeamon：后台守护线程。

## 线程安全的机制

共享内存是实现线程协作的基础，共享内存有两个问题：竞态条件、内存可见性。

- synchronized：隐式锁，不尝试获取锁，不响应中断。
- 显式锁：Lock接口，ReentrantLock实现类，显式锁，尝试获取锁（可限时），响应中断。
- volatile：解决内存可见性和禁止指令重排优化，set操作不能依赖于get操作。
- 原子变量：基础是CAS，适用于计数。
- 写时复制：共享对象是只读的，写的时候使用锁新创建一个对象。
- ThreadLocal：每个线程对同一个变量都有自己的副本，可避免方法传参。

Synchronized和ReentrantLock他们的开销差距是在释放锁时唤醒线程的数量，Synchronized是唤醒锁池里所有的线程+刚好来访问的线程，而ReentrantLock则是当前线程后进来的第一个线程+刚好来访问的线程。

### java内存模型

为了内存访问逻辑的差异，让一套代码在不同平台下能到达相同的访问结果。规定了一个线程所做的变化何时以及如何变成对其他线程可见。

- 原子性：原子性变量操作有read、load、use、assign、store、write、lock、unlock。
- 可见性：当一个线程修改了共享变量的值，其他线程能够立即得知这个修改。依赖主内存作为传递媒介的方式来实现。
- 有序性：如果在本线程内观察，所有操作都是有序的；如果在一个线程中观察另一个线程，所有操作都是无序的。

允许虚拟机将没有被volatile修饰的64位数据读写操作划分为2次32位的操作来进行，即允许虚拟机实现选择可以不保证64位数据类型的read、load、store、write这4个操作的原子性。

jvm提供了更高层次的字节码指令monitorenter和monitorexit来隐式的使用lock、unlock，这两个字节码也正是synchronized关键字的实现。

- 活性失败：多线性并发时，如果A线程修改了共享变量，此时B线程感知不到此共享变量的变化。
- 快速失败：对集合进行迭代时如果有其他线程对集合进行添加删除操作，迭代会快速报错。
- 安全失败：对集合迭代时对原集合进行一份拷贝，对拷贝的新元素进行迭代，修改操作下次迭代生效。

Happens-Before规则：

- 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
- 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
- volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
- 传递性规则：如果A happens-before B，且B happens-before C，那么A happens-before C。
- start规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
- join规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。
- interrupt规则：对线程interrupt方法的调用happens-before于被中断线程的代码检测到中断事件的发生。

## 线程的协作机制

- wait/notify：与synchronized配合使用，每个对象都有一把锁、一个锁等待队列（获取不到锁放入等待队列）、一个条件等待队列（调用wait等待notify的条件通知）。
- 显式条件：与显式锁配合使用，await/signal，支持多个条件队列。
- 线程的中断：取消或关闭线程的一种方式。
- 协作工具类：
  - ReentrantReadWriteLock：读写锁，内部使用同一个整数变量表示锁的状态，16位给读锁，16位给写锁，便于进行CAS操作，只有一个锁等待队列，写锁的获取确保没有其他线程持有，读锁的获取逐个唤醒排队的读锁。
  - Semaphore：信号量，acquire/release，限制对资源的并发访问数，不重入。
  - CountDownLatch：倒计时门栓，门栓一开始关闭，所有线程等待倒计时变为0，门栓打开后就不能关上了。
  - CyclicBarrier：循环栅栏，所有线程到达栅栏后等待，等待一起通过，是循环的，适用于并行迭代计算，每个线程计算一部分，一起计算完后交换数据和计算结果，再进行下一次迭代。
- 阻塞队列：封装了显式锁和两个条件。
- Future/FutureTask：异步调用，封装了主线程和执行线程关于执行状态和结果的同步。

### synchronized

内部区域的数据：

- ContentionList：锁竞争队列，所有请求锁的线程都被放在这里。
- EntryList：竞争候选列表，在ContentionList中有资格成为候选者来竞争锁资源的线程被移动到这里。
- WaitSet：等待集合，调用wait方法后被阻塞的线程被放在这里。
- OnDeck：竞争候选者，在同一时刻最多只有一个线程在竞争锁资源，该线程的状态为OnDeck。
- Owner：竞争到锁资源的线程被称为Owner状态线程。
- !Owner：在Owner线程释放锁后，会从Owner的状态变为!Owner。

新的线程会尝试自旋获取锁，如果获取不到进入ContentionList。Owner线程释放锁资源时将ContentionList中的部分线程移动到EntryList中，并指定EntryList中的某个线程为OnDeck线程。获取到锁资源的OnDeck线程会变为Owner线程。Owner线程在被wait方法阻塞后，会被转移到WaitSet中，直到被notify方法或notifyAll方法唤醒，会再次进入EntryList中。Owner线程在执行完毕后释放锁资源并变为!Owner状态。

锁会从偏向锁到轻量级锁再到重量级锁。

### valatile

主存速度跟不上CPU速度，CPU会先把主存数据读到CPU的高速缓存。多个CPU操作会导致缓存不一致的问题，有以下两个方式解决。

- 总线加LOCK#锁：只允许一个CPU使用这个变量的内存。
- 缓存一致性协议：当CPU写操作时发现变量是共享的，发出信号通知其他CPU将该变量的缓存行置为无效。

### ThreadLocal

在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本，key为当前ThreadLocal变量（每个线程中可有多个threadLocal变量），value为变量副本。

THreadLocalMap中的Entry的key使用的是ThreadLocal对象的弱引用，在没有其他地方对ThreadLocal依赖，ThreadLocalMap中的ThreadLocal对象在gc时会被回收掉，但是对应的value不会被回收，这个时候Map中就可能存在key为null但是value不为null的Entry，使用完毕需要及时调用remove方法避免内存泄漏。

### Lock

- lock、unlock：阻塞获取锁。
- lockInterruptibly：可响应中断，被其他线程中断，抛出InterruptedException。
- tryLock、tryLock(time, unit)：非阻塞获取锁，可限时，等待期间响应中断。

### ReentrantLock

继承Lock接口，可重入的独占锁，依赖CAS、LockSupport实现ReentrantLock。

- CAS：比较并交换，乐观锁，自旋等待，aba问题。
- LockSupport：park方法使当前线程放弃CPU，进入等待状态，unpark使指定的线程恢复可运行状态。

### AQS

AbstractQueuedSynchronizer，抽象的队列同步器，为了复用并发工具（ReentrantReadWriteLock、Semaphore、CountDownLatch、CyclicBarrier）的代码。

- 共享资源状态：封装了一个共享资源状态（是否被锁和持有数量），提供查询和设置状态的方法，setState、getState。
- 当前持有线程：用于实现锁时，保存锁的当前持有线程，提供查询和设置线程的方法，setExclusiveOwnerThread、getExclusiveOwnerThread。
- 线程等待队列：维护一个先进先出的线程等待队列，借助CAS实现了无阻塞算法进行更新。

- lock方法：如果未被锁定，则使用CAS进行锁定；如果已被当前线程锁定，则增加锁定次数；如果被其他线程锁定，加入等待队列，被唤醒后检查自己是否是第一个等待的线程，是则锁定。
- FairSync、NonfairSync参数：FairSync整体性能比较低，会让活跃线程得不到锁，进入等待状态，引起频繁的上下文切换。

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

线程池刚被创建时，只向系统申请一个用于执行线程队列和管理线程池的线程资源。调用execute()添加一个任务时，流程如下：

- 运行的线程数少于corePoolSize，线程池立刻创建线程并执行任务。
- 运行的线程数大于等于corePoolSize，任务放入阻塞队列。
- 阻塞队列已满，运行的线程数少于maxinumPoolSize，线程池创建非核心线程并执行任务。
- 阻塞队列已满，运行的线程数大于等于maxinumPoolSize，线程池拒绝执行任务并抛出RejectedExecutionException异常。
- 任务执行完毕后，从线程池队列中移除，线程池从阻塞队列中取下一个任务继续执行。
- 线程处于空闲状态（时间超过keepAliveTime），且运行的线程数大于corePoolSize，此任务被认定为空闲线程并停止，保证线程池维持corePoolSize大小。

### 线程池的拒绝策略

- AbortPolicy：直接抛出异常。
- CallerRunsPolicy：被丢弃的任务未关闭，则继续执行。
- DiscardOldestPolicy：移除线程队列中最早的一个线程任务，并尝试提交当前任务。
- DiscardPolicy：丢弃当前任务。
- 自定义拒绝策略：实现RejectedExecutionHandler接口。

### 常用线程池

- newCachedThreadPool：可缓存的线程池，可灵活回收空闲线程（60s），若无可回收，则新建线程。
- newFixedThreadPool：固定大小的线程池。
- newSingleThreadExecutor：单个线程的线程池。
- newScheduledThreadPool：可做任务调度的线程池。
- newWorkStealingPool：足够大小的线程池。

## 锁

- 从乐观和悲观的角度可分为乐观锁和悲观锁。
  - 乐观锁：每次读取数据时认为别人不会修改数据，所以不上锁，但在更新时会判断在此期间别人有没有更新该数据，采用版本号比较和交换。Java通过CAS操作实现。
  - 悲观锁：每次读取数据时认为别人会修改数据，所以读写时上锁，别人想读写时会阻塞等待锁。Java通过AQS操作实现。
- 从获取资源的公平性角度可分为公平锁和非公平锁。
  - 公平锁：排队，先到先得原则。
  - 非公平锁：随机、就近原则。
- 从是否共享资源的角度可分为共享锁和独占锁。
  - 独占锁：也叫互斥锁，每次只允许一个线程持有该锁。
  - 共享锁：允许多个线程同时获取该锁，并发访问共享资源。
- 从锁的状态的角度可分为偏向锁、轻量级锁（JDK6引入）和重量级锁。
  - 偏向锁：同个锁被同个线程多次获取，提高某个线程交替执行同步块时的性能。Mark Word存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单比较ThreadID。特点：只有等到线程竞争出现才释放偏向锁，持有偏向锁的线程不会主动释放偏向锁。之后的线程竞争偏向锁，会先检查持有偏向锁的线程是否存活，如果不存活，则对象变为无锁状态，重新偏向；如果仍存活，则偏向锁升级为轻量级锁，此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程会进入自旋等待获得该轻量级锁。
  - 轻量级锁：没有多线程竞争的前提下，提高线程交替执行同步块时的性能。在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，尝试拷贝锁对象目前的Mark Word到栈帧的Lock Record，若拷贝成功：虚拟机将使用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，并将Lock record里的owner指针指向对象的Mark Word。若拷贝失败：若当前只有一个等待线程，则可通过自旋稍微等待一下，可能持有轻量级锁的线程很快就会释放锁。但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁膨胀为重量级锁。
  - 重量级锁：基于操作系统的互斥量（Mutex）实现，导致用户态、内核态直接切换。
- JVM还设计了自旋锁以更快地使用CPU资源。
  - 如果持有锁的线程能在很短的时间内释放锁资源，那么等待竞争锁的线程只需等一等，就可避免用户态、内核态的切换。

### 产生死锁有四个必要条件

- 互斥：一次只有一个进程可以使用一个资源。其他进程不能访问已分配给其他进程的资源。
- 占有且等待：当一个进程在等待分配得到其他资源时，其继续占有已分配得到的资源。
- 非抢占：不能强行抢占进程中已占有的资源。
- 循环等待：存在一个封闭的进程链，使得每个资源至少占有此链中下一个进程所需要的一个资源。
一个CPU使用这个变量的内存。
- 缓存一致性协议：当CPU写操作时发现变量是共享的，发出信号通知其他CPU将该变量的缓存行置为无效。

### ThreadLocal

在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，key为当前ThreadLocal变量（每个线程中可有多个threadLocal变量），value为变量副本。

THreadLocalMap中的Entry的key使用的是ThreadLocal对象的弱引用，在没有其他地方对ThreadLocal依赖，ThreadLocalMap中的ThreadLocal对象就会被回收掉，但是对应的value不会被回收，这个时候Map中就可能存在key为null但是value不为null的项，这需要实际的时候使用完毕及时调用remove方法避免内存泄漏。

### Lock

- lock、unlock：阻塞获取锁。
- lockInterruptibly：可响应中断，被其他线程中断，抛出InterruptedException。
- tryLock、tryLock(time, unit)：非阻塞获取锁，可限时，等待期间响应中断。

### ReentrantLock

继承Lock接口，可重入的独占锁，依赖CAS、LockSupport可实现ReentrantLock。

- CAS：比较并交换，乐观锁，自旋等待，aba问题。
- LockSupport：park方法使当前线程放弃CPU，进入等待状态，unpark使指定的线程恢复可运行状态。

### AQS

AbstractQueuedSynchronizer，抽象的队列同步器，为了复用并发工具（ReentrantReadWriteLock、Semaphore、CountDownLatch、CyclicBarrier）的代码。

- 共享资源状态：封装了一个共享资源状态（是否被锁和持有数量），提供查询和设置状态的方法，setState、getState。
- 当前持有线程：用于实现锁时，保存锁的当前持有线程，提供查询和设置线程的方法，setExclusiveOwnerThread、getExclusiveOwnerThread。
- 线程等待队列：维护一个先进先出的线程等待队列，借助CAS实现了无阻塞算法进行更新。

- lock方法：如果未被锁定，则使用CAS进行锁定；如果已被当前线程锁定，则增加锁定次数；如果被其他线程锁定，加入等待队列，被唤醒后检查自己是否是第一个等待的线程，是则锁定。
- FairSync、NonfairSync参数：FairSync整体性能比较低，会让活跃线程得不到锁，进入等待状态，引起频繁的上下文切换。

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

线程池刚被创建时，只向系统申请一个用于执行线程队列和管理线程池的线程资源。调用execute()添加一个任务时，流程如下：

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

- 从乐观和悲观的角度可分为乐观锁和悲观锁。
  - 乐观锁：每次读取数据时认为别人不会修改数据，所以不上锁，但在更新时会判断在此期间别人有没有更新该数据，采用版本号比较和交换。Java通过CAS操作实现。
  - 悲观锁：每次读取数据时认为别人会修改数据，所以读写时上锁，别人想读写时会阻塞等待锁。Java通过AQS操作实现。
- 从获取资源的公平性角度可分为公平锁和非公平锁。
  - 公平锁：排队，先到先得原则。
  - 非公平锁：随机、就近原则。
- 从是否共享资源的角度可分为共享锁和独占锁。
  - 独占锁：也叫互斥锁，每次只允许一个线程持有该锁。
  - 共享锁：允许多个线程同时获取该锁，并发访问共享资源。
- 从锁的状态的角度可分为偏向锁、轻量级锁（JDK6引入）和重量级锁。
  - 偏向锁：同个锁被同个线程多次获取，提高某个线程交替执行同步块时的性能。Mark Word存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单比较ThreadID。特点：只有等到线程竞争出现才释放偏向锁，持有偏向锁的线程不会主动释放偏向锁。之后的线程竞争偏向锁，会先检查持有偏向锁的线程是否存活，如果不存活，则对象变为无锁状态，重新偏向；如果仍存活，则偏向锁升级为轻量级锁，此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程会进入自旋等待获得该轻量级锁。
  - 轻量级锁：没有多线程竞争的前提下，提高线程交替执行同步块时的性能。在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，尝试拷贝锁对象目前的Mark Word到栈帧的Lock Record，若拷贝成功：虚拟机将使用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，并将Lock record里的owner指针指向对象的Mark Word。若拷贝失败：若当前只有一个等待线程，则可通过自旋稍微等待一下，可能持有轻量级锁的线程很快就会释放锁。但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁膨胀为重量级锁。
  - 重量级锁：基于操作系统的互斥量（Mutex）实现，导致用户态、内核态直接切换。
- JVM还设计了自旋锁以更快地使用CPU资源。
  - 如果持有锁的线程能在很短的时间内释放锁资源，那么等待竞争锁的线程只需等一等，就可避免用户态、内核态的切换。
