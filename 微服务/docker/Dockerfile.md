# 十一、Dockerfile

```
用于构造镜像 
1、先创建DockerFile文件
2、编写指令 (关键字全大写 每一个指令都会创建一个新的镜像层)
3、sudo docker build -f 文件路径 -t  镜像名：TAG .

```

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-17 18.16.35.png" style="zoom:50%;" />

```
# 基础
FROM centos
# 挂载
VOLUME ["huochai","xiaoli"]
# 指令
CMD echo "my image"
CMD /bin/bash
```

```
FROM       							# 基础镜像
MAINTAINER 							# 维护者 姓名+邮箱
RUN 										# 镜像构造时需要的命令,在镜像中执行
ADD 										# 添加内容，如果是压缩包的话会自动解压
WORKDIR 								# 镜像的工作目录
VOLUME 									# 挂载的目录
EXPOSE 									# 暴露的端口号
CMD     								# 指定容器启动时执行的命令 只有最后一个生效
ENTRYPOINT 							# 指定容器启动时执行的命令 追加
ONBUILD 								# 当构建一个被继承的Dockefile会触发
COPY 										# 将文件copy到镜像
ENV  										# 环境变量

```

<img src="/Users/mac/Pictures/截图/src=http___seo-1255598498.file.myqcloud.com_full_4455082961bc017cb7c87a3463bb326e021013ce.jpg&refer=http___seo-1255598498.file.myqcloud.jpeg" style="zoom:50%;" />

```dockerfile
官方推荐命令 Dockerfile ，之后同文件夹下可以不指定 -f

FROM centos
MAINTAINER huochai<17633150922@163.com>

# 指定系统变量
ENV Mypath /usr/local
# 进入的目录
WORKDIR $Mypath
# 在镜像中 执行
RUN  yum -y  install vim
RUN  yum -y  install net-tools

EXPOSE 8077

CMD echo "创建mycentos"
CMD /bin/bash

CMD 命令是容器创建的时候执行的，可以在dokcer run xxx  的时候输入完整的命令作为执行命令
ENTRYPOINT 是在追加的形式
```

