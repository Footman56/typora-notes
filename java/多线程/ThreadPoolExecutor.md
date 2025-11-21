# 一、池化思想

提前将一些创建比较复杂的对象创建好，之后使用的时候可以直接拿来用，并且能够管理请求比较多的情况。本质是将多个资源都统一管理起来。

常见的实现有：

1. 内存池(Memory Pooling)：预先申请内存，提升申请内存速度，减少内存碎片。
2. 连接池(Connection Pooling)：预先申请数据库连接，提升申请连接的速度，降低系统的开销。
3. 实例池(Object Pooling)：循环使用对象，减少资源在初始化和释放时的昂贵损耗。

能够解决问题：

1. 避免频繁创建资源和销毁资源。
2. 能够合理有限的使用资源，避免系统资源耗尽

**其实并不是一开启就创建好一些线程，而是在有一定数量之前是来一个任务分配一个资源，之后在重复使用这些资源。也可以先预热线程池**

# 二、总体设计

ThreadPoolExecutor 类图

<img src="/Users/peilizhi/Desktop/类图.png" alt="图1 ThreadPoolExecutor UML类图" style="zoom:0%;" />

ThreadPoolExecutor实现的顶层接口是Executor，顶层接口Executor提供了一种思想：**将任务提交和任务执行进行解耦**。用户无需关注如何创建线程，如何调度线程来执行任务，***用户只需提供Runnable对象，将任务的运行逻辑提交到执行器(Executor)中，由Executor框架完成线程的调配和任务的执行部分***。ExecutorService接口增加了一些能力：（1）扩充执行任务的能力，补充可以为一个或一批异步任务生成Future的方法；（2）提供了管控线程池的方法，比如停止线程池的运行。

AbstractExecutorService则是上层的抽象类，将执行任务的流程串联了起来，保证下层的实现只需关注一个执行任务的方法即可。最下层的实现类ThreadPoolExecutor实现最复杂的运行部分，

ThreadPoolExecutor将会一方面维护自身的生命周期，另一方面同时管理线程和任务，使两者良好的结合从而执行并行任务。

线程池结构：

核心线程池：获取工作队列中的任务

工作队列：存储一些任务

额外线程：单独创建的线程。

本质上核心线程池与额外线程 没什么区别，都是先执行初始化分配的之后，之后依次从工作队列中获取任务。所以corePoolSize=maximumPoolSize

<img src="/Users/peilizhi/Desktop/线程池结构.png" alt="image-20220811101745638" style="zoom:50%;" />

参数：

```java
AtomicInteger ctl: 一个原子整数，整合了线程状态和有效线程数。可通过位运算来分别获取线程状态和有效线程数
BlockingQueue<Runnable> workQueue：工作队列，在核心线程都被分配的时候，将任务加入到工作队列
int corePoolSize: 核心池大小
int maximumPoolSize:最大池大小 。实际上最大容量是由 CAPACITY 控制。默认是
long keepAliveTime:等待线程最大超时数，如果线程超时时间大于最大超时数，可被终止。仅在线程数当存在多于 corePoolSize 或 allowCoreThreadTimeOut = true 时，线程使用此超时。否则，他们将永远等待新的工作。当空闲时间达到 keepAliveTime值时，额外线程会被销毁，直到只剩下 corePoolSize 个线程为止。
ThreadFactory threadFactory：新线程的工厂。所有线程都是使用这个工厂创建的
RejectedExecutionHandler handler:拒绝策略
RejectedExecutionHandler defaultHandler:默认拒绝策略
ReentrantLock mainLock:锁。锁都在资源的一个属性，只有访问者对个资源持有相同内存的锁，才算是占有锁。
HashSet<Worker> workers:包含池中所有工作线程的集合。仅在持有 mainLock 时访问。
```

对线程池的上锁能够保证，线程池执行多个任务的时候能够有完整、原子性的结果。

方法：

任务调度

```java
// 在未来的某个时间执行给定的任务。该任务可以在新线程或现有池线程中执行。
// 如果任务无法提交执行，要么是因为这个执行器已经关闭，要么是因为它的容量已经达到，任务由当前的RejectedExecutionHandler处理。
void execute(Runnable command) ;
  
//检查是否可以根据当前池状态和给定边界（核心或最大值）添加新工作人员。如果是这样，则相应地调整工作人员计数，并且如果可能，创建并启动一个新工作人员，将 firstTask 作为其第一个任务运行。如果池已停止或有资格关闭，则此方法返回 false。如果线程工厂在询问时未能创建线程，它也会返回 false。如果线程创建失败，要么是由于线程工厂返回 null，要么是由于异常（通常是 Thread.start() 中的 OutOfMemoryError），我们干净地回滚。【在有任务的时候添加工作者】
boolean addWorker(Runnable firstTask, boolean core);
```

