# mac 下载

```shell
brew install elastic/tap/elasticsearch-full
```

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202302022327843.png" alt="image-20230202232726568" style="zoom:50%;" />

重要文件

```
Data:    /usr/local/var/lib/elasticsearch/elasticsearch_peilizhi/
Logs:    /usr/local/var/log/elasticsearch/elasticsearch_peilizhi.log
Plugins: /usr/local/var/elasticsearch/plugins/
Config:  /usr/local/etc/elasticsearch/
```



启动

```sh
brew services start  elastic/tap/elasticsearch-full
```

启动成功后访问 ：http://127.0.0.1:9200/

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202302022328804.png" alt="image-20230202232824772" style="zoom:50%;" />

**注：不要在使用 brew install elasticsearch ,这个废弃了**

或者

从官网下载对应的版本，进入到 bin目录下，通过  命令来启动，

```
./bin/elsaticsearch 
```

```
# 后台运行
./bin/elsaticsearch  & 
```

![image-20230510181203934](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202305101812283.png)





# linux上下载安装

1. 下载压缩包

   ```sh
   wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.2-linux-x86_64.tar.gz
   ```

2. 解压

   ```sh
   tar -zxvf elasticsearch-7.6.2-linux-x86_64.tar.gz
   ```

3. 重命名

   ```sh
   mv elasticsearch-7.6.2/ elasticsearch
   ```

4. 启动，要到bin 目录下执行./elasticsearch 

   ```sh
   ./elasticsearch
   
   # 后台启动
   ./elasticsearch -d 
   ```

5. 访问，要使用公网端口

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202302030123867.png" alt="image-20230203012326834" style="zoom:50%;" />

   # 问题集锦

   1. 内存不足

   ![image-20230203000315735](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202302030003776.png)
   
   

```sh
cd /home/huochai/elasticsearch/config
vim jvm.options


-Xms256m
-Xmx256m
```



2. OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release. jvm  版本太低了，要下载高版本（可以忽略）

3. 启动的时候提示已杀死 

   ![image-20230203003630622](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202302030036655.png)

   还是内存不足

   ``` sh
   # 查看内存分配
   free -h 
   ```

4. ![image-20230203005924780](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202302030059820.png)

```sh
sudo vim /etc/sysctl.conf
vm.max_map_count = 262144
# 配置生效
sudo sysctl -p
```

```sh

sudo vim /etc/security/limits.conf
* soft nofile 65535
* hard nofile 65535
* soft nproc 2048
* hard nproc 4096
```

```sh
# max number of threads 
vim /etc/security/limits.conf
huochai   -  nproc  65535
huochai 是我启动es 的用户 
这个配置是在下一个会话中生效，需要先退出账号
```

在elasticsearch.yaml 中修改配置

```yaml
# 集群的名称
cluster.name: huochai
# 节点的名称
node.name: node-1
# 内网端口（服务器的内网地址）
network.host: 172.24.12.186
# http 请求端口
http.port: 9200
# 集群中的节点
cluster.initial_master_nodes: ["node-1"]
```



 启动的时候不能用root身份启动



启动时提示：

You must address the points described in the following [1] lines before starting Elasticsearch.







# 安装elasticsearch-head 

直接在google浏览器的应用商店里面安装

![image-20230204113948937](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202302041139219.png)

# 安装kibana

![image-20230205105720025](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202302051057219.png)

默认端口是 localhost:5601

从官网下载安装包，

进入bin 目录下，通过指令启动

```
./bin/kinaba &
```
