1. 卸载旧版本

   ```sh
   sudo yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-selinux \
                     docker-engine-selinux \
                     docker-engine
   ```

   

2. 使用国内源

   ```sh
   sudo yum-config-manager \
       --add-repo \
       https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
       
       
    sudo sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo   
   ```

3. 使用yum 安装

   ```sh
   sudo yum install -y yum-utils
   ```

4. 安装docker

   ```shell
   sudo yum install docker-ce docker-ce-cli containerd.io
   ```

5. 启动docker

   ```sh
   sudo systemctl enable docker
   sudo systemctl start docker
   ```

6. 镜像加速

   先执行 `systemctl cat docker | grep '\-\-registry\-mirror'` 如果没有输出的话，

   需要新建文件/etc/docker/daemon.json

   并且在json 文件中输入

   ```json
   {  
    "proxies": {
       "http-proxy": "https://q1m1ila0.mirror.aliyuncs.com"",
       "no-proxy": "*.test.example.com,.example.org,127.0.0.0/8"
   }
   }
   ```
   
   重启
   
   ```sh
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```


## mac

Mac 下安装docker

```sh
brew install --cask docker
```

或者手动下载

镜像加速

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410192249960.png" alt="image-20220713205224330" style="zoom:50%;" />

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410192249972.png" alt="image-20220713205353128" style="zoom:50%;" />

需要将docker指定配置成全局命令

```sh
vim .zshrc

# 输入指令
export DOCKER_PATH="/Applications/Docker.app/Contents/Resources/bin"
export PATH="$PATH:$DOCKER_PATH"


# 刷新环境
source ~/.zshrc

# 再次查看版本检查下载情况
docker --version 
```



# 二、阿里云镜像加速

## 1、登录阿里云

选择镜像加速器

## 2、配置

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://q1m1ila0.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

app:

通过client-ui 可以管理容器

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410192248006.png" alt="image-20220713205612293" style="zoom:50%;" />