任务执行

```java
// 在合理的情况下，从工作队列获取任务
Runnable getTask() ;

// 执行任务
void runWorker(Worker w);

// 有序关闭，如果正在执行的就等执行完，之后不在接受任务
void shutdown()；
  
// 停止所有正在执行的任务，停止等待任务的处理，并返回等待执行的任务列表。这些任务在从该方法返回时从任务队列中排出（删除）。  
List<Runnable> shutdownNow()；
```

```java
// 启动一个核心线程，使其空闲等待工作
boolean prestartCoreThread() ;

//// 预启动所有核心线程,使其空闲等待工作
prestartAllCoreThreads(); 
```

线程与任务

对于线程池来说，给次提交的的是任务，将任务分配给固定数量的线程（工人）。

## 2.1 执行流程图

![图2 ThreadPoolExecutor运行流程](/Users/peilizhi/Desktop/执行流程.png)



## 2.2  整体流程

​	![整体流程](/Users/peilizhi/Desktop/ThreadPoolExecutor.png)



## 2.3 调用链

![ThreadPoolExecutor执行链路](/Users/peilizhi/Desktop/流程.png)

# 三、生命周期

RUNNING：接受新任务并处理排队任务 

SHUTDOWN：不接受新任务，但处理排队任务 

STOP：不接受新任务，不处理排队任务，并中断正在进行的任务 

TIDYING：所有任务都已终止，workerCount 为零

TERMINATED：当 terminate() 钩子方法完成时，在 awaitTermination() 中等待的线程将在状态达到 TERMINATED 时返回，线程池彻底终止的状态。

![图3 线程池生命周期](/Users/peilizhi/Desktop/生命周期.png)

# 四、任务调度

## 4.1 提交任务

```java
// 将任务提交过来之后，就不再需要关注执行啦
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
// 获取当前线程池数据和线程状态
        int c = ctl.get();
// 检查有效线程是否小于核心线程数：小于时创建一个工作线程
        if (workerCountOf(c) < corePoolSize) {
          // 分配任务给线程
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
// 如果线程池处于执行状态，并且可以加入工作队列->将任务加入到工作队列中
        if (isRunning(c) && workQueue.offer(command)) {
          // 再次检查线程池数据和线程状态
            int recheck = ctl.get();
          // 线程池不在运行中&&如果该任务存在，则从执行器的工作队列中删除该任务-> 拒绝新任务
            if (! isRunning(recheck) && remove(command))
                reject(command);
          // 只有有效线程被占满时才加入到工作队列
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
// 不能将任务添加到额外队列时，拒绝任务
        else if (!addWorker(command, false))
            reject(command);
}
```

## 4.2 分配任务

尝试给核心线程池和额外线程池分配任务的时候是将任务交给线程的

