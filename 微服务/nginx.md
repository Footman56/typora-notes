---
typora-copy-images-to: ../static/images
---

# 一、安装Nginx

## 1、源安装

```
安装基础源
yum -y install gcc gcc-c++ autoconf pcre-devel make automake
yum -y install wget httpd-tools vim
```

## 2、基于Yum的方式

```
查看yum是否存在
yum list | grep nginx

安装
1、配置
vim /etc/yum.repos.d/nginx.repo
内容是
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1


2、yum
yum install nginx 

3、查看版本
nginx -v
```

## 3、brew 安装

```
brew install nginx

# 查看nginx 配置
通过brew info nginx
```

## 指令

```
nginx -s stop       快速关闭Nginx，可能不保存相关信息，并迅速终止web服务。
nginx -s quit       平稳关闭Nginx，保存相关信息，有安排的结束web服务。
nginx -s reload     因改变了Nginx相关配置，需要重新加载配置而重载。
nginx -s reopen     重新打开日志文件。
nginx -c filename   为 Nginx 指定一个配置文件，来代替缺省的。
nginx -t            不运行，而仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件。
nginx -v            显示 nginx 的版本。
nginx -V            显示 nginx 的版本，编译器版本和配置参数。
```



# 二、配置

## nginx.conf

```
#运行用户，默认即是nginx，可以不进行设置
user  nginx;
#Nginx进程，一般设置为和CPU核数一样
worker_processes  1;   
#错误日志存放目录
error_log  /var/log/nginx/error.log warn;
#进程pid存放位置
pid        /var/run/nginx.pid;


events {
    worker_connections  1024; # 单个后台进程的最大并发数
}


http {
    include       /etc/nginx/mime.types;   #文件扩展名与类型映射表
    default_type  application/octet-stream;  #默认文件类型
    #设置日志模式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;   #nginx访问日志存放位置

    sendfile        on;   #开启高效传输模式
    #tcp_nopush     on;    #减少网络报文段的数量

    keepalive_timeout  65;  #保持连接的时间，也叫超时时间

    #gzip  on;  #开启gzip压缩

    include /etc/nginx/conf.d/*.conf; #包含的子配置项位置和文件
}


nginx.conf 配置文件分为三部分：
	全局块: 从配置文件开始到 events 块之间的内容，主要会设置一些影响nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等
	events块:涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。
	http块:Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。

```

## Conf.d/default.conf

```

server {
    listen       80;   #配置监听端口
    server_name  localhost;  //配置域名

    #charset koi8-r;     
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;     #服务默认启动目录
        index  index.html index.htm;    #默认访问文件
    }

    #error_page  404              /404.html;   # 配置404页面

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;   #错误状态码的显示页面，配置后需要重启
    
    # 定位，把特殊的路径或文件再次定位。(文件 哪些可以访问)
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

# 三、启动、停止、状态

```
已经容器化管理，
启动、停止、状态皆与容器的命令相同
启动
systemctl start nginx


重启：
systemctl restart nginx

状态
systemctl status nginx
```

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221128141654594.png" alt="image-20221128141654594" style="zoom:50%;" />

# 四、Nginx访问权限

```markdown
 location / {
        allow  45.76.202.231;
        deny   all;
    }
结果：允许45.76.202.231,拒绝45.76.202.231 之外的访问
如果颠倒
location / {
		deny   all;
        allow  45.76.202.231;
    }
结果：拒绝所有访问

***就是在同一个块下的两个权限指令，先出现的设置会覆盖后出现的设置***
```

# 五、控制网络

需要先准备一个web应用（静态资源、http请求）、nginx 服务器





# 六、上传ssl

先下载源证书、私钥，之后在配置到nginx 上

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221128141814677.png" alt="image-20221128141814677" style="zoom:50%;" /> 



在nginx 目录创建cert

```
sudo mkdir cert

# 存放源证书
sudo touch ssl.pem

# 存放私钥
sudo touch ssl.key
```

修改配置文件

```
    server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        ssl_certificate "/etc/nginx/cert/ssl.pem";
        ssl_certificate_key "/etc/nginx/cert/ssl.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers PROFILE=SYSTEM;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

