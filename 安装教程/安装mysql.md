# 卸载mariadb

1. 检查是否安装

   ```sh
   yum list installed | grep mariadb
   ```

2. 安装后卸载

   ```sh
   yum -y remove mariadb*
   ```

3. 再次检查

# 卸载mysql

1. 检查是否安装

   ```sh
   rpm -qa|grep -i mysql
   ```

   

# 安装mysql

1. 创建目录

   ```sh
   cd /usr/local
   mkdir mysql
   cd mysql
   ```

2. 下载

   ```sh
   sudo wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.30-linux-glibc2.12-x86_64.tar.xz
   ```

3. 解压

   ```sh
   sudo tar xvJf mysql-8.0.30-linux-glibc2.12-x86_64.tar.xz
   ```

4. 重命名文件夹并删除压缩包

   ```sh
   sudo mv mysql-8.0.30-linux-glibc2.12-x86_64 mysql-8.0
   
   sudo rm -rf mysql-8.0.30-linux-glibc2.12-x86_64
   ```

   

# 创建用户组以及用户和密码

```sh
sudo groupadd mysql
sudo useradd -g mysql mysql
```

# 授权用户

```sh
sudo chown -R mysql.mysql /usr/local/mysql/mysql-8.0
```

# 初始化获取临时密码

```sh
./mysqld --user=mysql --basedir=/usr/local/mysql/mysql-8.0 --datadir=/usr/local/mysql/mysql-8.0/data/ --initialize
```

执行的时候报错

./mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory

需要先安装 libaio

```sh
sudo yum install -y libaio
```

执行成功后会生成默认密码

![image-20240524000509451](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202405240005496.png)

# 编辑配置文件

没有就创建

```sh
vi /etc/my.cnf
```

```
[mysql]
#MySQL 提示符配置
 
#用户名@主机名+数据库名
#prompt="\\u@\\h [\\d]>"
 
#用户名@主机名+mysql版本号+数据库名
prompt=\\u@\\h \\v [\\d]>\\_
 
#用户名@主机名+当前时间+mysql版本号+数据库名
#prompt="(\\u@\\h) \\R:\\m:\\s \\v [\\d] \n>"
 
[mysqld]
#mysql安装根目录
basedir = /usr/local/mysql/mysql-8.0/
 
#mysql数据文件所在位置
datadir = /usr/local/mysql/mysql-8.0/data/
 
#设置socke文件所在目录
socket = /tmp/mysql.sock
 
#数据库默认字符集, 主流字符集支持一些特殊表情符号（特殊表情符占用4个字节）
character-set-server = utf8mb4
 
#数据库字符集对应一些排序等规则，注意要和character-set-server对应
collation-server = utf8mb4_general_ci
 
#设置client连接mysql时的字符集,防止乱码
init_connect='SET NAMES utf8mb4'
```

# 添加mysql 服务到系统

```sh
cd /usr/local/mysql/mysql-8.0

cp -a ./support-files/mysql.server /etc/init.d/mysql

chmod +x /etc/init.d/mysql

chkconfig --add mysql

```

#  启动mysql

```sh
service mysql start

# 查看mysql 状态
service mysql status
```

![image-20240524001226946](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202405240012005.png)

# 添加到系统指令

之后在任意目录都可以使用mysql 指令

```sh
ln -s /usr/local/mysql/mysql-8.0/bin/mysql /usr/bin
```

# 登陆

密码是之前生成的

```sh
mysql -uroot -p
```

# 修改root密码

```sh
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'asdf1234';
flush privileges;
```

# 创建用户

```sh
create user  'huochai'@'%' IDENTIFIED BY 'asdf1234';
```

# 用户授权

赋予所有权限

```sh
GRANT all ON *.* TO 'huochai'@'%';
```

# 设置开机启动

如果 第3，4，5 项都开启的话就是开机启动

```sql
chkconfig --list

# 设置开机启动
systemctl enable mysqld.serviceaa
```

# 查看端口号

```sh
firewall-cmd --zone=public --query-port=3306/tcp
```

