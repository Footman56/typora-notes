线程工厂：用于创建线程

```java
public interface ThreadFactory {

    /**
     * Constructs a new {@code Thread}.  Implementations may also initialize
     * priority, name, daemon status, {@code ThreadGroup}, etc.
     *
     * @param r a runnable to be executed by new thread instance
     * @return constructed thread, or {@code null} if the request to
     *         create a thread is rejected
     */
    Thread newThread(Runnable r);
}
```

接口中仅有一个 newThread 方法。调用的时候只需要调用newThread就可以，不用显示的通过 new Thread() 来创建线程

线程工厂可以用于创建有规律的一些线程。方法里面可以设置一些属性给Thread。