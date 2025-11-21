#  case when then else end

```sql
--简单Case函数 

CASE column
WHEN <condition> THEN value
WHEN <condition> THEN value
......
ELSE value END

例：
CASE sex 
         WHEN '1' THEN '男' 
         WHEN '2' THEN '女' 
ELSE '其他' END 


--Case搜索函数 
CASE
WHEN <condition> [,<condition>] THEN value
WHEN <condition> [,<condition>] THEN value
......
ELSE value END

例：
CASE WHEN sex = '1' THEN '男' 
         WHEN sex = '2' THEN '女' 
ELSE '其他' END 


（1）已知数据按照另外一种方式进行分组，分析。
SELECT  SUM(population) ,
        CASE country 
                WHEN '中国'     THEN '亚洲' 
                WHEN '印度'     THEN '亚洲' 
                WHEN '日本'     THEN '亚洲' 
                WHEN '美国'     THEN '北美洲' 
                WHEN '加拿大'  THEN '北美洲' 
                WHEN '墨西哥'  THEN '北美洲' 
        ELSE '其他' END  
FROM    country 
GROUP BY CASE country 
                WHEN '中国'     THEN '亚洲' 
                WHEN '印度'     THEN '亚洲' 
                WHEN '日本'     THEN '亚洲' 
                WHEN '美国'     THEN '北美洲' 
                WHEN '加拿大'  THEN '北美洲' 
                WHEN '墨西哥'  THEN '北美洲' 
        ELSE '其他' END; 
        
        

(2) 更新
UPDATE country set population= 
CASE 
	WHEN population > 600 THEN
		population-200
	WHEN population<500 THEN
		population+300
	else population END;
	
建议加上 else ,否则如果条件都不满足的时候，属性会置为null
```





```
GROUP BY子句中列出来的每个列必须是检索列或有效的表达式（但不能是聚集函数），
如果在SELECT中使用了表达式，则必须在GROUP BY子句中指定相同的表达式。不能使用别名。
GROUP BY子句必须在WHERE 子句之后，ORDER BY 子句之前。

WHERE 子句用于过滤结果，但是对于分组的过滤WHERE子句不行。
因为WHERE子句，是针对行的过滤。要对分组结果进行过滤，必须使用HAVING子句
```



# 汉字排序

**默认编码是utf8mb4，所以其实数据库存储的并不是汉字文本。按照内部的存储方式**

```sql
select * from table order by convert(filed using gbk) asc;
```

# count

在统计数量的时候，也进行排序。

# 查看表锁使用情况

```sql
# 1.查看当前数据库运行的所有事务（一般查不到，除非发生死锁）
SELECT * FROM information_schema.INNODB_TRX; 

# 杀掉查询结果中锁表的trx_mysql_thread_id 
kill trx_mysql_thread_id


# 查询是否锁表 
show OPEN TABLES where In_use > 0;  
# 查询进程  
show processlist 
# 查询到相对应的进程===然后 kill    id 

 
 # 查看正在锁的事务 
 SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;  
 # 查看等待锁的事务 
 SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
 
 # 查看锁执行情况
 show status like 'innodb_row_lock_%';
# Innodb_row_lock_current_waits : 当前等待锁的数量
# Innodb_row_lock_time : 系统启动到现在，锁定的总时间长度
# Innodb_row_lock_time_avg : 每次平均锁定的时间
# Innodb_row_lock_time_max : 最长一次锁定时间
# Innodb_row_lock_waits : 系统启动到现在总共锁定的次数
```

# 查询sql 操作

```sql
show status like "com_insert%"; -- 获得mysql的插入次数;

show status like "com_delete%"; -- 获得mysql的删除次数;

show status like "com_select%"; -- 获得mysql的查询次数;

show status like "uptime"; -- 获得mysql服务器运行时间

show status like 'connections'; -- 获得mysql连接次数
```

# 函数

## ASCII(s)

返回字符串中第一个字符的ACSII码

```sql
select ASCII('xaioli');
# 结果： 120
```

## CHAR_LENGTH（s）

返回字符串的长度

```sql
select CHAR_LENGTH('xiaoli');
# 结果 6
```

## CONCAT(s1,s2,...,sn)

将多个字符串合并

```sql
select CONCAT('xiaoli','xiaohong');
# xiaolixiaohong
```

CONCAT_WS(x,s1,s2,...,sn)

通过字符X 将多个字符串合并

```sql
select CONCAT_WS(',','xiaoli','xiaohong');
# xiaoli,xiaohong
```



[sql函数]: https://www.runoob.com/mysql/mysql-functions.html	"Command + click 跳转"

​	

FIELD(`字段`，`数据`) 获取字段对应数据的主键

功能：结果集的顺序与in条件的顺序是一致

```sql
SELECT * 
FROM `kpi_plan_flow_setting_performance_calibration` 
WHERE `plan_id` IN (
    'd5cae0acd08c407bb774974f9b6d9390'
) 
AND `is_del` = 0 ORDER BY FIELD( `plan_id`, 'd5cae0acd08c407bb774974f9b6d9390',    '1f8b96be7e6140898b0e989f7d74da02'); 
```