```java
// Runnable firstTask： 第一个要执行的任务，在提交任务的时候，就将提交过来的任务绑定的工人身上
// ore – 如果为 true，则使用 corePoolSize 作为绑定，否则为 maximumPoolSize。 （此处使用布尔指示符而不是值来确保在检查其他池状态后读取新值）
private boolean addWorker(Runnable firstTask, boolean core) {
        retry:
        for (;;) {
          
            int c = ctl.get();
          // 获取线程池状态
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
          // 不能处理任务的状态，退出，不能创建线程
            if (rs >= SHUTDOWN &&
                ! (rs == SHUTDOWN &&
                   firstTask == null &&
                   ! workQueue.isEmpty()))
                return false;

            for (;;) {
                int wc = workerCountOf(c);
              // 如果任务数大于线程池所能处理的任务数->不能添加任务
                if (wc >= CAPACITY ||
                    wc >= (core ? corePoolSize : maximumPoolSize))
                    return false;
              // CAS的方式让workerCount数量增加1,如果成功, 终止循环
                if (compareAndIncrementWorkerCount(c))
                    break retry;
                c = ctl.get();  // Re-read ctl
               // 再次检查runState, 如果被更改, 重头执行retry代码
                if (runStateOf(c) != rs)
                    continue retry;
                // 其他的, 上面的CAS如果由于workerCount被其他线程改变而失败, 继续内部的for循环
            }
        }

        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
          //
            w = new Worker(firstTask);
            final Thread t = w.thread;
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                try {
                    // Recheck while holding lock.
                    // Back out on ThreadFactory failure or if
                    // shut down before lock acquired.
                    int rs = runStateOf(ctl.get());
                    // 如果是RUNNING状态, 或者是SHUTDOWN状态并且传入的task为null(执行workQueue中的task)
                    if (rs < SHUTDOWN ||
                        (rs == SHUTDOWN && firstTask == null)) {
                      // 线程已经被异动，抛出异常
                        if (t.isAlive()) // precheck that t is startable
                            throw new IllegalThreadStateException();
                      // 将新创建的Worker 加入到整个ThreadPoolExecutor中维护的Worker集合，表示有效的执行线程
                        workers.add(w);
                        int s = workers.size();
                        if (s > largestPoolSize)
                          // 记录最大数量的线程
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
                if (workerAdded) {
                  // 启动线程，执行worker中的run方法
                    t.start();
                  // 线程启动成功
                    workerStarted = true;
                }
            }
        } finally {
          // 如果线程启动失败, 执行addWorkerFailed方法
            if (! workerStarted)
                addWorkerFailed(w);
        }
        return workerStarted;
    }
```



```java
// 启动线程失败的情况
private void addWorkerFailed(Worker w) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (w != null)
              // 从Worker集合 中移动开启失败的线程
                workers.remove(w);
          // 递减 ctl 的 workerCount 字段
            decrementWorkerCount();
          // 尝试终止线程池
            tryTerminate();
        } finally {
            mainLock.unlock();
        }
    }

final void tryTerminate() {
        for (;;) {
            int c = ctl.get();
          // 如果是运行中或者 是SHUTDOWN但是工作队列不为空 或者 在TIDYING 之前的状态（STOP），退出不处理
            if (isRunning(c) ||
                runStateAtLeast(c, TIDYING) ||
                (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
                return;
          // 工作线程不为0
            if (workerCountOf(c) != 0) { // Eligible to terminate
              // 中断可能正在等待任务的线程，且只终止一个
                interruptIdleWorkers(ONLY_ONE);
                return;
            }

            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
              // 如果线程是TIDYING，并且没有运行的任务时
                if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                    try {
                      // 终止
                        terminated();
                    } finally {
                        ctl.set(ctlOf(TERMINATED, 0));
                        termination.signalAll();
                    }
                    return;
                }
            } finally {
                mainLock.unlock();
            }
            // else retry on failed CAS
        }
    }
```

# 五、任务执行

```java
Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
  // 利用ThreadFactory, 传入当前Worker对象(为了执行当前Worker中的run方法), 创建Thread对象.
            this.thread = getThreadFactory().newThread(this);
        }
```

在addWorker 中调用的start() 执行 Runnable#run

## 5.1 执行任务

```java
final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
  			// 将创建线程时设置的初始任务 为做要执行的任务
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
          //当传入的task不为null, 或者task为null但是从BlockingQueue中获取的task不为null，有任务要执行
            while (task != null || (task = getTask()) != null) {
                w.lock();
              // 线程池状态如果为STOP, 或者 当前线程是被中断并且线程池是STOP状态, 然而当前线程没有被中断;
			        // 则调用interrupt方法中断当前线程
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                    wt.interrupt();
                try {
                  // 执行beforeExecute 勾子方法
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                      // 处理任务
                        task.run();
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                      // 执行afterExecute 勾子方法
                        afterExecute(task, thrown);
                    }
                } finally {
                    task = null;
                  // 记录线程完成的任务数
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
          // 当前执行线程都正常执行完（false）或者意外退出时（true）：
            processWorkerExit(w, completedAbruptly);
        }
    }

private void processWorkerExit(Worker w, boolean completedAbruptly) {
        if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
            decrementWorkerCount();

        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
          // 记录完成任务数
            completedTaskCount += w.completedTasks;
          // 移除线程
            workers.remove(w);
        } finally {
            mainLock.unlock();
        }
        // 尝试终止线程池
        tryTerminate();
        int c = ctl.get();
  // 线程池状态是RUNNING，SHUTDOWN 并且是当前线程已经不能获取任务。
        if (runStateLessThan(c, STOP)) {
            if (!completedAbruptly) {
                int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
                if (min == 0 && ! workQueue.isEmpty())
                    min = 1;
                if (workerCountOf(c) >= min)
                    return; // replacement not needed
            }
          // 尝试创建一个额外的线程继续执行任务。
            addWorker(null, false);
        }
    }
```

