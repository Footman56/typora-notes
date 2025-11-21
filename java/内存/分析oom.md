

# 一、需要在java 启动命令中设置参数

```
-XX:-CITime：打印消耗在JIT编译的时间。
-XX:ErrorFile=./hs_err_pid.log：保存错误日志或数据到指定文件中。
-XX:HeapDumpPath=./java_pid.hprof：指定Dump堆内存时的路径。
-XX:-HeapDumpOnOutOfMemoryError：当首次遭遇内存溢出时Dump出此时的堆内存。
-XX:OnError=";"：出现致命ERROR后运行自定义命令。
-XX:-PrintClassHistogram：按下 Ctrl+Break 后打印堆内存中类实例的柱状信息，同JDK的 jmap -histo 命令。
-XX:-PrintConcurrentLocks：按下 Ctrl+Break 后打印线程栈中并发锁的相关信息，同JDK的 jstack -l 命令。
-XX:-PrintCompilation：当一个方法被编译时打印相关信息。
-XX:-PrintGC：每次GC时打印相关信息。
-XX:-PrintGCDetails：每次GC时打印详细信息。
-XX:-PrintGCTimeStamps：打印每次GC的时间戳。
-XX:-TraceClassLoading：跟踪类的加载信息。
-XX:-TraceClassLoadingPreorder：跟踪被引用到的所有类的加载信息。
-XX:-TraceClassResolution：跟踪常量池。
-XX:-TraceClassUnloading：跟踪类的卸载信息。
```

具体使用：

```java
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=/qjyd/log/appraisal/heap_dump_pid%p.hprof
-XX:+PrintGCDateStamps 
-XX:+PrintGCDetails 
-XX:ErrorFile=/qjyd/log/hs_err_pid-%p.log 
-Xloggc:/qjyd/log/appraisal/gc-%t.log
```

通过上述启动配置，可以在内存溢出的时候将信息打印到heap_dump_pid%p.hprof文件中

每次GC的时候回打印日志到/qjyd/log/appraisal/gc-%t.log，

# 二、GC文件

## -XX:+PrintGCDetails 

使用这个命令对打印

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220824233332966.png" alt="image-20220824233332966" style="zoom:50%;" />

# 三、分析dump

先从服务器上下载hprof 文件

# 四、找到最大对象，并且找到是如何生成的

1. 从当前对象集找到最大对象是什么，之后整理这个对象，右键使用选定对象

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220825090504002.png" alt="image-20220825090504002" style="zoom:50%;" />



2. <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220825090629361.png" alt="image-20220825090629361" style="zoom:50%;" />选择传入引用

   

3. 可以发现使用这个引入组合起来的对象

   <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220825090819690.png" alt="image-20220825090819690" style="zoom:50%;" />

   这个对象是业务对象的话，就能大致定位是业务哪步生成出来的

   如果还不能确定的话，需要继续合并引用

   4. 继续合并引用

      ![image-20220825091307267](/Users/peilizhi/Library/Application Support/typora-user-images/image-20220825091307267.png)

      <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220825091044622.png" alt="image-20220825091044622" style="zoom:50%;" />

5. 分析最大对象

   查看最大对象可以判断是不是对象过大

   <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220825091346649.png" alt="image-20220825091346649" style="zoom:50%;" />

6. 查看创建对象的线程堆栈

   <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220825091450884.png" alt="image-20220825091450884" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220825091518392.png" alt="image-20220825091518392" style="zoom:50%;" />

一直展开能看到线程堆栈，记录线程号

之后在线程转储中根据线程号找对应的执行流程

![image-20220825091615473](/Users/peilizhi/Library/Application Support/typora-user-images/image-20220825091615473.png)

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220825091727246.png" alt="image-20220825091727246" style="zoom:50%;" />

7. 分析业务，确定好问题在哪里