1. 查询机器上设置的内存

   ```
   GET _cat/nodes?h=heap.max
   ```

2. 查看分片情况

   显示索引名称、分片数、未分配的原因

   ```
   GET  _cat/shards?h=index,shard,prirep,state,unassigned.reason
   ```

   具体情况如下：

   ALLOCATION_FAILED：由于分片分配失败而未分配。
   CLUSTER_RECOVERED：由于集群恢复而未分配。
   DANGLING_INDEX_IMPORTED：由于导入了悬空索引导致未分配。
   EXISTING_INDEX_RESTORED：由于恢复为已关闭的索引导致未分配。
   INDEX_CREATED：由于API创建索引而未分配。
   INDEX_REOPENED：由于打开已关闭索引而未分配。
   NEW_INDEX_RESTORED：由于恢复到新索引而未分配。
   NODE_LEFT：由于托管的节点离开集群而未分配。
   REALLOCATED_REPLICA：确定了更好的副本位置，并导致现有副本分配被取消。
   REINITIALIZED：当分片从开始移动回初始化，导致未分配。
   REPLICA_ADDED：由于显式添加副本而未分配。
   REROUTE_CANCELLED：由于显式取消重新路由命令而未分配。

3. 查询单个索引配置信息

   ```
   GET /appraisal_kpi_assessee_v5/_settings
   
   number_of_shards 主分片数
   number_of_replicas 副本数
   ```

   

4. 查询单个索引分片信息

   ```
   GET /appraisal_kpi_assessee_v5/_search_shards
   ```

5. 查看磁盘使用空间

   ```
   GET /_cat/allocation?v
   ```

   ![image-20230523115914873](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202305231159201.png)

分片数（shards）：这是19，表示我们把数据分成多少块存储。
索引所占空间（disk.indices）：该节点中所有索引在该磁盘所占的空间。
磁盘使用容量（disk.used）：已经使用空间14.6gb。
磁盘可用容量（disk.avail）：可用空间34.4gb.
磁盘总容量（disk.total）：总共容量49gb
磁盘便用率（disk.percent）：磁盘使用率29%。

6. 删除数据（注，只是打上删除标志，但是没有释放物理内存）

   并且测试一次只删除1万条数据

   ```
   POST /employee_info_v1/_delete_by_query?wait_for_completion=false&slices=5&pretty&refresh
   {
     "query":{
       "match_all":{}
     }
   }
   ```

   

7. 合并段空间

   ```
   POST /_forcemerge?only_expunge_deletes=true
   ```

   

8. 查看集群

   ```
   GET _cat/health?v
   ```

   

9. 修复

   ```
   PUT /appraisal_kpi_assessee_target_v1/_settings
   {"index.blocks.read_only_allow_delete": null}
   ```

10. 提交所有操作

```
POST /appraisal_kpi_assessee_target_v1/_flush
```