## 5.2 获取任务

```java
private Runnable getTask() {
        boolean timedOut = false; // Did the last poll() time out?

        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
                decrementWorkerCount();
                return null;
            }

            int wc = workerCountOf(c);

            // Are workers subject to culling?
          // 如果allowCoreThreadTimeOut =true 或者工作线程数大于核心线程数才有时间限制
            boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

            if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
                if (compareAndDecrementWorkerCount(c))
                    return null;
                continue;
            }

            try {
              // 都是从工作队列获取任务
                Runnable r = timed ?
              // 在指定的时间内获取队列头中的任务    
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
                if (r != null)
                    return r;
                timedOut = true;
            } catch (InterruptedException retry) {
                timedOut = false;
            }
        }
    }
```

# 六、工作队列

##  直接提交队列

设置为SynchronousQueue队列，SynchronousQueue是一个特殊的BlockingQueue，**它没有容量**，每执行一个插入操作就会阻塞，需要再执行一个删除操作才会被唤醒，反之每一个删除操作也都要等待对应的插入操作。

```java
 private static ExecutorService pool;
    public static void main( String[] args )
    {
        //maximumPoolSize设置为2 ，拒绝策略为AbortPolic策略，直接抛出异常
        pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new SynchronousQueue<Runnable>(),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
        for(int i=0;i<3;i++) {
          // 执行完第二个任务之后，再执行第三个时候会执行拒绝策略
          pool.execute(new ThreadTask());
        }   
    }
```

## 有界任务队列

有界的任务队列可以使用ArrayBlockingQueue实现

```java
pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<Runnable>(10),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
```



使用ArrayBlockingQueue有界任务队列，若有新的任务需要执行时，线程池会创建新的线程，直到创建的线程数量达到corePoolSize时，则会将新的任务加入到等待队列中。若等待队列已满，即超过ArrayBlockingQueue初始化的容量，则继续创建线程，直到线程数量达到maximumPoolSize设置的最大线程数量，若大于maximumPoolSize，则执行拒绝策略。在这种情况下，线程数量的上限与有界任务队列的状态有直接关系，如果有界队列初始容量较大或者没有达到超负荷的状态，线程数将一直维持在corePoolSize以下，反之当任务队列已满时，则会以maximumPoolSize为最大线程数上限。

## 无界任务队列

无界任务队列可以使用LinkedBlockingQueue实现，

```java
pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());

```

无界任务队列，线程池的任务队列可以无限制的添加新的任务，而**线程池创建的最大线程数量就是你corePoolSize设置的数量**，也就是说在这种情况下maximumPoolSize这个参数是无效的，哪怕你的任务队列中缓存了很多未执行的任务，当线程池的线程数达到corePoolSize后，就不会再增加了；若后续有新的任务加入，则直接进入队列等待，当使用这种任务队列模式时，一定要注意你任务提交与处理之间的协调与控制，不然会出现队列中的任务由于无法及时处理导致一直增长，直到最后资源耗尽的问题。

##  优先任务队列

优先任务队列通过PriorityBlockingQueue实现

```java
public static void main( String[] args )
    {
        //优先任务队列
        pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new PriorityBlockingQueue<Runnable>(),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
          
        for(int i=0;i<20;i++) {
            pool.execute(new ThreadTask(i));
        }    
    }
public class ThreadTask implements Runnable,Comparable<ThreadTask>{
    
    private int priority;
    public int getPriority() {
        return priority;
    }
    public void setPriority(int priority) {
        this.priority = priority;
    }
    
    public ThreadTask(int priority) {
        this.priority = priority;
    }

    //当前对象和其他对象做比较，当前优先级大就返回-1，优先级小就返回1,值越小优先级越高
    public int compareTo(ThreadTask o) {
         return  this.priority>o.priority?-1:1;
    }
```

PriorityBlockingQueue它其实是一个特殊的无界队列，它其中无论添加了多少个任务，线程池创建的线程数也不会超过corePoolSize的数量，只不过其他队列一般是按照先进先出的规则处理任务，而PriorityBlockingQueue队列可以自定义规则根据任务的优先级顺序先后执行。

# 七、拒绝策略

在不能分配任务的时候拒绝

