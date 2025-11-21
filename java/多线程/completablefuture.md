# 背景

是为了补充Future 的不足。Future用于表示异步计算的结果，只能通过阻塞或者轮询的方式获取结果，而且不支持设置回调方法，也不能够多个Future进行组合使用。

CompletableFuture对Future进行了扩展，可以通过设置回调的方式处理计算结果，同时也支持组合操作，支持进一步的编排

```java
// 轮询判断是否计算完成
ExecutorService es = Executors.newFixedThreadPool(10);
        Future<Integer> f = es.submit(() -> {
            // 长时间的异步计算
            // ……
            // 然后返回结果
            Thread.sleep(1000);
            System.out.println("等待10秒");
            return 100;
        });
        while(!f.isDone()){
            System.out.println("还没有计算完成");
        }
```

# 实现思想

# 功能

## get

能够打印出当前线程的异常堆栈,并且能定位到出问题的代码位置

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220816010458014.png" alt="image-20220816010458014" style="zoom:50%;" />

## join

只能打印实现方法里面的异常，并且能定位到出问题的代码位置，但是看不到主线程的调用链

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220816010538841.png" alt="image-20220816010538841" style="zoom:50%;" />

## 零依赖

## 一元依赖

## 二元依赖

## 多元依赖

# 工具

# 注意

