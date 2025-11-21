1. 卸载自带的jdk

   ```bash
   # 通过以下命令查看是否已经安装jdk
   yum list installed |grep java
   
   # 安装了则通过以下命令删除
   yum -y remove #名称
   ```

2. 下载

   ```sh
   yum -y list java*
   
   选择其中一个
   sudo yum install java-1.8.0-openjdk.x86_64
   ```

3. 检查是否安装成功

   ```sh
   java -version
   ```

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410200033927.png" alt="image-20241020003348880" style="zoom:50%;" />

4. 获取jdk 的安装位置 

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410200040205.png" style="zoom:50%;" />

5. 配置环节变量

   ```sh
   vi /etc/profile
   ```

   ```
   export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.412.b08-1.el7_9.x86_64
   export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   export PATH=$PATH:$JAVA_HOME/bin
   ```

   ```
   source /etc/profile
   ```

6. 检查是否配置正确

   ```
   echo $JAVA_HOME
   echo $CLASSPATH
   echo $PATH
   ```

   

7. 卸载

   ```sh
   rpm -qa|grep jdk
   ```

   