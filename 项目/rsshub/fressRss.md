1. 拉取rsshub 镜像

   ```sh
   docker pull diygod/rsshub
   ```

2. 启动容器

   ```sh
   docker run -d --name rsshub -p 1200:1200 diygod/rsshub
   ```

3. 访问

   输入地址 ip:1200

4. 安全组开放1200 端口

5. 搭建FreshRss

   a. 创建容器

   ```
   docker network create freshrss-network
   ```

   b. 创建文件映射

   只在本地设置文件的话，就会映射到docker 容器内
   
   ```
   docker volume create freshrss-data
   docker volume create freshrss-extensions
   ```

   c. 启动容器
   
   ```sh
   docker run -d --restart unless-stopped --log-opt max-size=10m \
     -v freshrss-data:/var/www/FreshRSS/data \
     -v freshrss-extensions:/var/www/FreshRSS/extensions \
     -e 'CRON_MIN=4,34' \
     -e TZ=Asia/Shanghai \
     -p 8080:80 \
     --net freshrss-network \
     --label traefik.port=80 \
     --label traefik.frontend.rule='Host:freshrss.example.net' \
     --label traefik.frontend.headers.forceSTSHeader=true \
     --label traefik.frontend.headers.STSSeconds=31536000 \
     --name freshrss freshrss/freshrss
   ```

   d. 访问主页面
   
   ip:8080

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202405232343538.png" alt="image-20240523234332411" style="zoom:50%;" />

6. 配置页面

   安装步骤执行就行

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202405232344479.png" alt="image-20240523234424429" style="zoom:50%;" />

7. 安装数据库

   ```mysql
   CREATE DATABASE rsshub
     CHARACTER SET utf8mb4
     COLLATE utf8mb4_general_ci;
   ```

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202405240025976.png" alt="image-20240524002508924" style="zoom:50%;" />

8. 认证

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202405241057591.png" alt="image-20240524105654841" style="zoom:50%;" />



9 .添加订阅源

订阅源：https://docs.rsshub.app/guide/

目前使用公共的地址  rsshub.feeded.xyz

公共的地址不够稳定，使用自定义的rsshub  : http://123.57.235.173:1200

```
http://123.57.235.173:1200/36kr/hot-list/24

http://123.57.235.173:1200/cntv/TOPC1451528971114112

http://123.57.235.173:1200/cntv/TOPC1451551777876756

http://123.57.235.173:1200/yicai/news/jinrong


http://123.57.235.173:1200/who/news/Zh

http://123.57.235.173:1200/who/news-room/feature-stories/Zh

http://123.57.235.173:1200/gov/zhengce/zuixin

http://123.57.235.173:1200/hellogithub/article/hot


http://123.57.235.173:1200/hackernews/threads/comments_list/dang
http://123.57.235.173:1200/hackernews/index/sources/tristan-greene


https://cointelegraph.com/rss/

https://cointelegraph.com/rss/tag/blockchain

https://cointelegraph.com/editors_pick_rss

http://123.57.235.173:1200/newyorker/latest


http://123.57.235.173:1200/economist/:china

http://123.57.235.173:1200/caixin/livelihood

http://123.57.235.173:1200/aeon/category/社会


http://123.57.235.173:1200/gamersky/news/today

http://123.57.235.173:1200/hellogithub/volume


```

10. 安装插件

    需要将插件放入到  `/var/lib/docker/volumes/freshrss-extensions/_data`

    从仓库中clone下载过来，放入到这个目录就可以啦。

    之后到页面上打开这个就可以



11. 为非rsshub 中网页设置 rss

    a. 先安装google 插件 RSSHub Radar

    b. 先检查心仪网页是否有rss 标志

    <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202405252354365.png" alt="image-20240525235434141" style="zoom:50%;" />

​		c. 点开页面，添加的freshness 

​			![image-20240525235535999](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202405252355071.png) 







为了便于阅读，在mac上下载了reeder ,之后配置rss 链接即可，这个链接是

![image-20240526182044993](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202405261820046.png)

遇到问题不要慌，首先数据库是存在linux机器里面的，不要担心数据丢失。

只有重启docker容器就可以了



配置定时任务执行数据

### 步骤1：安装 `cron` 和 Docker

首先，请确保在你的 CentOS 系统上安装了 Docker 和 `cron`。如果你尚未安装 `cron`，可以使用以下命令来安装：

```bash
sudo yum install cronie  
```

确保 `crond` 服务正在运行：

```bash
sudo systemctl start crond  
sudo systemctl enable crond  
```

### 步骤2：创建定时任务

1. **编辑 Cron 表**：
   使用下面的命令编辑当前用户的 cron 表：

   ```bash
   crontab -e  
   ```

2. **添加定时任务**：
   在打开的编辑器中，添加以下行以设置每天早上 8 点的定时任务：

   ```bash
   * 8 * * * docker exec freshrss php /var/www/FreshRSS/app/actualize_script.php > /tmp/FreshRSS.log 2>&1
   ```

   这条 cron 表达式的意思是每天 8 点执行 `docker exec` 命令，进入 `freshrss` 容器并运行 `php` 脚本来刷新 RSS 源。

3. **保存并退出**：
   保存文件并退出编辑器，cron 会自动安装新的定时任务。

### 步骤3：检查 Cron 任务

你可以通过以下命令来查看已设置的 cron 任务以确认其是否正确添加：

```bash
crontab -l  
```
