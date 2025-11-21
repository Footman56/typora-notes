Sewers 作用是将微信公众号里面的内容整合到rss中

# 创建数据库

```
CREATE DATABASE werss
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_general_ci;
```



# 启动服务

```
docker pull cooderl/wewe-rss:latest
```

```sh
docker network create wewe-rss
```

```sh
docker run -d \
  --restart unless-stopped \
  -p 4000:4000 \
  -e DATABASE_URL='mysql://huochai:asdf1234@123.57.235.173:3306/werss?schema=public&connect_timeout=30&pool_timeout=30&socket_timeout=30' \
  -e DATABASE_TYPE=mysql \
  -e AUTH_CODE=123567 \
  -e MAX_REQUEST_PER_MINUTE=2 \
  --network wewe-rss \
  --name wewe-rss cooderl/wewe-rss
```

# 登陆

浏览器输入 `http:\\123.57.235.173:4000`

# 输入配置的AUTH_CODE



# 扫码登陆阅读账户



# 添加阅读链接

获取公众号的一个文章就可以，之后就能获取所有文章啦

![image-20240526162850303](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202405261628614.png)

# 制作rss 链接

![image-20240526162947087](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202405261629146.png)