# UNION ALL

多个sql 拼接结果集 

```sql
使用 union all 来代替 or  查询.

SELECT *
        FROM kpi.kpi_plan_info
        WHERE
        <include refid="select_plan_condition"/>
        <if test="accountId != null and accountId != ''">
            and create_acc_id = #{accountId}
        </if>

        UNION ALL

        SELECT *
        FROM kpi.kpi_plan_info
        WHERE
        <include refid="select_plan_condition"/>
        <if test="accountId != null and accountId != ''">
            and associates_acc_ids LIKE CONCAT('%', #{accountId}, '%')
        </if>
```

# replace()函数

用于替换string中的一个字符串

replace(file_name,old_String,new_string)

```sql
UPDATE `kpi_header_record` SET `headers`  = REPLACE (headers,"terminatedPlanNum","terminatedPlanNum") WHERE `company_id` ='827ca94aec3347df87836ef9f24bc4ed' and `is_del` =0;

```

**注replace 函数的第一个参数一定不要带“”， 否则的话就会将列里面的数据都更新为 file_name**



# count 优化

+ 使用count(*) 来统计

字段有索引 ： count(*) = count(1) >  count (字段) > count(id)    二级索引容量小于主键索引

字段无索引： count(*) = count(1) >   count(id) >  count (字段) 

count(1)跟count(字段)执行过程类似，不过count(1)不需要取出字段统计，就用常量1做统计，count(字段)还需要取出字段，

count(*)是例外，mysql并不会把全部字段取出来，，不取值，按行累加，效率很高，

count(字段) 不会统计为null 的数据，但是其余的都会统计。



# trace

用于分析sql 执行过程

1. 打开trace ，trace并不是一直打开的，打开的时候会有些性能问题，但是打开关闭很方便

   ```mysql
   # 打开标志，并且展示格式为json
   set optimizer_trace="enabled=on",end_markers_in_JSON=on;
   ```

2. 执行想要分析的SQL

3. 查看执行过程

   ```mysql
   select *from information_schema.optimizer_trace \G;
   ```


# 强制走索引

```mysql
EXPLAIN SELECT * FROM employees force index(idx_name_age_position) WHERE name > 'LiLei' AND age = 22 ND position ='manager';
```

强制走索引并不意味着效率会更好，因为二级索引需要回表，有时反而不如全表扫描

# update ... case... when ... 

用于批量更新

有两种格式

```mysql
Type 1: CASE value WHEN [compare-value] THEN result [WHEN [compare-value] THEN result ...] [ELSE result] END

UPDATE `goods` SET `type` = (
    CASE `name` WHEN 1 THEN 999  
    WHEN 2 THEN 1000  
    WHEN 3 THEN 1024  
    else type
    END)
```

注：此时不带where 条件的话，会更新所有内容

**如果条件中没有携带 ELSE 时 其他的内容（name = 4,5）时，type 会变成null**没有匹配的结果值,则返如果没有ELSE 部分,则返回值为NULL,如果字段为NOT NULL则会根据不同数据类型返回不同的值（字符串类型时返回空字符串，数值类型时返回0，其它类型未做测试）.

```mysql
UPdate `goods` set `type` = (
case 
  when `name` = 1 then 999
  when `name` = 2 then 1000
  when `name` = 3 then 1024
  else `type`
  end
);
```



# trim 标签

标签用于截取并拼接，即可以在条件判断完的 SQL 语句前后，添加或者去掉指定的字符。

属性：

+ prefix：（添加前缀）trim标签体中是整个字符串拼串后的结果，是给拼串后的整个字符串加一个前缀
+ suffix：（添加后缀）suffix给拼串后的整个字符串加一个后缀
+ prefixOverrides：前缀覆盖： 去掉整个字符串前面多余的字符（去掉前缀）
+ suffixOverrides：后缀覆盖：去掉整个字符串后面多余的字符（去掉后缀）

```sql
select * from table 
<trim prefix = 'where' suffix = ';'  prefixOverrides = 'and|or' suffixOverrides = 'and|or'>
     <if test = condition1 >
       xxx = 'a' and 
     </if>
     <if test = condition2 >
       yyy = 'b' or
     </if>
</trim>

# 如果condition1 不满足的话，最终结果是：
select * from table where yyy= 'b';


# 如果condition2 不满足的话，最终结果是：
select * from table where xxx = 'a';

# 如果condition1、condition2 不满足的话
select * from table;
```

# in

mysql 不会对in条件集合做控制，但是会对整条sql语句做限制。mysql根据配置文件会限制server接受的数据包大小。有时候大的插入和更新会受max_allowed_packet 参数限制

```mysql
show VARIABLES  like 'max_allowed_packet';
67108864 / 1024/ 1024 = 64M
```

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202401031115572.png" alt="image-20240103111518818" style="zoom:50%;" />



