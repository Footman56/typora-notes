# 镜像指令

```bash
docker -- help 帮助命令
docker info 查看dockre 基本信息
docker images 查看本地安装的镜像
sudo docker image history hello-world 查看hello-world镜像的历史
sudo docker image prune  -a 移除所有未使用的镜像
sudo docker search xxx   在github中查找所有镜像
docker search --filter=stars=3  busybox
	
sudo docker image pull  NAME[:TAG]   默认下载最新版
sudo docker image rm -f IMAGE 强制删除镜像  
docker rmi -f IMAGE [ $(docker images -aq) ]  删除所有镜像
```



# 容器指令

```bash
要有镜像才能创建容器

docker kill CONTAINER 杀死一个容器
docker start CONTAINER 启动一个容器
docker run [参数] image  新建容器并启动
	--name Name   指定容器的名称
	-d 后台运行
	-it 使用交互方式运行，进入容器查看内容    只有容器实现时候指定了交互方式才能使用 -it 
	-p xx:xx  将主机端口与容器端口映射起来

docker logs CONTAINER查看一个容器的日志
```

