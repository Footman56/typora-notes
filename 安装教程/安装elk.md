# mac 安装（brew）

一、 docker 命令 获取对应镜像

```
docker pull elasticsearch:7.17.6
docker pull kibana:7.17.6
```

二、获取analysis-ik 分词插件

https://release.infinilabs.com/analysis-ik/stable/

版本一定要与elastic版本一致。

安装在：/Users/peilizhi/elastic目录下

三、 创建docker-compose.yml

```yaml
version: '3.8'

services:
  es01:
    image: elasticsearch:7.17.6
    container_name: es01
    restart: always
    privileged: true
    ports:
      - "9200:9200"
    environment:
      # 节点名称
      - node.name=es01
      # 集群名称
      - cluster.name=es-docker-cluster
      # 其它节点
      - discovery.seed_hosts=es02
      # 配置一个或多个节点, 用于在es集群初始化时选举主节点master
      - cluster.initial_master_nodes=es01,es02
      # 锁定内存地址
      - bootstrap.memory_lock=true
      # 设置java堆内存大小
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      # es data路径
      - "/home/huochai/my_elastic/es01/data:/usr/share/elasticsearch/data"
      # es log路径
      - "/home/huochai/my_elastic/es01/logs:/usr/share/elasticsearch/logs"
      # 集成分词插件
      - "/home/huochai/my_elastic/plugins/analysis-ik-7.17.6:/usr/share/elasticsearch/plugins/elasticsearch-analysis-ik-7.17.6"
    networks:
      testNet:
  es02:
    image: elasticsearch:7.17.6
    container_name: es02
    restart: always
    privileged: true
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01
      - cluster.initial_master_nodes=es01,es02
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
       # es data路径
      - "/home/huochai/my_elastic/es02/data:/usr/share/elasticsearch/data"
      # es log路径
      - "/home/huochai/my_elastic/es02/logs:/usr/share/elasticsearch/logs"
      # 集成分词插件
      - "/home/huochai/my_elastic/plugins/analysis-ik-7.17.6:/usr/share/elasticsearch/plugins/elasticsearch-analysis-ik-7.17.6"
    networks:
      testNet:
  kibana:
    image: kibana:7.17.6
    container_name: kibana
    restart: always
    ports:
      - "5601:5601"
    environment:
      # es访问地址
      ELASTICSEARCH_HOSTS: '["http://es01:9200","http://es02:9200"]'
      # kibana语言配置：en、zh-CN、ja-JP
      I18N_LOCALE: "zh-CN"
    networks:
      testNet:
    depends_on:
      - es01
      - es02
networks:
  testNet:
```

四、启动

需要在docker-compose.yml 目录下执行

```
docker-compose up -d
```

