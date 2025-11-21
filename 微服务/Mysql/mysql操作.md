```mysql
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using passwor:yes)
密码错误

修改密码
1、在 etc/my.cnf文件中添加skip-grant-tables 用于跳过密码登录
2、重启mysql  systemctl restart mysqld.service
3、mysql -uroot -p 直接回车跳过密码登录
4、use mysql
5、update user set authentication_string=’’ where user=‘root’;  先将密码社为空
6、select user,host from user; 查看host
7、修改密码  alter user'root'@'%' IDENTIFIED BY 'Admin123@';  密码要负责（大小写，数字，特殊字符）
8、etc/my.cnf文件中去掉skip-grant-tables 安全
```

```
卸载mysql
//yum安装方式下载
1.查看已安装的mysql
命令：rpm -qa | grep -i mysql
2.卸载mysql
命令：yum remove mysql-community-server-5.6.36-2.el7.x86_64
查看mysql的其它依赖：rpm -qa | grep -i mysql
 
//卸载依赖
yum remove mysql-libs
yum remove mysql-server
yum remove perl-DBD-MySQL
yum remove mysql
```

# yum 安装

```shell
安装MySQL8.0

wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm

#安装yum源
yum localinstall mysql80-community-release-el7-1.noarch.rpm

#更新yum源
yum clean all
yum makecache

#开始安装MySQL
yum install mysql-community-server
--------------------- 


配置文件的位置是 /etc/my.cnf
vim /etc/my.cnf
#修改或添加skip-grant-tables跳过密码
skip-grant-tables


#重启mysql服务：  
service mysqld restart

#将旧密码置空
mysql -u root -p    //提示输入密码时直接敲回车。

#选择数据库
use mysql

#将密码置空
update user set authentication_string = '' where user = 'root';

#退出
quit

#去除免密码登陆

#删掉步骤1的语句  skip-grant-tables

#重启服务  service mysqld restart

#修改密码

mysql -u root -p  //提示输入密码时直接敲回车，刚刚已经将密码置空了

#密码形式过于简单则会报错
ALTER USER 'root'@'localhost' IDENTIFIED BY 'yourpassword' -----------------
```



# brew 安装

```sh
brew install mysql

brew services start mysql
```

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306301037149.png" alt="image-20230630103731784" style="zoom:50%;" />

```shell
user ERROR 1449 (HY000): The user specified as a definer ('mysql.infoschema'@'localhost') does not exist

1、DROP USER 'mysql.infoschema'@'localhost';
2、刷新   FLUSH PRIVILEGES;
3、CREATE USER 'mysql.infoschema'@'localhost' IDENTIFIED BY 'Admin123@';
4、GRANT SELECT ON *.* TO `mysql.infoschema`@`localhost`;
5、FLUSH PRIVILEGES;
```

```
启动失败的时候查看启动日志

sudo vim /var/log/mysqld.log
```

```
1045 access denied for user 'root'@'localhost' using password yes
给用户赋权

use mysql;
select host,user from user;
update user set host="%" where user='xx'
```

# 创建用户

```mysql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';

create user  'huochai'@'%' IDENTIFIED BY 'asdf1234';



# 修改用户密码验证规则，使用简单规则（估计是直接配置字符串，不加密） 正常密码要求是 不少于8字符，保护大小写 等
ALTER USER 'huochai'@'%' IDENTIFIED BY 'asdf1234' PASSWORD EXPIRE NEVER; 
```

```
username：你将创建的用户名
host：指定该用户在哪个主机上可以登陆，如果是本地用户可用localhost，如果想让该用户可以从任意远程主机登陆，可以使用通配符%
password：该用户的登陆密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器
```

# 授权

```mysql
GRANT privileges ON databasename.tablename TO 'username'@'host'


privileges：用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所的权限则使用 ALL
databasename：数据库名
tablename：表名，
如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，  如*.* 全部的库和表

GRANT all ON *.* TO 'huochai'@'%';

SHOW GRANTS FOR user1;
```

**此时创建的用户没有权限去给其他用户授权**

```sql
GRANT privileges ON databasename.tablename TO 'username'@'host' 
WITH GRANT OPTION;
```

# 撤销用户权限

```mysql
REVOKE privilege ON databasename.tablename FROM 'username'@'host';

privileges：用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所的权限则使用 ALL
databasename：数据库名
tablename：表名，
```

**<font color="red"> 常见sql</font>**

```s
select * from(
  select row_number() over (partition by 分组字段 order by 排序字段 desc) as rn,u.*
  from 表名 u 
) t where t.rn=1;

mysql8.0中 查询每个分组中最新数据
在mysql8.0中 group by 列名 与 select 中的列名必须相同
```

架构图

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-05-20 23.39.35.png" style="zoom:50%;" />



```
连接层：最上层是一些客户端和连接服务。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。


服务层：第二层服务层，主要完成大部分的核心服务功能， 包括查询解析、分析、优化、缓存、以及所有的内置函数，所有跨存储引擎的功能也都在这一层实现，包括触发器、存储过程、视图等


引擎层：第三层存储引擎层，存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取


存储层：第四层为数据存储层，主要是将数据存储在运行于该设备的文件系统之上，并完成与存储引擎的交互
```

