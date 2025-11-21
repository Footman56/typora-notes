1. 正常下载需要的jdk

2. 在.bash_profile 中配置路径

   ```bash
   #========java config=============
   JAVA_HOME_BASIC=/Library/Java/JavaVirtualMachines
   JAVA_HOME_8=$JAVA_HOME_BASIC/jdk1.8.0_291.jdk/Contents/Home
   JAVA_HOME_17=$JAVA_HOME_BASIC/jdk-17.jdk/Contents/Home
   
   
   # 默认是java8
   export JAVA_HOME=$JAVA_HOME_8
   alias jdk8="export JAVA_HOME=$JAVA_HOME_8 && echo current JDK has switched to oracle jdk version 1.8. && java -version"
   alias jdk17="export JAVA_HOME=$JAVA_HOME_17 && echo current JDK has switched to openjdk version 17. && java -version"
   
   #========class_path==============
   CLASS_PATH=$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:.
   
   PATH=$PATH:$JAVA_HOME/bin:
   export JAVA_HOME=$JAVA_HOME
   ```

   