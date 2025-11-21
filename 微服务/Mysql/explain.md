explain 用于查看sql执行的执行的情况，如果form 中有子查询的话，仍会执行该子查询，结果存入临时表中

```sql
EXPLAIN SELECT * FROM table_name WHERE plan_id ='';
```

| id                  | select_type        | table          | Partitions | type         | possible_keys            | key                         | key_len | ref                             | rows                                          | Filtered | extra      |
| ------------------- | ------------------ | -------------- | ---------- | ------------ | ------------------------ | --------------------------- | ------- | ------------------------------- | --------------------------------------------- | -------- | ---------- |
| SELECT 查询的标识符 | SELECT 查询的类型. | 查询的是哪个表 |            | 执行索引类型 | 此次查询中可能选用的索引 | 此次查询中确切使用到的索引. |         | 哪个字段或常数与 key 一起被使用 | 显示此查询一共扫描了多少行. 这个是一个估计值. |          | 额外的信息 |

# id

d列的编号是 select 的序列号，有几个 select 就有几个id，并且id的顺序是按 select 出现的顺序增长的。

**id列越大执行优先级越高**，id相同则从上往下执行，id为NULL最后执行。

# select_type

+ SIMPLE  表示此查询不包含 UNION 查询或子查询

  ```mysql
  explain  select * from film where id = 1;
  ```

  <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262228665.png" alt="image-20230626222825540" style="zoom:50%;" />

+ PRIMARY: 复杂查询中最外层的select

  <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262242108.png" alt="image-20230626224206057" style="zoom:50%;" />

+ UNION, 表示此查询是 UNION 的第二或随后的查询

  ```mysql
  explain select * from `film` UNION  all select * from `film` as one  where id =1
  ```

  <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262249839.png" alt="image-20230626224916776" style="zoom:50%;" />

+ UNION RESULT, UNION 的结果

+ SUBQUERY :子查询, 不在from 查询语句中

+ derived ：包含在 from 子句中的子查询。MySQL会将结果存放在一个临时表中，也称为派生表

```mysql
# 关闭5.7之后对衍生表的合并优化
set session optimizer_switch='derived_merge=off'; 

# 打开对衍生表的合并优化
set session optimizer_switch='derived_merge=on'; 
```

未关闭衍生表合并优化的执行情况

![image-20230626223256020](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262232057.png)

关闭后效果：

要执行3次Select ，并且创建了临时表

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262236421.png" alt="image-20230626223613376" style="zoom:50%;" />

# table 

当前查询语句正在使用哪张表

有比较简单的：使用单表；也有复杂的使用临时表

使用单表：

![image-20230626223256020](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262255883.png)

使用临时表：

< derived id> 表示当前查询依赖 id=N 的查询，于是先执行 id=N 的查询。

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262236421.png" alt="image-20230626223613376" style="zoom:50%;" />

# type

可以根据type 来确定如何查询

+ Null ： 在Sql 优化分析的时候，发现可以不用查询表或者索引，能够立刻返回结果

  ```mysql
  explain select max(id) from film;
  ```

  <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262303463.png" alt="image-20230626230307401" style="zoom:50%;" />

+ system：结果集只有一条数据. System 是特殊的const

+ const: 针对主键或唯一索引的**与常数比较**, 表最多有一个匹配行，读取1次

  ```mysql
  explain  select * from (select * from film where id = 1) tmp;
  ```

  ![image-20230626230749652](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262307717.png)

+ eq_ref:   primary key 或 unique key 索引的所有部分**被连接使用** ，最多只会返回一条符合条件的记录,一般不会出现

  ```mysql
  explain select * from film_actor left join film on film_actor.film_id = film.id ;
  ```

  <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262314675.png" alt="image-20230626231403608" style="zoom:50%;" />

  如果在左边补充一些条件的话，就不会是全表扫描

  ```mysql
  explain select * from film_actor left join film on film_actor.film_id = film.id where film_actor.id =3 ;
  ```

  <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262317226.png" alt="image-20230626231714159" style="zoom:50%;" />

