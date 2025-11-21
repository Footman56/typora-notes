1. 安装

   ```sh
   yum -y install maven
   ```



2. 检查是否正确

   ```sh
   mvn -version
   ```

   ![image-20241020011527558](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410200115617.png)

2. vi /etc/maven/settings.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>D:\dev_mavenRepository</localRepository>
  <pluginGroups>    
  </pluginGroups>
  <proxies>
  </proxies>
  <servers>
  </servers>
  <mirrors>
    <mirror>
	  <id>nexus-aliyun</id>
	  <mirrorOf>central</mirrorOf>
	  <name>Nexus aliyun</name>
	  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
	</mirror>
  </mirrors>
  <profiles>
  </profiles>
</settings>
```

