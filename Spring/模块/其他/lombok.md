# 一、lombok

## idea 引入lombok

1. 先下载对应的插件

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220716185743125.png" alt="image-20220716185743125" style="zoom:50%;" />

2. 配置相关设置

   <img src="https://img-blog.csdnimg.cn/20200214154552256.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NjMzMjAz,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom: 50%;" />

3. 引入对应jar

   ```xml
   <lombok.version>1.18.22</lombok.version>  
   <dependency>
                   <groupId>org.projectlombok</groupId>
                   <artifactId>lombok</artifactId>
                   <version>${lombok.version}</version>
               </dependency>
   ```

   

4. 快乐使用