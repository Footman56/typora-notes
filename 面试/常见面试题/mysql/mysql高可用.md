mysql 也会有这个问题 （主从复制，主节点）

解决方式为：

+ MHA   (被替代啦)

  **原理：** 监控主库 → 主库挂了 → 选一个数据最新的从库提升为主库 → 重建复制关系

  + 依赖 SSH
  + 半自动/自动
  + 需要自己维护

+ 主从+  Orchestrator + proxySQL 

  实时监控 MySQL 拓扑结构，主库故障时自动选举最优从库提升

  ```
  主从复制
  Orchestrator 负责选主 （实时监控 MySQL 拓扑，自动故障转移）
  proxySQL 负责路由
  
  # 自动化高
  ```

+ InnoDB  Cluster (官方推荐)

  ```
  MGR 多节点集群 + MySQL Router
  强一致、自动选举、类似分布式数据库
  ```

+ mysql + Proxy 层

​	高可用提现在：

```
数据复制
+ 故障检测
+ 自动选主
+ 流量切换
+ 防脑裂
```

![image-20260301101116538](https://raw.githubusercontent.com/Footman56/images-2/master/img202603011011184.png)