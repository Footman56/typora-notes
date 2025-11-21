charles 是mac上的抓包工具

# 一、简单使用

​    首先要开启抓包

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20230213172636905.png" alt="image-20230213172636905" style="zoom:50%;" />

## 1、开启mac 代理



<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220323172512057.png" alt="image-20220323172512057" style="zoom:50%;" />

 ## 2、配置代理设置

Proxy->Proxy settings

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220323172625367.png" alt="image-20220323172625367" style="zoom:50%;" />

## 3、兼容其他代理

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220323172747468.png" alt="image-20220323172747468" style="zoom:50%;" />

PAC模式不适合抓包情景，可以不抓包时优先使用

# 三、HTTPS

proxy -> SSL  proxy settings 

开启ssl 代理 

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220323174147023.png" alt="image-20220323174147023" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220323174313743.png" alt="image-20220323174313743" style="zoom:50%;" />



下载证书，并设置根系统信任

![image-20231207182022775](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202312071820187.png)

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202312071820295.png" alt="image-20231207182059257" style="zoom:50%;" />

# 四、手机代理

## 0、按照证书

### a、下载证书

+ 浏览器打开chls.pro/ssl

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220323180827672.png" alt="image-20220323180827672" style="zoom:25%;" />

+ 安装证书

+ 安全->凭据存储->从存储设置安装证书->WLAN 证书-> 选择之前下载好的证书

  

## 1、查看Mac ip 地址

### a、charles 

Help -> Local IP address

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220323175738451.png" alt="image-20220323175738451" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220323175831477.png" alt="image-20220323175831477" style="zoom:50%;" /> 

### b、命令行

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220323175942670.png" alt="image-20220323175942670" style="zoom:50%;" />

## 2、 链接相同wifi

## 3、wifi 配置

+ 长按 wifi 进入配置页面

+ 主机名写 mac 的ip

+ 端口写 代理的端口：8888

  

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220323180300909.png" alt="image-20220323180300909" style="zoom:20%;" />

**注：在正常使用wifi 时需要关闭代理配置**

## 4、mac 同意监听

# 五、页面操作

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220323181633421.png" alt="image-20220323181633421" style="zoom:50%;" />

扫把：清空所有请求

红点：红点亮表示正在抓取请求，红点暗表示不抓去请求

乌龟：灰色乌龟网络设置正常，绿色乌龟表示慢速网络

六角形图标：断点图标，灰色说明断点未开启，红色说明在使用断点。

钢笔图标：编辑请求，点击之后可以修改请求的内容。先点击请求，再点击钢笔

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220422143341006.png" alt="image-20220422143341006" style="zoom:50%;" />

刷新图标：重复发送请求的图标，先选定某一请求点击该图标则请求会被再次发送。