<img src="/Users/mac/Library/Application Support/typora-user-images/image-20210528131605628.png" alt="image-20210528131605628" style="zoom:50%;" />

```
客户端请求 ---> 连接器（验证用户身份，给予权限）  ---> 查询缓存（存在缓存则直接返回，不存在则执行后续操作） ---> 分析器（对SQL进行词法分析和语法分析操作）  ---> 优化器（主要对执行的sql优化选择最优的执行方案方法）  ---> 执行器（执行时会先看用户是否有执行权限，有才去使用这个引擎提供的接口） ---> 去引擎层获取数据返回（如果开启查询缓存则会缓存查询结果）

在存储引擎出调用接口获取文件中存储的数据
```

查看存储引擎

```mysql
-- 查看支持的存储引擎
SHOW ENGINES

-- 查看默认存储引擎
SHOW VARIABLES LIKE 'storage_engine'

--查看具体某一个表所使用的存储引擎，这个默认存储引擎被修改了！
show create table tablename

--准确查看某个数据库中的某一表所使用的存储引擎
show table status like 'tablename'
show table status from database where name="tablename"
```

在 MySQL中建立任何一张数据表，在其数据目录对应的数据库目录下都有对应表的 `.frm` 文件，`.frm` 文件是用来保存每个数据表的元数据(meta)信息，包括表结构的定义等，与数据库存储引擎无关，也就是任何存储引擎的数据表都必须有`.frm`文件，命名方式为 数据表名.frm，如user.frm。




```
MyISAM 物理文件结构为：

.frm文件：与表相关的元数据信息都存放在frm文件，包括表结构的定义信息等
.MYD (MYData) 文件：MyISAM 存储引擎专用，用于存储MyISAM 表的数据
.MYI (MYIndex)文件：MyISAM 存储引擎专用，用于存储MyISAM 表的索引相关信息

InnoDB 物理文件结构为：


.frm 文件：与表相关的元数据信息都存放在frm文件，包括表结构的定义信息等

.ibd 文件或 .ibdata 文件： 这两种文件都是存放 InnoDB 数据的文件，之所以有两种文件形式存放 InnoDB 的数据，是因为 InnoDB 的数据存储方式能够通过配置来决定是使用共享表空间存放存储数据，还是用独享表空间存放存储数据。
独享表空间存储方式使用.ibd文件，并且每个表一个.ibd文件
共享表空间存储方式使用.ibdata文件，所有表共同使用一个.ibdata文件（或多个，可自己配置）
```

```
InnoDB 支持事务，MyISAM 不支持事务。这是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一；
InnoDB 支持外键，而 MyISAM 不支持。对一个包含外键的 InnoDB 表转为 MYISAM 会失败；
InnoDB 是聚簇索引，MyISAM 是非聚簇索引。聚簇索引的文件存放在主键索引的叶子节点上，因此 InnoDB 必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而 MyISAM 是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。
InnoDB 不保存表的具体行数，执行select count(*) from table 时需要全表扫描。而 MyISAM 用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快；
InnoDB 最小的锁粒度是行锁，MyISAM 最小的锁粒度是表锁。一个更新语句会锁住整张表，导致其他查询和更新都会被阻塞，因此并发访问受限。这也是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一；
InnoDB 缓存索引还缓存真实数据，对内存要求搞.MYISAM只缓存索引，不缓存真实数据
```

```
MyISAM表会把自增主键的最大ID 记录到数据文件中，重启MySQL自增主键的最大ID也不会丢失；
InnoDB 表只是把自增主键的最大ID记录到内存中，所以重启数据库或对表进行OPTION操作，都会导致最大ID丢失。
```

```
char(n) 和 varchar(n) 中括号中 n 代表字符的个数，并不代表字节个数，比如 CHAR(30) 就可以存储 30 个字符。
存储时，前者不管实际存储数据的长度，直接按 char 规定的长度分配存储空间；而后者会根据实际存储的数据分配最终的存储空间
```

mysql 可以通过参数设置来开启或者关闭 表监控，锁监控。

```sql
# 是否开启标准监控
show variables like 'innodb_status_output';
# 是否开启锁监控
show variables like 'innodb_status_output_locks';
# 是否记录死锁日志
show variables like 'innodb_print_all_deadlocks';
```

# 完整卸载mysql

```sh
sudo rm /usr/local/mysql
sudo rm -rf /usr/local/mysql*
sudo rm -rf /Library/StartupItems/MySQLCOM
sudo rm -rf /Library/PreferencePanes/My*
rm -rf ~/Library/PreferencePanes/My*
sudo rm -rf /Library/Receipts/mysql*
sudo rm -rf /Library/Receipts/MySQL*
sudo rm -rf /var/db/receipts/com.mysql.*
```

执行完成之后再检查，删除完检查一下下面这些文件是否删除了，没有的话则删除掉：

```
/usr/local/Cellar 里的mysql文件
/usr/local/var 里的mysql文件
/tmp 里的mysql.sock, mysql.sock.lock, my.cnf文件
pid文件和err文件都在/usr/local/var/mysql里确保删除了
brew安装的安装包存储在/usr/local/Library/Cache/Homebrew也可以一并删除
```

最后执行 brew cleanup