在mybatis  xml 写查询sql 的时候， test 条件不能出现 is 

```xml
# 这是错误的
<if test="maxCalibrationScore is not null">
# 正确的是
<if test="maxCalibrationScore != null">
```



需求：mybatis  在xml文件实现下面功能，参数minSelfScore表示最小值，selfMinClosedInterval为0 是小于，selfMinClosedInterval为1是小于等于，对应sql 字段是self_score

```xml
<if test="minSelfScore != null">
            <choose>
                <when test="selfMinClosedInterval == 1">
                    and record.self_score &gt;=  #{minSelfScore}
                </when>
                <otherwise>
                    and record.self_score &gt;  #{minSelfScore}
                </otherwise>
            </choose>
</if>
```

left join 时 条件写在on 与 where 中的区别

结论：在where 中会对联表的结果进行过滤。 在on 中

```sql
SELECT * FROM `kpi_plan_flow_setting_evaluate_detail` a left join `kpi_plan_info` b on a.`plan_id` =b.`plan_id`  and a.is_del = 0 AND b.`is_del` =0 
WHERE `flow_id` is  null    ORDER BY  a.`addtime`  asc 
```

这种条件 在on 中配置的话，会出现右侧表有一些为null的，因为left join 取的是左侧表的全部符合条件的数据（条件在 where 中）

![image-20240831132503724](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408311325304.png)



```sql

SELECT * FROM `kpi_plan_flow_setting_evaluate_detail` a left join `kpi_plan_info` b on a.`plan_id` =b.`plan_id`  
WHERE `flow_id` is  null   and a.is_del = 0 AND b.`is_del` =0   ORDER BY  a.`addtime`  asc 
```

这种条件 在on 中配置的话，就不会出现右侧表有一些为null的，因为where 会对结果进行过滤

![image-20240831132715713](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408311327849.png)

```sql
SELECT * FROM `kpi_plan_flow_setting_evaluate_detail` a  inner  join `kpi_plan_info` b on a.`plan_id` =b.`plan_id`   and a.is_del = 0 AND b.`is_del` =0
WHERE `flow_id` is  null      ORDER BY  a.`addtime`  asc 
;
```

使用inner join  条件写在on 中也能实现相同的效果，并且性能稍微好一点

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408311332866.png" alt="image-20240831133250802" style="zoom:50%;" />

同样是inner join  ,条件写在 on 和where中效果是一样的，但是在where中的性能较差

![image-20240831133228203](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408311332318.png)



**综上，最好的执行方式是 inner join ，将一些条件 写在on, 在构建笛卡尔集的时候就进行数据过滤**

统计数量: 同时返回两种不同条件下统计的数量，比如返回包含评分标准的，不包含评分标准的数量

```mysql
SELECT 
    company_id,
    COUNT(CASE WHEN target_items LIKE '%评分标准%' THEN 1 END) AS count_with_criteria,
    COUNT(CASE WHEN target_items NOT LIKE '%评分标准%' THEN 1 END) AS count_without_criteria
FROM 
    kpi_assessee_target_detail
GROUP BY 
    company_id;
```



两张表，两个字段的校验规则不一致时，可以显示统一校验规则

```mysql
SELECT *
FROM 表A
INNER JOIN 表B
ON 表A.字段1 COLLATE utf8mb4_general_ci = 表B.字段2;
```

两个

```mysql
SELECT 
    type,
    CASE 
        WHEN type = 31 THEN '年度考核'
        WHEN type = 30 THEN '季度考核'
        ELSE '其他考核'
    END AS 考核类型
FROM assessments;
```



#  慢sql 优化

1. 检查数据量是否太大，如果太大的话建议分库分表
2. 如果数据数据太大的话，检查里面数据是否有大量的无用数据（is_del=1这种），如果有的话，就通过定时任务删除数据
3. 拿到sql 通过explain分析下是否走索引，如果不走的话就建立索引，如果走的话，检查是否合适。很多情况下是不需要太多独立字段作为索引的，可以建立联合索引来解决问题。比较通用的字段在第一个，注意最左前缀规则。
4. 检查代码中是否可以通过传递参数来走合适的索引。



修改mysql索引时 分两种情况，

+ 数据量比较小【已千万级为标准】，直接执行即可
+ 数据量比较大的话，需要使用algrothm= inplace ，性能比较好。若使用default 时会锁表





# 批量删除数据

```mysql
DELIMITER //

CREATE PROCEDURE BatchDelete()
BEGIN
    DECLARE rows_deleted INT DEFAULT 1;

    WHILE rows_deleted > 0 DO
        DELETE FROM kpi_draft_temp_data WHERE is_del = 1 LIMIT 10000;
        SET rows_deleted = ROW_COUNT();
        COMMIT;
    END WHILE;
END //

DELIMITER ;

CALL BatchDelete();



# 上面sql 执行完之后再执行
DROP PROCEDURE IF EXISTS BatchDelete;
```

