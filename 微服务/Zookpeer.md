# ZOOKPEER

# 一、安装

### 1.下载安装包

原来从目前的最新版本3.5.5开始，带有bin名称的包才是我们想要的下载可以直接使用的里面有编译后的二进制的包

### 2.解压

```
tar -zxvf sudo tar -zxvf apache-zookeeper-3.5.9-bin.tar.gz
```

### 3.复制配置文件

```
cd conf
sudo cp zoo_sample.cfg  zoo.cfg
```

### 4.创建data文件

```
/usr/local/zookeeper/apache-zookeeper-3.5.9-bin/data 
```



### 5.修改配置文件

<img src="/Users/mac/Desktop/屏幕快照 2021-03-07 22.30.27.png" style="zoom:50%;" />

### 6. 启动

```
cd bin
sudo ./zkServer.sh start
```

### 7.查看启动状态

```
sudo ./zkServer.sh status
ps -ef | grep zookeeper
```



# 二、特点

```
1、Zookeeper：一个领导者，多个跟随者
2、集群只要有半数以上（不包括半数），Zookeeper集群就能正常服务
3、全局数据一致
4、更新请求按照顺序进行
5、数据更新原子性
```



# 三、数据结构

```
Zookeeper的数据模型很类型Unix文件系统 每一个节点称为Znode,每一个Znode默认存储1MB的数据
```

<img src="/Users/mac/Desktop/屏幕快照 2021-03-13 10.10.13.png" style="zoom:50%;" />



# 四、应用

```
1、统一命名服务（存储域名，用域名来映射ip）
2、统一配置管理
	所有节点的配置信息是一致的
	服务提供真更改信息之后，Zookpeeer通知消费者（观察者模式）
3、统一集群管理
4、服务器动态上下线（如果服务器下线之后，Zookeeper通知访问其他服务器）
5、软负载均衡（让访问数最小的服务器处理最新的客户端请求）
```

# 五、参数

```
1、tickTime=2000  通信心跳数（毫秒） 客户端与服务器之间的心跳时间
2、initLimit=10  初始化时心跳次数 time=initLimit*tickTime 超过这个时间就认为客户端连接失败
3、syncLimit=5 同步通信时限   time=syncLimit*tickTime  正常运行时超过这个时间就认为客户端连接失败
4、dataDir 存储数据的位置
```

# 六、内部机制

```
1、半数机制:Zookeeper适合安装奇数台服务器

2、选举机制
	新来服务器选举时投自己一票，如果票数大于一半就当Leader，身份就固定，不因后面的机器而改变，如果票数不够一半，就将票投给编号最大的
	（1） A：1 
	（2） A:1,B:1->【未够半数，b的编号大】A:0,B:2
	（3） A:0,B:2,C:1->A:0,B:0,C:3 (超过半数，c当Leader
```

# 七、节点类型

```
1、持久型：客户端和服务器断开连接之后，节点不消失
	1）持久化目录节点
	2）持久化顺序编号节点（每一个节点有自己的编号，编号递增）
2、临时型：客户端和服务器断开连接之后，节点消失（可用于检查机器是否在线）
	1）临时目录节点
	2）临时顺序编号节点（每一个节点有自己的编号，编号递增）
```

