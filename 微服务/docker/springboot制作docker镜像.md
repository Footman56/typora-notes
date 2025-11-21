1. 构建项目  在target目录下就会有jar包

   ```shell
   mvn clean package
   ```

   

2. 编写Dockerfile 需要在根目录下

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410192101495.png" style="zoom:50%;" />

   /home/huochai/app/json-change

   ```dockerfile
   FROM centos:7
   LABEL authors="peilizhi"
   
   # 指定维护者信息
   LABEL maintainer="peilizhi450@gmail.com"
   
   # 将工作目录设置为/app
   WORKDIR /app
   
   # 创建jdk目录
   RUN mkdir /usr/local/java         
   
   # 将编译好的jar文件复制到/app目录下
   # COPY是Dockerfile的一个指令，用于从构建上下文（通常是Dockerfile所在的目录及其子目录）复制文件或目录到Docker镜像中。
   #target/my-spring-boot-app-1.0.0.jar指定了源路径，这是相对于Dockerfile位置的路径。在这个例子中，它指的是构建Spring Boot应用后生成的jar文件的路径。这个文件在构建Docker镜像之前必须存在于该路径下。
   #app.jar指定了目标路径和文件名，在这里是Docker镜像内的/app目录下，并将复制过来的文件重命名为app.jar。如果目标路径不存在，Docker会自动创建这个目录。
   COPY target/json-change-0.0.1-SNAPSHOT.jar app.jar
   
   # 复制宿主机中的jdk到镜像中
   COPY /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.412.b08-1.el7_9.x86_64   /usr/local/java/jdk8      
    
   # 设置环境变量
   ENV JAVA_HOME=/usr/local/java/jdk8        
   ENV JRE_HOME $JAVA_HOME/jre
   ENV CLASSPATH $JAVA_HOME/bin/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
   
   # 暴露9100端口
   EXPOSE 9100
   
   # 运行jar文件
   ENTRYPOINT ["java","-jar","app.jar"]
   ```

    **注 FROM openjdk:8-jdk-alpine 这里有问题，获取不到镜像**

3. 构建docker镜像

   ```shell
   docker build -t json-change-java .
   ```

   