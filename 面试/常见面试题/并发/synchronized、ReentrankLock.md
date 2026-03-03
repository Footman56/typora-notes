synchronized 是JVM层面关键字、自动上锁

ReentrantLock  是jdk API 层面的类型、手动通过lock 加锁 必须在finally 中调用 unlock()  释放锁。

ReentrantLock 功能更强

+ 等待中断    lockInterruptibly()   可以响应中断，不想等可以取消
+ 公平锁支持：  构造函数true 可以实现公平锁，synchronized 只能是非公平
+ 多条件绑定，通过Condition  可以分组唤醒。 synchronized 只能全部唤醒或者随机唤醒

底层实现：

synchronized 是基于对象头和Monitor 

ReentrantLock