+ ref: 不使用唯一索引，而是使用普通索引或者唯一性索引的部分前缀，索引要和某个值相比较，**可能会找到多个符合条件的行。**

  ```mysql
  explain select * from film where name = 'film1';
  ```

  

+ range： 使用索引范围查询, 通过索引字段范围获取结果记录. 这个类型通常出现在 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN() 操作中. 无论这个索引是什么类型，主键、二级索引、唯一索引

  ```mysql
  explain select * from actor where id > 1;
  ```

![image-20230626232203029](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306262322105.png)

+ index: **扫描全索引就能拿到结果**，一般是扫描某个二级索引，这种扫描不会从索引树根节点开始快速查找，而是直接对二级索引的叶子节点遍历和扫描，速度还是比较慢的，这种查询一般为使用覆盖索引，二级索引一般比较小，所以这种通常比ALL快一些.

  当结果集可以通过主键索引或者二级索引获取到的情况下，mysql会选择二级索引，因为二级索引数据量小，查询较快

  覆盖索引是从二级索引中就能获取想要的数据，不需要再回表

  ```sql
  EXPLAIN SELECT name FROM  user_info;
  # name 是索引
  
  # film 中只有主键和二级索引，所有查询全部的
  explain select * from film;
  ```

+ ALL: 表示全表扫描

  ```mysql
  explain select * from actor;
  ```

性能：

ALL < index < range < ref < eq_ref < const < system  

#  possible_keys

可能用到的索引

# key

实际用到的索引，如果没有使用索引，则该列是 NULL

可能会出现 possible_keys 里面有多个索引， 但是实际执行的时候 key 为null. 因为实际执行的时候数据可能很少，没有必要走索引（走辅助索引还涉及到回表）

如果想强制mysql使用或忽视possible_keys列中的索引，在查询中使用 force index、ignore index。

# key_len

mysql在索引里使用的字节数，用于具体确认使用了哪些索引

常见类型的字节长度：

char(n)和varchar(n)，5.0.3以后版本中，**n**均代表字符数，而不是字节数，如果是utf-8，一个数字或字母占1个字节，一个汉字占3个字节

| 类型       | 字节数                                                       |
| ---------- | ------------------------------------------------------------ |
| tinyint    | 1                                                            |
| smallint   | 2                                                            |
| int        | 4                                                            |
| bigint     | 8                                                            |
| date       | 3                                                            |
| timestamp  | 4                                                            |
| datetime   | 8                                                            |
| char(n)    | 如果存汉字长度就是 3n 字节                                   |
| varchar(n) | 如果存汉字则长度是 3n + 2 字节，加的2字节用来存储字符串长度，因为varchar是变长字符串 |

如果字段允许为 NULL，需要1字节记录是否为 NULL

索引最大长度是768字节，当字符串过长时，mysql会做一个类似左前缀索引的处理，将前半部分的字符提取出来做索引

# extra

+ Using filesort: MySQL 需额外的排序操作,查询 CPU 资源消耗大.
+ Using index: 覆盖索引扫描, 表示查询在索引树中就可查找所需数据，覆盖索引一般针对的是辅助索引，**不需要回表操作**
+ Using temporary: 查询有使用临时表, 一般出现于排序，可以考虑创建索引来避免
+ Using where： 使用 where 语句来处理结果，并且查询的列未被索引覆盖

+ Using index condition：查询的列不完全被索引覆盖，where条件中是一个前导列的范围；

+ Using filesort ：将用外部排序而不是索引排序，数据较小时从内存排序，否则需要在磁盘完成排序。这种情况下一般也是要考虑使用索引来优化的

+ Select tables optimized away：使用某些聚合函数（比如 max、min）来访问存在索引的某个字段

是否走索引很大程度上取决与你查询的数据是否有序，不建议讲索引的计算结果作为查询条件