+ AbortPolicy策略：该策略会直接抛出异常，阻止系统正常工作；

+ CallerRunsPolicy策略：如果线程池的线程数量达到上限，该策略会把任务队列中的任务放在调用者线程当中运行；
+ DiscardOledestPolicy策略：该策略会丢弃任务队列中最老的一个任务，也就是当前任务队列中最先被添加进去的，马上要被执行的那个任务，并尝试再次提交
+ DiscardPolicy策略：该策略会默默丢弃无法处理的任务，不予任何处理。当然使用此策略，业务场景中需允许任务的丢失
+ 自定义策略： 实现RejectedExecutionHandler接口

# 八、关闭线程池

## shutdown

有序关闭线程池：不会接受新的任务、关闭空闲线程、**工作队列中的任务还要执行**、线程池状态为SHUTDOWN、不等待正在执行的线程完成、尝试设置线程状态为TERMINATED（最终状态）

```java
public void shutdown() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
          // 如果有安全管理器，请确保调用者通常有权关闭线程
            checkShutdownAccess();
          // 一直尝试将线程池状态设置为SHUTDOWN，原子性设置值
            advanceRunState(SHUTDOWN);
          // 关闭空闲的线程
            interruptIdleWorkers();
            onShutdown(); // hook for ScheduledThreadPoolExecutor
        } finally {
            mainLock.unlock();
        }
        // 尝试进入线程池的最终状态 
        tryTerminate();
    }
```

如何区别线程是空闲的还是忙碌的？

因为是通过Worker这个对象来执行任务的，所以线程状态取决于Worker的状态。`Worker`继承了`AbstractQueuedSynchronizer`，`AbstractQueuedSynchronizer `是`JUC`下很多类的父类，有个`state`字段。通过state 来判断状态

1. `tryLock` -> `tryAcquire(1)` -> `compareAndSetState(0, 1)` 将`state`值从`0`变成`1`。
2. `lock` -> `acquire(1)` -> `tryAcquire(1)` -> `compareAndSetState(0, 1)` 将`state`值从`0`变成`1`。
3. `unlock` -> `release` -> `tryRelease` -> `setState(0)` 将`state`值设为`0`。

Worker状态变化：

1. `Worker`创建的时候，将`state`初始化为`-1`。
2. 启动时：`runWorker` -> `w.unlock`将`state`值设为`0`，。
3. 执行任务时：`getTask()` -> `w.lock` -> `task.run()` -> `w.unlock()`，在获取到任务之后将`state`值设为`1`，执行完任务之后将`state`值设为`0`。

```java
 Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
				// 设置state=0，允许被中断
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
            while (task != null || (task = getTask()) != null) {
              // 开始执行任务，设置state=1，不允许被中断
                w.lock();
                // If pool is stopping, ensure thread is interrupted;
                // if not, ensure thread is not interrupted.  This
                // requires a recheck in second case to deal with
                // shutdownNow race while clearing interrupt
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                    wt.interrupt();
                try {
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                        task.run();
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                        afterExecute(task, thrown);
                    }
                } finally {
                    task = null;
                    w.completedTasks++;
                  // 执行完当前任务，设置state=0
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
```

- -1：初始状态，线程还没启动。
- 0：线程已执行完任务，还没执行新任务。（空闲，已释放锁）
- 1：线程正在执行任务。（繁忙，拥有锁）

```java
private void interruptIdleWorkers(boolean onlyOne) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
          // 遍历所有线程
            for (Worker w : workers) {
                Thread t = w.thread;
              // 线程空闲并且没有被终止->才尝试终止
              // 在这里尝试给线程上锁，如果上锁成功的话，说明state=0，没有执行任务
                if (!t.isInterrupted() && w.tryLock()) {
                    try {
             // 线程的中断
                        t.interrupt();
                    } catch (SecurityException ignore) {
                    } finally {
                        w.unlock();
                    }
                }
                if (onlyOne)
                    break;
            }
        } finally {
            mainLock.unlock();
        }
    }
```

## shutdownNow

立刻关闭线程池：尝试停止所有正在执行的任务、停止工作队列中任务的处理、并返回等待执行的任务列表。

这些任务在从该方法返回时从任务队列中排出（删除）

