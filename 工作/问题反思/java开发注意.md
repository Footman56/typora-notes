不用使用  new BigDecimal(double) 这种构造方法

要使用BigDecimal.valueOf(double) ，这种会先讲double 转换成String ,再构造

# 注解

注解里面的方法的返回值是固定的，不能是自定义的bean

```
1. A primitive type(java的八种基本类型：byte、short、int、long、float、double、char、boolean)
2. String
3. Class
4. An enum type
5. An annotation type
6. An array type ：类型为以上任一类型的数组
```





```
Unable to open socket file: target process not responding or HotSpot VM not loaded
```

1. 可能是命令中的pid 与 java进程的pid 不一致

2. 运行命令的用户与java 进程的用户不一致

3. 高版本  自定义命令

   这是一个从Java 6开始。 jvm启动时产生进程号的临时文件目录优先使用-Djava.io.tmpdir指定的目录，没有指定-Djava.io.tmpdir参数才使用/tmp/hsperfdata_$USER。jps 、jstack 从/tmp/hsperfdata_$USER目录读取不到pid信息

4. pid 所在目录被删除

   jvm运行时会生成一个目录hsperfdata_$USER($USER是启动java进程的用户)，在linux中默认是/tmp。目录下会有些pid文件，存放jvm进程信息。jps、jstack等工具读取/tmp/hsperfdata_$USER下的pid文件获取连接信息。 如果没有这个目录的话就会提示报错。

   没有目录的原因是 系统每天会用tmpwatch命令检查并删除 /tmp 下超过240小时未访问过的文件和目录。

   配置在 /etc/cron.daily/tmpwatch 中



在修改pom文件之后，要进行clean 、compile 