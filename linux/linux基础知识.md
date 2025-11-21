

#  一、目录

**在linux中一切皆文件**

```shell
/ 			一切的起源，根目录
/bin 		存放常用的指令
/sbin 	存放系统管理员使用的系统管理程序
/home 	存放普通用户的主目录，一个用户对应一个目录
/root   root用户相应的目录
/boot   存放的是启动linux时使用的核心文件 
/lib		系统开机所需要的动态链接共享库
/etc  	配置文件和子目录
/usr   	用户的应用程序和文件
/dev    硬件设备
/media  linux系统自动识别一些设备，例如u盘
/opt 		给主机额外安装软件的所在目录
/usr/local  主机安装额外软件的目录
/var 		目录中存放不断扩充的动心，将经常要修改的目录放在这里，通常放日志
/selinux 安全子系统


如果末尾带 “/”表示这是一个目录
```

# 二、指令

命令格式

```bash
command parameters（命令 参数）


单个参数：ls -a（a 是英文 all 的缩写，表示“全部”）
多个参数：ls -al（全部文件 + 列表形式展示）
单个长参数：ls --all
多个长参数：ls --reverse --all
长短混合参数：ls --all -l

长参数通过 “--” 来区别，参数是单词。
短参数 ”-“ 是单个字母


短参数：command -p 10（例如：ssh root@121.42.11.34 -p 22）
长参数：command --paramters=10（例如：ssh root@121.42.11.34 --port=22）
长参数通过“=”来获取参数
```

```java
reboot 重启
shutdown 
  shutdown -h now 立即关机
  shutdown -h 1 1分钟后关机
  shutdown -r now 立即重启
syn  把内存中的数据写入到磁盘

在关机或者重启的时候先执行 syn 指令
```

## 0、将一个指令运行的结果作为另一个指令的参数

+ ``
+ $()
+ 

## 1、vi&& vim 

本质就是文本编译器

### a、正常模式

```
进入之后就是正常模式，可以使用快捷键

yy 复制当前行
dd  删除当前行
p 粘贴
/查找的内容 +出车
G进入最后一行
gg 回到开头
u 撤销 每按一次回退一步操作
```



### b、编辑模式

```
从正常模式输入 i 进入编辑模式
按ESC进入 进入正常模式
```



### c、命令行模式

``` 
在正常模式输入: 进入命令行模式

：wq 保存退出（做了修改）
：q  直接退出（不做修改）
：q! 强制退出，不保存
：set nu 显示行号
：set nonu 不显示行号
```

## 2、指令级别

```
0：关机
1：单用户（用于找回密码）
2：多用户无网络
3：多用户有网络（常用）
4：系统未使用 保留给用户
5：图形界面
6：重启
在/etc/inittab指定运行级别

切换到指定运行级别  init [0123456]



如何找回丢失的root密码？
进入单用户模式（1）,修改root密码
```

## 3、帮助指令

```
man   指令
help	指令
```

## 4、文件目录

```shell
pwd   显示当前目录的绝对路径

ls [选项] 目录或者文件 :展示目录中的所有文件
	-a  显示隐藏文件
	-l  列表显示
	-h 展示符合人阅读习惯的

绝对路径  从根目录开始计算
相对路径  从当前目录计算
cd 目录							:切换目录
  cd ~ 或者 cd 进入家目录
  cd .. 回到上一级目录


mkdir [选项] 目录  创建目录
	-p	创建多级目录

rmdir [选项] 目录  删除空目录
rm -rf 目录       删除非空目录

touch 文件名称     创建空文件
cp [选项] source target
	-r   递归复制整个文件
\cp -r ss bb 强制覆盖      结果：bb目录中会有ss

mv 操作文件时是移动并且重命名。
目标目录与原目录一致，指定了新文件名，效果就是仅仅重命名。
mv  /home/ffxhd/a.txt   /home/ffxhd/b.txt   

目标目录与原目录不一致，没有指定新文件名，效果就是仅仅移动
mv  /home/ffxhd/a.txt   /home/ffxhd/test/ 

目标目录与原目录一致, 指定了新文件名，效果就是：移动 + 重命名。
mv  /home/ffxhd/a.txt   /home/ffxhd/test/c.txt


批量移动文件和文件夹：
mv  /home/ffxhd/testThinkPHP5/tp5/*  /home/ffxhd/testThinkPHP5
注意：需要先执行显示隐藏文件命令，否则，隐藏文件以及隐藏文件夹不会被移动到新目录。



```

## 5、读写