```java
public List<Runnable> shutdownNow() {
        List<Runnable> tasks;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
          // 如果有安全管理器，请确保调用者通常有权关闭线程
            checkShutdownAccess();
          // 设置线程池状态为STOP
            advanceRunState(STOP);
          
            interruptWorkers();
          // 返回工作队列中的任务
            tasks = drainQueue();
        } finally {
            mainLock.unlock();
        }
       // 尝试进入线程池的最终状态 
        tryTerminate();
        return tasks;
    }
```

```java
private void interruptWorkers() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
          // 遍历所有线程
            for (Worker w : workers)
                w.interruptIfStarted();
        } finally {
            mainLock.unlock();
        }
    }

void interruptIfStarted() {
            Thread t;
  					// 线程处于繁忙或者空闲并且没有被中断
            if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
                try {
           // 中断线程
                    t.interrupt();
                } catch (SecurityException ignore) {
                }
            }
        }
```

shutdownNow与shutdown 都不在接受新的任务，shutdown会执行工作队列中的任务，也仅是终止空闲的线程；shutdownNow会中止工作队列中的任务，中断所有线程.

都执行 Thread#interrupt(); 方法，但是如果恰好有线程接受到任务，之后在执行interrupt() 也会将任务执行完毕，interrupt只是设置一个标志位

## tryTerminate

如果（SHUTDOWN 并且池和队列为空）或（STOP 并且池为空），则转换到 TERMINATED 状态。必须在任何可能导致终止的操作之后调用此方法——减少工作人员数量或在关闭期间从队列中删除任务

```
final void tryTerminate() {
        for (;;) {
            int c = ctl.get();
            if (isRunning(c) ||
                runStateAtLeast(c, TIDYING) ||
                (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
                return;
            if (workerCountOf(c) != 0) { // Eligible to terminate
                interruptIdleWorkers(ONLY_ONE);
                return;
            }

            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                    try {
                        terminated();
                    } finally {
                        ctl.set(ctlOf(TERMINATED, 0));
                        termination.signalAll();
                    }
                    return;
                }
            } finally {
                mainLock.unlock();
            }
            // else retry on failed CAS
        }
    }
```



# 九、线程池监控

常见的思路是 实现 ThreadPoolExecutor中的beforeExecute(wt, task)、afterExecute(task, thrown)、terminated（）。中打印一些线程池的一些情况

获取线程池的一些参数

| getActiveCount()        | 线程池中正在执行任务的线程数量                               |
| ----------------------- | ------------------------------------------------------------ |
| getCompletedTaskCount() | 线程池已完成的任务数量，该值小于等于taskCount                |
| getCorePoolSize()       | 线程池的核心线程数量                                         |
| getLargestPoolSize()    | 线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过，也就是达到了maximumPoolSize |
| getMaximumPoolSize()    | 线程池的最大线程数量                                         |
| getPoolSize()           | 线程池当前的线程数量                                         |
| getTaskCount()          | 线程池已经执行的和未执行的任务总数                           |

