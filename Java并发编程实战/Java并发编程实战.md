# Java并发编程实战

# 线程安全性
Java中主要同步机制是关键字synchronized，但同步这个术语还包括volatile类型的变量，显式锁以及原子变量。

## 什么是线程安全

* 无状态对象一定是线程安全的

## 原子性

### 竞态条件


应尽可能的使用现有的线程安全对象,如AtomicLong来管理类的状态.

## 加锁机制

### 内置锁
Java提供一种内置锁机制来支持原子性:同步代码块.

同步代码块包括两部分:一个作为锁的对象引用,一个作为由这个锁保护的代码块.

synchronized

以synchronized来修饰的方法就是一种横跨整个方法体的同步代码块,其中该同步代码块的锁就是方法调用所在的对象.

静态synchronized方法以Class对象作为锁.

### 重入
内置锁是可重入的,如果某个线程试图获得一个已由他自己持有的锁,这个请求就会成功.

重入意味着获取锁的操作粒度是线程,而不是调用.

## 用锁来保护状态

一种常见加锁约定:将所有可变状态都封装在对象内部,通过对象的内置锁对所有访问可变状态的代码路径进行同步,是的在该对象上不会发生并发访问.Vector和其他同步集合类都使用了这种模式.

## 活跃性与性能

# 对象的共享
## 可见性
### 失效数据

### 非原子的64位操作

### 加锁与可见性

### volatile变量
确保将变量的更新操作通知到其他线程.

volatile变量是一种比sychronized关键字更轻量级的同步机制.

volatile变量通常用做某个操作完成,发生中断或者状态的标志.

加锁机制既可以确保可见性又可以确保原子性,而volatile变量只能确保可见性.

当且仅当满足一下所有条件时,才应该使用volatile变量:

* 对变量的写入操作不依赖变量的当前值,或者你能确保只有单个线程更新变量的值.
* 该变量不会与其他状态变量一起纳入不变性条件中
* 在访问变量时不需要加锁

## 发布与逸出

## 线程封闭

### Ad-hoc线程封闭
是指维护线程封闭性的职责完全由程序实现来承担.

### 栈封闭
是线程封闭的一个特例,在栈封闭中,只能通过局部变量才能访问对象.

### ThreadLocal类
通常用于防止对可变的单实例变量或全局变量进行共享.

## 不变性
不可变对象一定是线程安全的.

满足以下条件时,对象才是不可变的:

* 对象创建后其状态就不能修改
* 对象所有域都是final类型
* 对象是正确创建的,在对象创建期间,this引用没有逸出

### final域
final类型的域是不能修改的

## 安全发布

### 不可变对象与初始化安全性

### 安全发布的常用模式

# 对象的组合
## 设计线程安全的类
设计安全类的过程中，需要包含以下三个基本要素：

1. 找出构成对象状态的所有变量。
2. 找出约束状态变量的不变性条件。
3. 建立对象状态的并发访问管理策略。

要分析对象的状态，首先从对象的域开始，如果对象中所有的域都是基本类型的变量，那么这些域将构成对象的全部状态。

如果在对象的域中引用了其他对象，那么该对象的状态将包含被引用对象的域。

### 收集同步需求
要确保类的线程安全性，就需要确保它的不变性条件不会在并发访问的情况下被破坏。

## 实例封闭
### Java监视器模式
遵循Java监视器模式的对象会把对象的所有可变状态都封装起来，并由对象自己的内置锁来保护。

# 基础构建模块
## 同步容器类
同步容器类包括Vector和HashTable。这些类实现线程安全的方式是：将他们的状态封装起来，并对每个共有方法都进行同步，是的每次只有一个线程能访问容器状态。

### 同步容器类的问题
同步容器类都是线程安全的，但在某些情况下可能需要额外的客户端加锁来保护复合操作。

容器常见的复合操作包括：迭代，跳转，以及条件运算。

### 迭代器与ConcurrentModificationException

### 隐藏迭代器
加锁可以防止迭代器抛出ConcurrentModificationException。

容器的hashCode和equals等方法会间接的执行迭代操作，当容器作为另一个容器的元素或键值时也会间接的进行迭代操作。containsAll，removeAll，retainAll等。都可能抛出ConcurrentModificationException。

