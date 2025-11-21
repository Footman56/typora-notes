# 比较版本冲突

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220427172512703.png" alt="image-20220427172512703" style="zoom:50%;" />

排除标红的依赖：

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220427173609785.png" alt="image-20220427173609785" style="zoom:50%;" />

resolution will not be reattempted until the update interval of nexus has elapsed or updates are forced

需要在maven仓库中删除对应包里面的.lastUpdated 之后重新刷新





仓库使用阿里云镜像

在setting.xml中添加

```xml
<mirrors>
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
</mirrors>
```





如果编译项目的时候提示：

Non-resolvable parent POM for jt-manage:jt-manage:[unknown-version]: Could not find artifact com.jt:jt-parent:pom:0.0.1-SNAPSHOT and '**parent.relativePath**'， points at wrong local POM @ line 9, column 11

解决方式：需要先将父工程 install 下。再编译





如果项目目录下一直有个pom 文件置灰并且有滑线。可能是被忽略啦。取消忽略就可以啦

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202311021602499.png" alt="image-20231102160226152" style="zoom:50%;" />







报错：

```
Blocked mirror for repositories: [nexus (http://123.57.204.47:8081/nexus/content/groups/public, default, releases+snapshots)]

Since Maven 3.8.1 http repositories are blocked.
```

是因为 maven 升级到 3.8.1 之后，不能通过http 接口去请求代码仓库了。

1. 降级，不使用 3.8 之后的版本，使用 3.6就没有问题

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202402281140881.png" alt="image-20240228114040542" style="zoom:50%;" />

**不同的项目，可以使用不同的maven配置。**

切换版本就可以解决这个问题。

2. 在配置文件中 添加mirror 【不推荐】

   ```xml
       <mirror>
         <id>maven-default-http-blocker</id>
         <mirrorOf>external:http:*</mirrorOf>
         <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
         <url>http://0.0.0.0/</url>
         <blocked>true</blocked>
       </mirror> 
   ```

3. 仓库地址改成https ，需要仓库支持这种