```java
/**
 * 线程池监控类
 * 实现ThreadPoolExecutor 中的一些方法。
 *
 * @author peilizhi
 * @date 2022/8/14 13:48
 **/
@Slf4j
public class ThreadPoolMonitor extends ThreadPoolExecutor {

    /**
     * 保存任务开始执行的时间，当任务结束时，用任务结束时间减去开始时间计算任务执行时间
     */
    private ConcurrentHashMap<String, LocalDateTime> startTimes;

    /**
     * 线程池名称，一般以业务名称命名，方便区分
     */
    private String poolName;

    /**
     * 调用父类的构造方法，并初始化HashMap和线程池名称
     *
     * @param corePoolSize    线程池核心线程数
     * @param maximumPoolSize 线程池最大线程数
     * @param keepAliveTime   线程的最大空闲时间
     * @param unit            空闲时间的单位
     * @param workQueue       保存被提交任务的队列
     * @param poolName        线程池名称
     */
    public ThreadPoolMonitor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
                             TimeUnit unit, BlockingQueue<Runnable> workQueue, String poolName) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
                Executors.defaultThreadFactory(), poolName);
    }


    /**
     * 调用父类的构造方法，并初始化HashMap和线程池名称
     *
     * @param corePoolSize    线程池核心线程数
     * @param maximumPoolSize 线程池最大线程数
     * @param keepAliveTime   线程的最大空闲时间
     * @param unit            空闲时间的单位
     * @param workQueue       保存被提交任务的队列
     * @param threadFactory   线程工厂
     * @param poolName        线程池名称
     */
    public ThreadPoolMonitor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
                             TimeUnit unit, BlockingQueue<Runnable> workQueue,
                             ThreadFactory threadFactory, String poolName) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory);
        this.startTimes = new ConcurrentHashMap<>();
        this.poolName = poolName;
    }

    /**
     * 线程池延迟关闭时（等待线程池里的任务都执行完毕），统计线程池情况
     */
    @Override
    public void shutdown() {
        // 统计已执行任务、正在执行任务、未执行任务数量
        log.info("{} Going to shutdown. Executed tasks: {}, Running tasks: {}, Pending tasks: {}",
                this.poolName, this.getCompletedTaskCount(), this.getActiveCount(), this.getQueue().size());
        super.shutdown();
    }

    /**
     * 线程池立即关闭时，统计线程池情况
     */
    @Override
    public List<Runnable> shutdownNow() {
        // 统计已执行任务、正在执行任务、未执行任务数量
        log.info("{} Going to immediately shutdown. Executed tasks: {}, Running tasks: {}, Pending tasks: {}",
                this.poolName, this.getCompletedTaskCount(), this.getActiveCount(), this.getQueue().size());
        return super.shutdownNow();
    }

    /**
     * 任务执行之前，记录任务开始时间
     */
    @Override
    protected void beforeExecute(Thread t, Runnable r) {
        startTimes.put(String.valueOf(r.hashCode()), LocalDateTime.now());
        super.beforeExecute(t, r);
    }

    /**
     * 任务执行之后，计算任务结束时间
     */
    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        LocalDateTime startDate = startTimes.remove(String.valueOf(r.hashCode()));
        LocalDateTime finishDate = LocalDateTime.now();
        final long millis = Duration.between(startDate, finishDate).toMillis();
        // 统计任务耗时、初始线程数、核心线程数、正在执行的任务数量、
        // 已完成任务数量、任务总数、队列里缓存的任务数量、池中存在的最大线程数、
        // 最大允许的线程数、线程空闲时间、线程池是否关闭、线程池是否终止
        log.info("{}-pool-monitor: " +
                        "Duration: {} ms, PoolSize: {}, CorePoolSize: {}, Active: {}, " +
                        "Completed: {}, Task: {}, Queue: {}, LargestPoolSize: {}, " +
                        "MaximumPoolSize: {},  KeepAliveTime: {}, isShutdown: {}, isTerminated: {}",
                this.poolName,
                millis, this.getPoolSize(), this.getCorePoolSize(), this.getActiveCount(),
                this.getCompletedTaskCount(), this.getTaskCount(), this.getQueue().size(), this.getLargestPoolSize(),
                this.getMaximumPoolSize(), this.getKeepAliveTime(TimeUnit.MILLISECONDS), this.isShutdown(), this.isTerminated());
        // 子类中最好调用父类的方法，否则当前方法不会执行
        super.afterExecute(r, t);
    }

    /**
     * 创建固定线程池，代码源于Executors.newFixedThreadPool方法，这里增加了poolName
     *
     * @param nThreads 线程数量
     * @param poolName 线程池名称
     * @return ExecutorService对象
     */
    public static ExecutorService newFixedThreadPool(int nThreads, String poolName) {
        return new ThreadPoolMonitor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>(), poolName);
    }

    /**
     * 创建缓存型线程池，代码源于Executors.newCachedThreadPool方法，这里增加了poolName
     *
     * @param poolName 线程池名称
     * @return ExecutorService对象
     */
    public static ExecutorService newCachedThreadPool(String poolName) {
        return new ThreadPoolMonitor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<>(), poolName);
    }

    /**
     * 生成线程池所用的线程，只是改写了线程池默认的线程工厂，传入线程池名称，便于问题追踪
     */
    static public class EventThreadFactory implements ThreadFactory {
        private static final AtomicInteger poolNumber = new AtomicInteger(1);
        private final ThreadGroup group;
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;

        /**
         * 初始化线程工厂
         *
         * @param poolName 线程池名称
         */
        public EventThreadFactory(String poolName) {
            SecurityManager s = System.getSecurityManager();
            group = Objects.nonNull(s) ? s.getThreadGroup() : Thread.currentThread().getThreadGroup();
            namePrefix = poolName + "-pool-" + poolNumber.getAndIncrement() + "-thread-";
        }

        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(group, r, namePrefix + threadNumber.getAndIncrement(), 0);
            if (t.isDaemon()) {
                t.setDaemon(false);
            }
            if (t.getPriority() != Thread.NORM_PRIORITY) {
                t.setPriority(Thread.NORM_PRIORITY);
            }
            return t;
        }
    }
}
```



