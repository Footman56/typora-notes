# 一、准备工作

检查是否有docker

```sh
docker --version
```

# 二、创建网络

```sh
docker network create mysql-net
```

# 三、启动主MySQL容器

```sh
docker run --name mysql-master -p 3307:3306  --network=mysql-net -e MYSQL_ROOT_PASSWORD=root_password  \
-v /home/huochai/my_mysql/mysql-master/log:/var/log/mysql \
-v /home/huochai/my_mysql/mysql-master/data:/var/lib/mysql \
-v /home/huochai/my_mysql/mysql-master/conf:/etc/mysql \
-v /home/huochai/my_mysql/mysql-master/mysql-files:/var/lib/mysql-files \
-d   mysql:8
```

在/home/huochai/my_mysql/mysql-master/conf 创建my.cnf

```
[mysqld]

## 设置server_id，同一局域网中需要唯一

server_id=101 

## 指定不需要同步的数据库名称

binlog-ignore-db=mysql  

## 开启二进制日志功能

log-bin=mall-mysql-bin  

## 设置二进制日志使用内存大小（事务）

binlog_cache_size=1M  

## 设置使用的二进制日志格式（mixed,statement,row）

binlog_format=mixed  

## 二进制日志过期清理时间。默认值为0，表示不自动清理。

expire_logs_days=7  

## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。

## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致

slave_skip_errors=1062
```

重启容器

```
docker restart mysql-master
```

# 四、 配置MySQL主节点

1. 进入容器

```
docker exec -it mysql-master bash
```

2. 登录mysql

```mysql
mysql -u root -p root_password
```

3. 执行从库角色命令

   **高版本 mysql 8.0 需要区别创建用户和赋予权限**

```
create user 'replica'@'%' identified by 'slave_password';
grant all privileges on *.* to 'replica'@'%' WITH GRANT OPTION;
ALTER  USER  'replica'  IDENTIFIED  WITH  mysql_native_password  BY  'slave_password';    
FLUSH PRIVILEGES;
```

4. 获取主数据库的二进制日志文件

```
SHOW MASTER STATUS;
```

<img src="https://raw.githubusercontent.com/Footman56/images/master/img202502071812209.png" alt="image-20250207181245023" style="zoom:50%;" />

# 五、启动从节点

1. docker 创建容器

```sh
docker run --name mysql-slave -p 3308:3306 --network=mysql-net -e MYSQL_ROOT_PASSWORD=salve_password  \
-v /home/huochai/my_mysql/mysql-slave/log:/var/log/mysql \
-v /home/huochai/my_mysql/mysql-slave/data:/var/lib/mysql \
-v /home/huochai/my_mysql/mysql-slave/conf:/etc/mysql \
-v /home/huochai/my_mysql/mysql-slave/mysql-files:/var/lib/mysql-files \
-d   mysql:8
```

2. 在/home/huochai/my_mysql/mysql-slave/conf 创建my.cnf

```
[mysqld]

## 设置server_id，同一局域网中需要唯一

server_id=102

## 指定不需要同步的数据库名称

binlog-ignore-db=mysql  

## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用

log-bin=mall-mysql-slave1-bin  

## 设置二进制日志使用内存大小（事务）

binlog_cache_size=1M  

## 设置使用的二进制日志格式（mixed,statement,row）

binlog_format=mixed  

## 二进制日志过期清理时间。默认值为0，表示不自动清理。

expire_logs_days=7  

## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。

## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致

slave_skip_errors=1062  

## relay_log配置中继日志

relay_log=mall-mysql-relay-bin  

## log_slave_updates表示slave将复制事件写进自己的二进制日志

log_slave_updates=1  

## slave设置为只读（具有super权限的用户除外）c
read_only=1
```

3. 重启容器

```
docker restart mysql-slave
```

# 六、配置从节点

1. 进入容器

```
docker exec -it mysql-slave bash
```

2. 登录mysql

```
mysql -u root -p
```

3. 配置主从

```mysql
STOP SLAVE;
CHANGE MASTER TO MASTER_HOST='mysql-master', MASTER_USER='replica', MASTER_PASSWORD='slave_password', MASTER_LOG_FILE='mall-mysql-bin.000001', MASTER_LOG_POS=1161;
START SLAVE;
```

+ `MASTER_HOST='mysql-master'`：主数据库容器名称。

+ `MASTER_USER='replica'`：复制用户。

+ `MASTER_PASSWORD='slave_password'`：复制用户的密码。

+ `MASTER_LOG_FILE='mysql-bin.000003'`：主数据库上的二进制日志文件名（从主数据库`SHOW MASTER STATUS`获取）。

+ `MASTER_LOG_POS=323`：二进制日志的位置（从主数据库`SHOW MASTER STATUS`获取）。

4. 检查配置是否正常

   ```
   SHOW SLAVE STATUS\G
   ```

   Last_IO_Error: error connecting to master 'replica@mysql-master:3306' - retry-time: 60 retries: 1 message: Authentication plugin 'caching_sha2_password' reported error: Authentication requires secure connection.

   解决方式：进入从库，给root设置密码

   ```
   ALTER  USER  'root'  IDENTIFIED  WITH  mysql_native_password  BY  '1234567';    
   
   ALTER  USER  'replica'  IDENTIFIED  WITH  mysql_native_password  BY  '1234567';    
   ```

   有下面数据才算正确

   ```
    Slave_IO_Running: Yes
    Slave_SQL_Running: Yes
   ```

   

# 七、测试

在主库执行操作

```
CREATE DATABASE test_db;
```

在从库执行

```
SHOW DATABASES;
```

```
create user 'huochai'@'%' identified by 'asdf1234';
grant all privileges on *.* to 'huochai'@'%' WITH GRANT OPTION;
ALTER  USER  'huochai'  IDENTIFIED  WITH  mysql_native_password  BY  'asdf1234';    
FLUSH PRIVILEGES;
```

