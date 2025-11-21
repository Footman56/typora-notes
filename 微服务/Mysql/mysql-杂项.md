# 字段设计

1. 先确定字段类型 是数字、字符串、时间
2. 确定字段的范围
   + 如果是整数，但是位数有限 就选择 tinyint(5) , tinyint  固定1字节，5表示显示宽度。需要搭配  ZEROFILL效果是：    00005
   + 如果是整数，范围比较大就选择int
   + 如果是小数，就选 Decimal  精度大
   + 如果是固定的字符串就选择 char (5)   如果字符串是 'a' ，底层存储的时候会补充空格  'a____',检索的时候会忽略
   + 如果是不固定的字符串就选择varchar(5)  ,插入时位数不足的时候不会补充空格
   + 字符过长的时候选择 TEXT， 最好单独选择一张表，进行主键关联
   + 时间就选择DATETIME

varchar 长度变化

有两种策略

+ COPY: 创建一章新的表，会缩表
  + 缩小长度
  + 字节数<= 255  调整到  字节数>= 255
  + 更换字符集
+ INSTANT： 实时更新不需要索引
  + 字节数<= 255  调整到 字节数<= 255 
  + 字节数 > 255 调整到 字节数 > 255 

字节数计算 取决于字符集   utf8bm4 一个字符占用4个字节，utf8bm4 的临界值为 255/4 = 63 小于 63都是安全的

varcha(10) -> varchar(100)  : 10 * 4 = 40 字节   变成   100* 4 = 400字节  采用COPY 策略



# mysql 执行顺序

```
mysql 执行顺序
1、FROM  table1 left join table2 on 将table1和table2中的数据产生笛卡尔积，生成Temp1

2、JOIN table2  所以先是确定表，再确定关联条件

3、ON table1.column = table2.columu 确定表的绑定条件 由Temp1产生中间表Temp2

4、WHERE  对中间表Temp2产生的结果进行过滤  产生中间表Temp3

5、GROUP BY 对中间表Temp3进行分组，产生中间表Temp4

6、HAVING  对分组后的记录进行聚合 产生中间表Temp5

7、SELECT  对中间表Temp5进行列筛选，产生中间表 Temp6

8、DISTINCT 对中间表 Temp6进行去重，产生中间表 Temp7

9、ORDER BY 对Temp7中的数据进行排序，产生中间表Temp8

10、LIMIT 对中间表Temp8进行分页，产生中间表Temp9
```

# select

Mysql以表格形式(行和列)显示查询输出。第一行包含列的标签。下面的行是查询结果。通常，列标签是从数据库表中获取的列的名称。如果检索的是一个表达式的值，mysql使用表达式本身来标记该列。

**Mysql显示了返回了多少行以及查询执行了多长时间**

如果接口的入参是List<Integer> 的时候，在执行判断集合是否为空的情况下，if 判断条件不能 使用

<if test="planIds!=null and planIds!='' ">,如果使用的话，会默认将入参转换成List<String>形式。

```xml
select todo.id as id, todo.plan_id as planId, todo.company_id as companyId, todo.assessee_emp_id as
        assesseeEmpId,
        todo.todo_emp_id as todoEmpId , todo.todo_id as todoId, process.review_person_type as reviewType
        from kpi.kpi_process_todo todo
        left join kpi.kpi_process_info process
        on todo.process_id = process.process_id
        where todo.company_id = #{companyId}
        and todo.process_status = 0
        and todo.todo_status = 0
        and todo.is_del = 0
        <if test="empIds!=null and empIds.size>0 ">
            and todo.assessee_emp_id in
            <foreach collection="empIds" item="emp" open="(" close=")" separator=",">
                #{emp}
            </foreach>
        </if>

        <if test="planIds!=null and planIds.size>0">
            and todo.plan_id in
            <foreach collection="planIds" open="(" close=")" separator="," item="plan">
                #{plan}
            </foreach>
        </if>
        and process.company_id = #{companyId}
        and todo.todo_emp_id = #{todoEmpId}
        <if test="reviewTypes!=null and reviewTypes.size>0">
            and process.review_person_type in
            <foreach collection="reviewTypes" open="(" close=")" separator="," item="reviewType">
                #{reviewType}
            </foreach>
        </if>
```

# 变量

MySQL 中变量一般分为两类：**用户变量** 和 **系统变量**，用户变量的变量名格式为 `@variable`，而系统变量的格式为 `@@variable`，`tx_isolation` 是系统变量，所以变量名为 `@@tx_isolation`。

其中，系统变量又可以分为 **全局变量** 和 **会话变量**，默认情况下使用 `select @@variable` 查询出来的是会话变量的值，如果要查询全局变量的值，则使用 `select @@global.variable`。

在mysql 5.0 中变量叫tx_isolation ，在8.0 变量叫：transaction_isolation 用于查看当时的事务隔离级别

# order by

排序有两种,index ，file sort ，索引排序和文件排序。使用索引排序的性能很好，文件排序的性能会差很多。想要使用 索引排序的时候，需要满足一下条件：

1. 查询字段的顺序，和order by条件要满足辅助索引的顺序。最左前缀原则
2. 排序条件只能是升序才会走索引，Mysql8以上版本有降序索引可以支持该种查询方式。
3. 索引前面字段使用范围查询的话，不会使用索引排序



file sort  分两种类型：

单路排序：将满足条件的全部数据（排序列、普通数据列）一次性放入sort buffer 中，在sort buffer 中排序。排序好之后全部返回

sort_model：< sort_key, additional_fields >或者< sort_key, packed_additional_fields > 

双路排序：将排序条件列 ，主键id  放入内存中，在内存中排序，排序好之后根据id 回表查询。

sort_model :< sort_key, rowid > 



MySQL 通过比较系统变量 max_length_for_sort_data(**默认1024字节**) 的大小和需要查询的字段总大小来 判断使用哪种排序模式。 

如果 字段的总长度小于max_length_for_sort_data ，那么使用 单路排序模式； 

如果 字段的总长度大于max_length_for_sort_data ，那么使用 双路排序模∙式。 

 

Order by 与limit 关系：

将 LIMIT row_count 与 ORDER BY 结合使用，MySQL 会在找到排序结果的前 row_count 行后立即停止排序，而不是对整个结果进行排序。

在这种情况下，排序规则不确定的情况下（只对col1, col2 排序的时候，如果col1 中有相同的值，那么出现几次排序的时候顺序不一致的问题）。

想要保证分页下每页的数量都是一致的话，就要对唯一的字段进行排序