```
less 加载文件（看到哪里加载到哪里）
more读取文件（只读）

> 输出重定向（会覆盖原文的内容）
>> 追加（在文件末尾追加）
ls -l > a.txt 将列表内容覆盖写到a.txt
ls -l >> a.txt 将列表内容追加写到a.txt'
cat xxx > xxx 将文件内容写入到另一文件，没有就创建
echo 内容 >> a.txt  将内容追加写入到a.txt


echo [选项] 输出内容   输出命令到控制台

head 显示文件开头部分
head 默认显示前10行
head -n X 显示指定

tail 显示文件尾部的内容
tail 文件   查看文件后10行内容
tail -n 5 文件   查看文件后5行内容
tail -f 文件     实时显示文件内容（随更新而变化）

```

## 6、链接

```  
ln 软连接（符号连接）  类似快捷方式，主要存放连接其他文件的路径，源文件变化的时候，链接文件同时变化。
删除源文件的时候，链接文件变成不可见。类型是 l 开头

ln -s [原文件或者目录] [软链接]  
但是pwd显示的还是当前目录 
使用rm -rf 连接



history 查看执行的历史指令
history X 查看最近X条历史指令
!x 执行第X条指令，配合history指令查看指令编号
```

## 7、时间日期

``` 
date 显示当前时间
date+%Y 显示当前年份
date "+%Y:%m  "  格式化输出（+不能少）

date -s "时间格式" 设置系统时间
cal 显示日历信息
```

## 8、查找

```
find [搜索范围] 选项

find / -name 文件名（包含* ）
find / -user 用户   根据用户查找
find / -size +20M  查找目录下大于20M
find / -name *.txt 查找txt格式的文件
find /mysql/backup/files -mtime +7 -name "*.gz"  在XX下查找七天以前的文件

find / -name 文件名 2>/dev/null 不输出错误文件

locate 
需要先创建locate数据库  执行 sudo locate指令
locate 文件名

grep [选项] 查找内容 源文件
	-n 显示匹配行及行号
	-i 忽略字母大小写
管道 “|” 将前一个命令的处理结果交给后面的指令处理
```

## 9、压缩解压缩

```
gzip 文件名  用于将文件压缩成 .gz格式，不保留源文件
gunzip 文件名.gz   将.gz文件解压缩


需要先install zip 或者uzip
zip 压缩文件 
unzip 用于解压缩
zip [选项]  文件名或者目录.zip  要压缩的目录或者文件名
	-r 递归压缩整个目录
zip -r huochai.zip /home/huochai/(要压缩的目录)
	
unzip [选项] 解压目录
	-d 自定解压后的文件存放目录
unzip -d 存放目录 压缩文件名



tar 
tar 打包指令  压缩后的格式是 tar.gz
tar [选项] XX.tar.gz 打包的内容
	-c  产生tar打包文件
	-v  显示详细信息
	-f  指定压缩后的文件名
	-z 打包时并压缩
	-x   解压

tar -zcvf a.tar.gz a.txt b.txt
	a.tar.gz 压缩后的文件名 
	a.txt b.txt 要打包压缩的文件
	
tar -zcvf myhome.tar.gz /home 对整个目录进行压缩
执行tar 的时候文件名不能带有“:”


tar -zxvf a.tar.gz 解压到当前目录
tar -zxvf a.tar.gz -C 待解压的文件目录
```

## 10、ifconfig

ifconfig 是一个用来查看、配置、启用或禁用网络接口的工具，这个工具极为常用的

```
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	options=400<CHANNEL_IO>
	ether 88:66:5a:3a:f9:fa
	inet6 fe80::1463:90fe:7358:6746%en0 prefixlen 64 secured scopeid 0x6
	inet 10.109.20.155 netmask 0xfffffc00 broadcast 10.109.23.255
	nd6 options=201<PERFORMNUD,DAD>
	media: autoselect
	status: active
```

