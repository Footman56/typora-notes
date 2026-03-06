# 内存模型 

CMS  基于 经典 分代模型、空间是连续的

G1 Region分区模型，每个region 大小相同，可以动态扮演角色：Eden、Survior、Old、Humongous 

# 回收算法

CMS : 标记清除算法

1. 初试标记（STW）
2. 并发标记
3. 重新标记(STW)
4. 并发清除

不会整理内存



G1 Mark + COPY + COMPACT

复制 + 压缩



# 停顿模型区别

CMS 追求并发 、停顿短

G1 可预测停顿，保证停顿在目标时间内

# 碎片处理

CMS 清除后会有碎片问题，导致分配大对象失败，触发Full GC

G1 使用复制整理，不会产生大量碎片，大对象直接使用 Humongous Region

# 堆大小

CMS:4GB ～20GB  ；大堆扫描老年队成本高

G1: 大堆  8GB～1TB



CMS被淘汰

1. 碎片问题严重
2. 无法控制停顿时间
3. 大堆性能差
4. 维护复杂