## 并发容器
Jdk1.5提供了多种并发容器来改进同步容器的性能。同步容器将所有对容器的访问都串行化，以实现他们的线程安全性。代价是严重降低并发性，吞吐量严重降低。

ConcurrentHashMap代替同步且散列的Map。CopyOnWriteArrayList用于在遍历操作作为主要操作的情况下代替同步List。

新的ConcurrentMap接口中增加了对一些常见复合操作的支持，如：若没有则添加，替换以及有条件删除等。

通过并发容器代替同步容器，可以极大的提高伸缩性并降低风险。

Jdk1.5新增了两种类型的容器：Queue和BlockingQueue。

Queue用来临时保存一组等待处理的元素。实现：ConcurrentLinkedQueue传统的先进先出队列，PriorityQueue，非并发的优先队列。Queue上的操作不会阻塞，如果队列为空，那么获取元素的操作会返回空值。

BlockingQueue扩展了Queue，增加了可阻塞的插入和获取操作。如果队列为空，获取元素的操作将一直阻塞，直到队列中出现一个可用的元素。如果队列已满，那么插入元素的操作将一直阻塞，直到队列出现一个可用空间。

Jdk1.6引入了ConcurrentSkipListMap和ConcurrentSkipListSet，分别作为同步的SortedMap和SortedSet的并发替代品。

### ConcurrentHashMap
同步容器类在执行每个操作期间都持有一个锁，其他线程在这段时间内不能访问该容器。

ConcurrentHashMap也是基于散列的Map，但是使用了一种完全不同的加锁策略来提供更高的并发性和伸缩性。

ConcurrentHashMap使用分段锁，任意数量的读取线程可以并发的访问Map，执行读取操作的线程和执行写入操作的线程可以并发的访问Map，并且一定数量的写入线程可以并发的修改Map。并发访问环境下实现更高的吞吐量。

提供的迭代器不会抛出ConcurrentModificationException。迭代过程不需要加锁。ConcurrentHashMap的迭代器具有弱一致性，可以容忍并发的修改，当创建迭代器时，会遍历已有元素，并可以在迭代器被构造后将修改反映给容器。

对于size和isEmpty方法返回的不是准确值，而是估计值。

### 额外的原子Map操作
复合操作，如：若没有则添加，若相等则移除，若相等则替换等，都已经实现为原子操作，并在ConcurrentMap接口中声明。

### CopyOnWriteArrayList
用于代替同步List，某些情况下提供了更好的并发性，迭代期间不需要对容器进行加锁或复制。

写入时复制，每次修改时都会创建并重新发布一个新的容器副本，从而实现可变性。

容器的迭代器保留一个指向底层基础数组的引用。多个线程可以同时对这个容器进行迭代。不会抛出ConcurrentModificationException，返回的元素与迭代器创建时的元素完全一致，不必考虑之后修改操作带来的影响。

每次修改都会复制底层数组。仅当迭代操作远多于修改操作时，采用。比如多事件通知系统。

## 阻塞队列和生产者-消费者模式
阻塞队列提供了可阻塞的put和take方法，以及支持定时的offer和poll方法。

如果队列满了，put方法将阻塞直到有空间可用，如果队列为空，take方法将会阻塞直到有元素可用。

队列可以是有界也可以是无界的，无界队列永远不会充满，所以无界队列上的put方法永远不会阻塞。

阻塞队列支持生产者-消费者这种设计模式。

offer方法，如果不能添加到队列，返回一个失败状态。

BlockingQueue的实现：LinkedBlockingQueue，ArrayBlockingQueue是FIFO队列。

PriorityBlockingQueue是一个按优先级排序的队列。

SynhronousQueue，不会为队列中元素维护存储空间。没有存储功能，put和take操作会一直阻塞，直到有另一个线程已经准备好参与到交付过程中。仅当有足够多的消费者，并且总是有一个消费者准备好获取交付的工作时，才适合用同步队列。

### 串行线程封闭
### 双端队列与工作密取
Jdk1.6增加了Deque和BlockingDeque，分别对Queue和BlockingQueue进行了扩展。

Deque双端队列，实现了在列头和列尾的高效插入和移除。具体实现：ArrayDeque和LinkedBlockingDeque。

阻塞队列适用于生产者消费者模式，所有消费者有一个共享的工作队列。

