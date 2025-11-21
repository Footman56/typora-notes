**文档**

https://arthas.aliyun.com/doc

从远程下载

```bash
# 获取
curl -O https://arthas.aliyun.com/arthas-boot.jar
# 启动arthas
java -jar arthas-boot.jar
```

**执行该程序的用户需要和目标进程具有相同的权限**

# dashboard

综合展示线程、内存情况。这是实时刷新的

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408021904110.png" alt="image-20230512115600306" style="zoom:50%;" />

# thread 查看线程情况

![image-20230512115658403](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408021904198.png)

# watch  查看接口返回

# stack 查看调用链路

# trace 

方法内部调用路径，并输出方法路径上的每个节点上耗时.只能统计一级函数的用时，不会向下深入统计接口用时





# 诊断 Docker 里的 Java 进程

```sh
docker exec -it  '7e3181666917' /bin/bash -c "wget https://arthas.aliyun.com/arthas-boot.jar && java -jar arthas-boot.jar"
```

# 查看内存情况

```sh
memory
```

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408021856621.png" alt="image-20240802185635251" style="zoom:50%;" />

# 查看jvm 情况

```she
jvm
```

# dump 内存

复制文件到docker内

1. 先获取docker 容器名称

   docker ps -a |grep ''

   ![image-20230914163316194](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408021907698.png)

2. 复制文件到容器内

   先写服务器中的文件，再写容器内的目录

   ```sh
   docker cp /tmp/arthas-boot.jar f7aac24a2a90:/tmp
   ```

3. 进入容器内 

   ```sh
   sudo docker exec -it 4b2c2f085f13 bash
   ```

   

4. 启动arthas 

   ```
   cd tmp
   java -jar arthas-boot.jar
   ```

5. 获取dump 文件

   ```
   heapdump [文件路径]
   ```

   

6. 拷贝文件到服务器

   ```
   # 归档前文件
   docker cp devtest_appraisal_1:/tmp/heapdump2024-08-02-15-507540841825929176331.hprof /home/saas/log/appraisal
   
   # 归档后文件
   docker cp devtest_appraisal_1:/tmp/heapdump2024-08-02-15-597660752462417294412.hprof /home/saas/log/appraisal
   ```

7. 下载到本地

   ```
   tar -czvf archive_dump.tar.gz heapdump2024-08-02-15-597660752462417294412.hprof
   
   scp saas@47.94.45.156:/home/saas/log/appraisal/archive_dump.tar.gz /Users/peilizhi
   ```

