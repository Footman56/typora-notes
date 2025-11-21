# 一、安装

```
wget http://download.sonatype.com/nexus/3/nexus-3.14.0-04-unix.tar.gz # 下载对应的安装包
tar zxvf nexus-3.14.0-04-unix.tar.gz # 解压缩
mv nexus-3.14.0-04/ /usr/local/nexus
cd /usr/local/nexus/bin
```

# 二、修改配置

```
cd /usr/local/nexus/nexus-3.12/bin
sudo vim nexus.vm
修改分配的堆大小
```