双端对列适用于工作密取，每个消费者都有各自的双端队列。一个消费者完成了自己双端队列中的任务，那可以从其他消费者双端队列末尾秘密的获取工作。

## 阻塞方法与中断方法
线程可能会阻塞或暂停执行，原因：等待I/O操作结束，等待获取一个锁，等待从Thread.sleep方法中醒来，或是等待另一个线程计算结果。

线程阻塞，通常会被挂起，处于某种阻塞状态blocked，waiting，timed waiting。

BlockingQueue的put和take等会抛InterruptedException，这表示是一个阻塞方法，如果方法被中断，它努力提前结束阻塞状态。

## 同步工具类
阻塞队列，信号量Semaphore，栅栏Barrier，闭锁Latch等。

### 闭锁(Latch)
闭锁的作用相当于一扇门:在闭锁到达结束状态之前,这扇门一直是关闭的,并且没有任何线程能通过,当到达结束状态时,这扇门会打开并允许所有的线程通过.当闭锁达到结束状态后,将不会再改变状态,因此这扇门将永远保持打开状态。闭锁可以用来确保某些活动直到其他活动都完成后才继续执行。

例如：

* 确保某个计算在其需要的所有资源都被初始化之后，才继续执行。
* 确保某个服务在其依赖的所有其他服务都已经启动之后才启动。
* 等待直到某个操作的所有参与者就绪再继续执行。

CountDownLatch，可以使一个或者多个线程等待一组事件发生。闭锁状态包括一个计数器，计数器初始化为一个正数，表示需要等待的事件数量。countDown方法递减计数器，表示有一个事件发生了，await方法等待计数器到达0，表示所有的需要等待的事件都已经发生。如果计数器非零，await会一直阻塞直到计数器为0，或者等待中线程中断，或者等待超时。

### FutureTask
表示一种抽象的可生成结果的计算，表示的计算是通过Callable来实现的。

Future.get，如果任务完成，立即返回结果，否则get将阻塞直到任务完成，然后返回结果或跑出异常。
### 信号量
用来控制同时访问某个特定资源的操作数量或者同时执行某个指定操作的数量。

Semaphore中管理着一组虚拟的许可，执行操作时可以首先获得许可，在使用后释放。如果没有许可，acquire将阻塞直到有许可。

release方法将返回一个许可给信号量。

### 栅栏Barrier
所有线程必须同时到达栅栏位置才能继续执行。

闭锁用于等待事件，栅栏用于等待其他线程。

CyclicBarrier可使一定数量的参与方反复的在栅栏位置汇集。

Exchanger是一个两方栅栏。

# 任务执行
## 在线程中执行任务

## Executor框架
该框架能支持多种不同类型的任务执行策略.

提供了标准的方法将任务的提交过程与执行过程解耦开来,用Runnable来表示任务。Executor的实现还提供了对生命周期的支持，以及统计信息收集，应用程序管理机制和性能监视等机制。

Executor基于生产者消费者模式，提交任务的操作相当于生产者，执行任务的线程相当于消费者。

### 线程池
Executors中的静态工厂方法来创建线程池：

newFixedThreadPool 将创建一个固定长度的线程池,每当提交一个任务时就创建一个线程,直到达到线程池的最大数量.

newCachedThreadPool 创建一个可缓存的线程池,如果线程池当前规模超过了处理需求时,将回收空闲空间的线程,当需求增加时,则可以添加新的线程,线程池的规模不存在任何限制.

newSingleThreadExecutor 是一个单线程的Executor,它创建单个工作者线程来执行任务.

newScheduledThreadPool 创建一个固定长度的线程池,而且以延迟或定时的方式来执行任务,类似于Timer.

## 找出可利用的并行性
要使用Executor，必须将任务表述成一个Runnable。

可以通过多种方法创建一个Future来描述任务，ExecutorService中的所有submit方法都将返回一个Future，从而将一个Runnable或Callable提交给Executor，并得到一个Future用来获得任务的执行结果或取消任务。还可以显式的为某个指定的Runnable或Callable实例化一个FutureTask。由于FutureTask实现了Runnable，因此可以将它提交给Executor来执行，或者直接调用它的run方法。

### 在异构任务并行化中存在的局限
只有当大量相互独立且同构的任务可以并发进行处理时，才能体现出将程序的工作负载分配到多个任务中带来的真正性能提升。

### Executor的生命周期

