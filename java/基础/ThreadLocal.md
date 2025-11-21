ThreadLocal 用于在线程内存储变量。一个ThreadLocal对应一个变量，

一个线程内可以定义多个ThreadLocal

在Threa中有ThreadLocal.ThreadLocalMap 用于存储<ThreadLocal ，value>形式的数据。

ThreadLocal 是弱引用



如果不调用ThreadLocal.remove就有可能导致内存泄露哦



# 如果使用？

threadLocal.get() 获取值

threadLocal.set()值

# 结构

## 扩容



# 拓展-弱引用

# 多线程共享

