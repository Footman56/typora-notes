synchronized 是JVM层面关键字、自动上锁

ReentrantLock  是jdk API 层面的类型、手动通过lock 加锁 必须在finally 中调用 unlock()  释放锁。

ReentrantLock 功能更强

+ 等待中断    lockInterruptibly()   可以响应中断，不想等可以取消
+ 公平锁支持：  构造函数true 可以实现公平锁，synchronized 只能是非公平
+ 多条件绑定，通过Condition  可以分组唤醒。 synchronized 只能全部唤醒或者随机唤醒

底层实现：

synchronized 是基于对象头和Monitor 

ReentrantLock：通过AQS和CAS操作实现的



# 解释一下AQS 

一个状态、一个队列、两个动作

已的核心思想可以概括为：‘一个状态，一个队列，两个动作’。
1. 核心数据结构：
。 State（资源状态）：内部维护了一个 volatile int state 变量。
•比如 ReentrantLock 中，state=0代表无锁，state>0代表有锁及重入次数。
• 比如 Semaphore 中，state 代表剩余信号量的数量。
。 CLH 队列（等待队列）：当线程抢不到资源（State）时，AQS 会把该线程封装成一个 Node 节点，扔到一个 FIFO 的双向链表中排队挂起。
2. 核心工作流程：
• 加锁（Acquire）：尝试用 CAS 操作修改 state。
•如果成功，拿锁走人。
• 如果失败，封装成 Node 入队，并调用 LockSupport.park（）让线程阻塞。

解锁（Release）：修改 state（通常是减 1）。
• 如果 state 归零，表示锁彻底释放，AQS 会唤醒（unpark）队列头部的线程去抢锁。
3.设计模式：
图灵课堂
图灵课堂
。AQS 使用了模板方法模式（Template Method Pattern）。它封装了排队、阻塞、唤醒等复杂的底层逻辑，子类（如 ReentrantLock）只需要实现
tryAcquire 和 tryRelease 等简单的方法来定义“什么是获取成功，什么是释放成功’即可。”







# 多线程下保证线程安全

1. 使用无状态的对象（只使用局部变量），不可变 final 字段修饰的

2. 互斥同步锁

   1. synchronized（方法、代码块、静态方法）
   2.  ReentrantLock

3.  非阻塞同步

   1. 原子类  （AtomicInteger） 利用CAS

   2. Volatile  **可见性**（一个线程修改了值，其他线程立马能看到）和**禁止指令重排序**。它**不能保证原子性**。例如 `count++` 这种“读取-修改-写入”的操作，使用 `volatile` 修饰依然不安全，需要配合原子类或锁使用。

4. ThrealLock  ,数据单个线程内访问

5. 安全的容器

   1. `ConcurrentHashMap`（替代 `Hashtable` 或 `Collections.synchronizedMap`）
   2. `CopyOnWriteArrayList`（适合读多写少的场景）
   3. `BlockingQueue`（常用于生产者-消费者模式）