ExecutorService添加了一些用于生命周期管理的方法

生命周期有3种状态:运行,关闭,已终止.

shutdown方法执行平缓的关闭过程，不再接受新任务，同时等待已经提交的任务执行完成，包括还未开始执行的任务。

shutdownNow执行粗暴的关闭，尝试取消所有运行中的任务，不再启动队列中尚未开始执行的任务。

通过调用awaitTermination来等待ExecutorService到达终止状态，或通过isTerminated来轮询ExecutorService是否已经终止。

### 延迟任务与周期任务

### 携带结果的任务Callable与Future
Executor框架使用Runnable作为其基本的任务表示形式，但有很大局限性，不能返回一个值或抛出一个受检查的异常。

Executor执行的任务有4个生命周期：创建，提交，开始，完成。

已提交但尚未开始的任务可以取消，但对于已经开始的任务，只有响应中断时才能取消，取消已经完成的任务不会有任何影响。

Future表示一个任务的生命周期，提供相应的方法来判断是否已经完成或取消，以及获取任务的结果和取消任务等。

get方法的行为取决于任务的状态。如果任务已经完成，get就会立即返回或者抛出一个Exception，如果任务没有完成，那么get将阻塞并指导任务完成。如果任务抛出了异常，那么get将该异常封装为ExecutionException并重新抛出。

ExecutorService中的所有submit方法都将返回一个Future，从而将一个Runnable或Callable提交给ExecutorService，并得到一个Future用来获得任务的执行结果或者取消任务。还可以显式的为某个Runnable或Callable实例化一个FutureTask。

### CompletionService，Executor与BlockingQueue
CompletionService将Executor和BlockingQueue功能融合到一起。你可以将Callable任务任务提交给它来执行，然后使用类似于队列操作的take和pull等方法来获取已完成的结果，结果会在完成时被封装成Future。ExecutorCompletionService实现了CompletionService，并将计算部分委托给一个Executor。

Future.get

# 取消与关闭
## 任务取消
### 中断
每个线程都有一个boolean类型的中断状态，当中断线程时，这个线程的中断状态将被设置为true。

在Thread中包含了中断线程以及查询线程中断状态的方法。interrupt方法能中断目标线程，而isInterrupted方法能返回目标线程的中断状态。静态的interrupted方法将清除当前线程的中断状态，并返回它之前的值，这也是清除中断状态的唯一方法。

### 通过Future来实现取消
Future的cancel方法带有一个boolean类型的参数mayInterruptIfRunning表示取消操作是否成功。如果mayInterruptIfRunning为true并且任务当前正在某个线程中运行，那么这个线程能被中断。如果这个参数为false，意味着若任务还没有启动就不要运行它。

### 处理不可中断的阻塞
许多可阻塞的方法都是通过提前返回或者抛出InterruptedException来响应中断请求的。

并非所有的可阻塞方法或者阻塞机制都能响应中断。

获取某个锁 如果一个线程由于等待某个内置锁而阻塞，那么将无法响应中断，因为线程认为它肯定会获得锁，所以将不会处理中断请求。

Lock类中提供了lockInterruptibly方法，该方法允许在等待一个锁的同时仍能响应中断。

### 采用newTaskFor来封装非标准的取消
newTaskFor方法是一个工厂方法，它将创建Future来代表任务。

还能返回一个RunnableFuture接口，该接口扩展了Future和Runnable。

## 停止基于线程的服务

### 关闭ExecutorService
shutdown和shutdownNow方法

# 线程池的使用
### 设置线程池大小
对于计算密集型的任务，拥有N个处理器，当线程池的大小为N+1时能实现最优的利用率。

对于包含I/O操作或其他阻塞操作的任务，由于线程不会一直执行，因此线程池的规模应该更大。

## 配置ThreadPoolExecutor

ThreadPoolExecutor的通用构造函数：

```
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,
								long keepAliveTime, TimeUnit unit,
								BlockingQueue<Runnable> workQueue,
								ThreadFactory threadFactory,RejectedExecutionHandler handler){...}
```

### 线程的创建与销毁
线程池的基本大小corePoolSize，就是线程池的目标大小，即在没有任务执行时，线程池的大小，并且只有在工作队列满了的情况下才会创建出这个数量的线程。