# 十、实战

线程池中的线程数取决于业务是什么样的？

+ Cpu 密集型：执行大量计算，线程数设置为 CPU核心数+1 
+ IO 密集型：大量IO操作，线程数设置为  2* CPU核心数

两种使用线程池的常用思路

+ 一个线程处理一个业务

  处理的时候看起来很快，但是如果业务过多（业务数大于线程池最大数量），并且处理的时间略长的话，可能会导致无法获取到线程连接。

  使用于业务量小的情景。并且需要快速返回的情景

+ 一个线程处理多个业务

  不会有请求的线程大于线程池，出现拒绝、阻塞的情况。

  但是处理起来很慢，不太适合需要快速响应的需求。

  可用于业务中的洗数据逻辑。

java 中查看CPU 核心数 ： int ncpus = Runtime.getRuntime().availableProcessors();

**线程池不允许使用Executors去创建，而要通过ThreadPoolExecutor方式**

设置线程池数量：

```java
            /**
             * Nthreads=CPU数量
             * Ucpu=目标CPU的使用率，0<=Ucpu<=1
             * W/C=任务等待时间与任务计算时间的比率
             */
            Nthreads = Ncpu*Ucpu*(1+W/C)
```

使用线程池时最好任务独立，不需要返回值。



在Springboot中有两种方式使用线程池：

+ 注册ThreadPoolExecutor 到容器中
+ 使用@Async 

## 注册ThreadPoolExecutor

```java
@Bean("doSomethingExecutor")
    public Executor doSomethingExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 核心线程数：线程池创建时候初始化的线程数
        executor.setCorePoolSize(10);
        // 最大线程数：线程池最大的线程数，只有在缓冲队列满了之后才会申请超过核心线程数的线程
        executor.setMaxPoolSize(10);
        // 缓冲队列：用来缓冲执行任务的队列
        executor.setQueueCapacity(500);
        // 允许线程的空闲时间60秒：当超过了核心线程之外的线程在空闲时间到达之后会被销毁
        executor.setKeepAliveSeconds(60);
        // 线程池名的前缀：设置好了之后可以方便我们定位处理任务所在的线程池
        executor.setThreadNamePrefix("do-something-");
        // 设置拒绝策略  如果线程池没有Shutdown，则直接调用Runnable任务的run()方法。
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
```

```java
@Resource
    private ThreadPoolTaskExecutor doSomethingExecutor;

// 执行没有返回值的
    public void doSomething() {
        for (int i = 0; i < 50; i++) {
            doSomethingExecutor.execute(() -> {
                while (true) {
                    log.info("doSomething");
                }
            });
        }
    }

    public void doSomething2() throws ExecutionException, InterruptedException {
      // 获取有返回值的
        for (int i = 0; i < 50; i++) {
            Future<String> something2 = doSomethingExecutor.submit(() -> {
                return "kjfhgh";
            });

            final String s = something2.get();
            log.info("s={}", s);
        }
    }
```

## @Async 

1、首先开启自动异步

可以在配置类上，也可以在启动类上

```java
@Configuration
@EnableAsync
public class AsyncConfiguration 
```

2、@Async 

```java
// 执行执行器
@Async(value = "annotionExecutor")
    public void doSomething3() {
        for (int i = 0; i < 50; i++) {
            log.info("doSomething3");
        }
    }
```

如果只使用ThreadPoolTaskExecutor, 是可以不用在Application启动类上面加上@EnableAsync注解的哦！！！ 

多次测试发现ThreadPoolTaskExecutor执行比@Async要快！！！

SpringBoot 默认线程池配置

```properties
# 核心线程数
spring.task.execution.pool.core-size=8  
# 最大线程数
spring.task.execution.pool.max-size=16
# 空闲线程存活时间
spring.task.execution.pool.keep-alive=60s
# 是否允许核心线程超时
spring.task.execution.pool.allow-core-thread-timeout=true
# 线程队列数量
spring.task.execution.pool.queue-capacity=100
# 线程关闭等待
spring.task.execution.shutdown.await-termination=false
spring.task.execution.shutdown.await-termination-period=
# 线程名称前缀
spring.task.execution.thread-name-prefix=task-
# 默认的拒绝测试是 new ThreadPoolExecutor.AbortPolicy()
```