en0表示第一块网卡，inet ：网卡的IP地址，netmask：网络掩码，broadcast：广播地址，status ：活跃， HWaddr 表示网卡的物理地址，可以看到目前这个网卡的物理地址(MAC地址）

```
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 16384
	options=1203<RXCSUM,TXCSUM,TXSTATUS,SW_TIMESTAMP>
	inet 127.0.0.1 netmask 0xff000000
	inet6 ::1 prefixlen 128
	inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1
	nd6 options=201<PERFORMNUD,DAD>
```

lo 是表示主机的回坏地址，这个一般是用来测试一个网络程序，但又不想让局域网或外网的用户能够查看，只能在此台主机上运行和查看所用的网络接口。

# 三、用户管理

````
useradd  XXX   此时创建用户，并创建一个同名的组
useradd -d 当前目录 用户名   给用户指定相应的目录
useradd -g 组名 用户名
passwd   xxx   指定密码

/etc/group  组配置文件
每行含义：组名：口令：组标识符：组内用户列表

/etc/passwd  用户配置文件
每行含义：用户名：口令：用户标识符：组标识号：注解性描述：主目录：登录shell
/etc/shadow  口令的配置文件
````

创建个人账号，并且通过ssh 公钥登录（不需要输入密码）

1. [服务器]先创建用户 ，设置密码

   ```shell
   useradd huochai
   passwd  huochai XXXXX
   ```

2. [服务器]修改ssh 配置，允许ssh 远程登录

   ```sh
   sudo vim /etc/ssh/sshd_config
   ```

   修改的字段为：

   ```
   PermitRootLogin yes 
   PubkeyAuthentication yes
   
   #GSSAPIAuthentication yes
   #GSSAPICleanupCredentials no
   
   UsePAM yes
   ```

3. [服务器]重启 ssh 服务

   ```sh
   systemctl restart sshd
   ```

4. [服务器]查看当前用户的家目录是否具有权限，没有就需要赋予权限

   ```
   ll -ld /home/[your-username]
   chmod 0700 /home/[your-username]
   ```

5. [服务器] 查看用户家目录下是否有.ssh文件，是否有权限.没有的话需要创建，并且赋予权限

   ```
   mkdir .ssh
   chmod 0700 .ssh
   ```

6. [服务器]  查看.ssh 文件下是否有authorized_keys ，没有就需要创建，并且赋予权限

   ```
   touch /home/[username]/.ssh/authorized_keys
   chmod 0600 /home/[username]/.ssh/authorized_keys
   ```

7. [服务器] 手动将要登录的主机的公钥放在authorized_keys 

8. 客户端登录

   ```
   ssh huochai@47.93.247.254
   ```

   客户端生成ssh 公钥

   ```
   ssh-keygen -t rsa -C "youremail@domain.com"
   ```

   之后会生成两个文件

   - `id_rsa`：生成的私钥，保留在电脑即可。
   - `id_rsa.pub`：生成的公钥，打开后，复制内容，后文部署到服务器上。

# 四、组管理

```
每一个文件都对应一个所有者，所有组
文件的创建者就是文件的所有者，创建者的组就是文件的组
groupadd 添加组
chgrp 组名 文件名   :改变文件所在的组
chgrp -R 组名 目录  :递归改变目录所在组

usermod -g 组名 用户名    :改变用户所在组
usermod -d 目录名 用户名  :改变用户初始登录目录
```

 ```
-rw-rw-r-- 1 huochai huochai   929311 3月   9 22:29 zipkin.jar
	第一个字符 -文件 d目录 l:软连接 b：块文件（硬盘） c:字符设备（键盘，鼠标）
	2-4字符：文件创建者的权限
	5-7字符：同组内其他用户权限
	8-10字符：其他用户权限
	文件硬链接的个数（1），目录为其下子目录数（包含 .  .. 目录）
	用户
	所在组
	文件大小 目录固定为4096（目录的分配空间）
	最后修改时间
	
	rwx 作用到文件
	r 表示可读
	w 可写，但是删除文件需要对目录有写的权限
	x 可以执行
	rwx 作用到文件
	r 可以读取 
	w 可写，可以修改目录内创建、删除、重命名目录
	x 可以进入该目录
	
chmod 修改文件的权限
	+ 添加权限
	- 删除权限
	= 修改权限
 u 所有者 g:所有组 o 其他人 a:所有人
 chmod u=rwx,g=wx
 			 u+w
chown 修改文件所有者
chown 用户 文件 更改文件的所有者
chown -R 用户 目录 将目录及目录下子目录的所有者改成用户
 ```

#  五、定时任务调度

```
crontab 定时任务调度
任务调度：系统在某个时间内执行特定的命令或者程序
	系统工作
	个人用户工作
crontab [选项]
	-e 编辑定时任务
	-l 查询contab
	-r 终止任务调度
service crond restart 重启任务调度
```

## 1、编写脚本

```
如果任务很简单就不需要编写脚本
	直接使用 crontab -e 编辑
	文件中输入 cron表达式  指令 
 cron 表达式（5个占位符）
 		1：0-59 一小时中的分钟
 		2：0-23 一天中的小时
 		3：1-31 一月中的天
 		4：1-12 一年当中的月
 		5：0-7  星期（0，7都表示星期日）
 	
 		* 任意时间
 		，不连续时间
 		- 连续范围时间
 		*/n 每隔多久执行一次
    
.sh 中写要执行的指令
给文件执行权限（注意你当前用户）
```

## 2、配置定时任务

```
crontab -e
格式： cron表示式 脚本的位置
保存退出时显示 installing new crontab
也可以使用 crontab -l展示定时任务
```

# 六、分区

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-11 10.52.22.png" style="zoom:50%;" />

```markdown
***挂载就是将硬盘的分区与系统的文件关联起来***

Linux 硬盘分为 IDE 硬盘和SCSI硬盘 目前基本SCSI硬盘（串行）
表示 "hdx~"   
	hd表示设备类型（IDE硬盘）  sd为SCSI硬盘
	x 为盘号 a:基本盘 b:基本从属盘 c:辅助主盘 d:辅助从属盘
	~ 分区  1-4 主分区   5+逻辑分区

查看分区和挂载情况  lsblk -f 
```

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-11 10.59.25.png" style="zoom:50%;" />

```
如何增加一块硬盘
1、添加硬盘
2、分区 
3、格式化
4、挂载
5、设置可以自动挂载
```

```
磁盘情况查询
df -h  系统整体占用情况
du [选项] 目录 查询目录使用情况
	-s 指定目录占用大小汇总
	-h 含计量单位
	-a 含文件
	-maxdepth=1 子目录深度
	
du -ah --max-depth=1  /home/huochai
```

```
统计目录下有多少文件
ll 目录 |grep "^-"|wc -l

统计目录下有多少目录
ll 目录 |grep "^d"|wc -l

统计目录下（含子目录）文件个数
ls -lR /home/huochai |grep "^d" |wc -l

tree 以树状形式展示
```

```
/etc/sysconfig/network-scripts/ifcfg-eth0   网卡配置文件
```

# 七、进程管理

```
每一个进程都有一个父进程
进程有两种进程：前台与后台（守护进程）

ps 查看进程使用信息
	-a  显示当前终端所有进程
	-u 以用户的格式展示进程
	-x 显示后台进程运行的参数
	-ef 显示父进程
	
进程状态
	s:休眠
	r:运行
	n:进程的优先级比普通进程优先级低
	d:短期等待
	z:僵死进程
	t:被追踪或被停止
```

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-11 11.39.03.png" style="zoom:50%;" />

```
使用
ps -a |grep XXX 进行过滤

杀死进程
kill [选项] 进程号
	-9 强制杀死
	
杀死非法用户
ps -ef|grep sshd  查看用户登录进程   huochai@pts/0这个对应的进程号
kill -9 PID

杀死所有编辑进程
killall gedit 

强制杀死终端
ps -ef |grep bash
kill -9 PID

动态监控进程
top [选项]
	-d :指定每隔几秒更新
	-i:不显示闲置或者僵死进程
	-p：
	

top 类似任务管理器
输入 u  后续输出用户名 查看与用户相关
q退出

交互执行
N PID 排序
M 内存
p cpu 排序


netstat [选项]
 -an 
 -p 显示哪个进程在调用
```

# 八、服务

```
服务本质就是进程，但是运行在后台，监听某个端口，等待其他进程请求
centos7 以后要使用Systemctl

telnet ip:port  检查服务进程是否启动

查看哪些服务
setup 
在 /etc/init.d查看服务

开机-》BIOS-》/boot-》init进程-》运行级别-》运行相应级别的服务

chkconfig  显示服务列表，在各个级别下的运行情况

chkconfig 服务名 --list 查询某个服务的运行状况
chkconfig --level 5 服务 on/off   控制服务在运行级别下是否自启动
```

#  九、rpm yum

```
rpm 互联网下载包的打包及安装工具

查询已经安装的rpm 列表 
rpm -qa
python-chardet-2.2.1-3.el7.noarch
后缀中noarch 表示通用
rpm -q 软件包 查询是否安装软件包
rpm -ql 软件包 查询软件安装的位置
rpm -qf 文件  查询这个文件属于哪个软件包
rpm -e 软件包   卸载软件包
rpm -e -nodeps 软件包 强制卸载软件包

rpm -ivh 软件包
	-i :安装
	-v :提示
	-h :进度条
	

yum:自动下载安装软件包
yum list    yum服务器上安装列表
yum install XXX 下载安装

```



# 十一、centos8 已经不在维护了

在下载的时候如此出现

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221126025903729.png" alt="image-20221126025903729" style="zoom:50%;" /> 

需要执行

```
sudo sed -i -e "s|mirrorlist=|#mirrorlist=|g" /etc/yum.repos.d/CentOS-*
sudo sed -i -e "s|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g" /etc/yum.repos.d/CentOS-*
```



# 查看运行的java 项目

```
ps -ef|grep java|awk '{print $NF}'
```

# 项目启动失败

```
1、项目代码本身问题
2、项目需要的容量太大，服务器能够分配的容器不够
3、
```



# curl  下载文件

```sh
curl -O 'http://' 
```