线程池的最大大小maximumPoolSize表示可同时活动的线程数量的上限。如果线程的空闲时间超过了存活时间，那么将被标记为可回收的，并且当线程池的当前大小超过了基本大小时，这个线程将被终止。

newFixedThreadPool工厂方法将线程池的基本大小和最大大小设置为参数中指定的值，而且创建的线程池不会超时。

newCachedThreadPool工厂方法将线程池的最大大小设置为Integer.MAX_VALUE，而将基本大小设置为0，并将超时设置为1分钟，线程池可以被无限扩展，并且当需求降低时会自动收缩。

### 管理队列任务
ThreadPoolExecutor允许提供一个BlockingQueue来保存等待执行的任务，基本的任务排队方法有3种：无界队列，有界队列，和同步移交。

newFixedThreadPool和newSIngleThreadExecutor默认情况下将使用一个无界的LinkedBlockingQueue。如果所有工作者线程都处于忙碌状态，那么任务将在队列中等候。如果任务持续快速的到达，并且超过了线程池处理它们的速度，那么队列将无限制的增加。

有界队列有助于避免资源耗尽的情况发生。

对于非常大的或者无界的线程池，可以通过使用SynchronousQueue来避免任务排队，以及直接将任务从生产者移交给工作者线程。SynchronousQueue不是真正的队列，而是一种在线程之间进行 移交的机制。要将一个元素放入SynchronousQueue中，必须有另一个线程正在等待接受这个元素。

# 显式锁
## Lock与ReentrantLock
Lock提供了一种无条件的，可轮询的，定时的，可中断的锁获取操作，所有加锁和解锁的方法都是显式的。

ReentrantLock实现了Lock接口，提供了与synchronized相同的互斥性和内存可见性。获取ReentrantLock时，有着与进入同步代码块相同的内存语义，在释放ReentrantLock时有着与退出同步代码块相同的内存语义。

### 轮询锁与定时锁
可定时的与可轮询的锁获取模式是由tryLock方法实现的。避免死锁的发生。

### 可中断的锁获取操作
lockInterruptibly方法能够在获得锁的同时保持对中断的响应，由于它包含在Lock中，因此无需创建其他类型的不可中断阻塞机制。

### 非块结构的加锁

## 公平性
ReentrantLock的构造函数提供了两种公平性选择：创建一个非公平的锁（默认），或者一个公平的锁。

## 读写锁
ReadWriteLock，允许多个读操作同时进行，但每次只允许一个写操作。

释放优先，读线程插队，重入性，降级，升级。

ReentrantReadWriteLock为这两种锁都提供了可重入的加锁语义。在构造时可以选择是一个非公平（默认）或者是一个公平的锁。

# 构建自定义的同步工具

## 显式的Condition对象
Condition是一种广义的内置条件队列。

一个Condition和一个Lock关联在一起，要创建一个Condition，可以再相关联的Lock上调用Lock.newCondition方法。

在每个锁可存在多个等待，条件等待可以是可中断的活不可中断的，基于时限的等待，以及公平的活非公平的队列操作。

Condition对象中与wait，notify和notifyAll方法对应的分别是await，singal和singalAll。

## Synchronizer剖析
ReentrantLock和Semaphore两个接口存在许多共同点，可以用作一个阀门，每次只允许一定数量的线程通过，当线程达到阀门时，可以通过（在调用lock或acquire时成功返回），也可以等待（在调用lock或acquire时阻塞），还可以取消（在调用tryLock或tryAcquire时返回false），两个接口都可以支持可中断的，不可中断的以及限时的获取操作，并且也都支持等待线程执行公平或非公平的队列操作。

## AbstrsctQueuedSynchronizer
AQS用于构建锁和同步器的框架。ReentrantLock，Semaphore，CountDownLatch，ReentrantReadWriteLock,SynchronousQueue,FutureTask等都是基于AQS构建的。

## java.util.concurrent同步器类中的AQS
### ReentrantLock
ReentrantLock只支持独占方式的获取操作，因此实现了tryAcquire，tryRelease和isHeldExclusively。

# 原子变量与非阻塞同步机制

### 比较并交换
CAS指令，包含了3个操作数：需要读写的内存位置V，进行比较的值A，拟写入的新值B。当且仅当V的值等于A时，CAS才会通过原子方式用新值B来更新V的值，否则不会执行任何操作。返回V原有的值。


# Java内存模型



