# Zipkin

```
服务追踪的追踪单元是从客户发起请求（request）抵达被追踪系统的边界开始，到被追踪系统向客户返回响应（response）为止的过程，称为一个trace。
每个 trace 中会调用若干个服务，为了记录调用了哪些服务，以及每次调用的消耗时间等信息，在每次调用服务时，埋入一个调用记录，称为一个span。
这样，若干个有序的 span 就组成了一个 trace。在系统向外界提供服务的过程中，会不断地有请求和响应发生，也就会不断生成 trace，把这些带有 span 的 trace 记录下来，就可以描绘出一幅系统的服务拓扑图。


目的：明确每一次请求的调用链路，无论请求是http，dubbo,数据库访问，mq,xxl-job.都能够提供明确的traceId，供日志查询


ZipKin可以分为两部分，一部分是zipkin server，用来作为数据的采集存储、数据分析与展示；zipkin client是zipkin基于不同的语言及框架封装的一些列客户端工具，这些工具完成了追踪数据的生成与上报功能.
上报应用信息，dubbo调用，sql 执行


不光要在日志上打印， 还需要上报的服务器来生产链路图

```

<img src="/Users/mac/Desktop/屏幕快照 2021-03-09 12.39.58.png" style="zoom:50%;" />

```
在追踪日志中，有几个基本概念spanId、traceId、parentId

    traceId：用来确定一个追踪链的16字符长度的字符串，在某个追踪链中保持不变。

    spanId：区域Id，在一个追踪链中spanId可能存在多个，每个spanId用于表明在某个服务中的身份，也是16字符长度的字符串。

    parentId：在跨服务调用者的spanId会传递给被调用者，被调用者会将调用者的spanId作为自己的parentId，然后自己再生成spanId。
```

**client 负责提供日志**

**Service 负责收集，查询，展示日志**

## 零、安装

```
docker run -d -p 9411:9411 openzipkin/zipkin

```

## 一、dubbo + zipkin

#### 1、引入pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
     <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
        <relativePath/>
        <!-- lookup parent from repository -->
    </parent>
  
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>dubbo-apache</name>
    <description>Demo project for Spring Boot</description>
 
    <properties>
        <java.version>1.8</java.version>
        <dubbo.version>2.7.4.1</dubbo.version>
      	<brave.version>5.9.1</brave.version>
    		<zipkin-reporter.version>2.11.1</zipkin-reporter.version>
    </properties>
 
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
        <dependency>
            <groupId>com.lmax</groupId>
            <artifactId>disruptor</artifactId>
            <version>3.4.2</version>
        </dependency>
        <!-- Dubbo Spring Boot Starter -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>${dubbo.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <version>${dubbo.version}</version>
            <type>pom</type>
             <exclusions>
                <exclusion>
                    <artifactId>log4j</artifactId>
                    <groupId>log4j</groupId>
                </exclusion>
 
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
      
      <!--  tracing  -->
    <dependency>
        <groupId>io.zipkin.brave</groupId>
        <artifactId>brave-instrumentation-dubbo</artifactId>
    </dependency>
    <dependency>
        <groupId>io.zipkin.brave</groupId>
        <artifactId>brave-spring-beans</artifactId>
    </dependency>
  
    <dependency>
        <groupId>io.zipkin.brave</groupId>
        <artifactId>brave-context-slf4j</artifactId>
    </dependency>
    <!--  tracing & zipkin  -->
    <dependency>
        <groupId>io.zipkin.reporter2</groupId>
        <artifactId>zipkin-sender-okhttp3</artifactId>
    </dependency>
    <!-- tracing & spring mvc  -->
    <dependency>
        <groupId>io.zipkin.brave</groupId>
        <artifactId>brave-instrumentation-spring-webmvc</artifactId>
    </dependency>
    <dependency>
        <groupId>io.zipkin.brave</groupId>
        <artifactId>brave-instrumentation-httpclient</artifactId>
    </dependency>
    </dependencies>
  
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>io.zipkin.brave</groupId>
        <artifactId>brave-bom</artifactId>
        <version>${brave.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <dependency>
        <groupId>io.zipkin.reporter2</groupId>
        <artifactId>zipkin-reporter-bom</artifactId>
        <version>${zipkin-reporter.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
     </dependencies>
    </dependencyManagement>
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
 
</project>
  
```

 

#### 2、配置zipkin

```properties
#通过http协议上报tracing信息，这里的地址是zipkin服务的地址
#spring.zipkin.base-url=http://localhost:9411
#服务名称
spring.zipkin.service.name=my-consumer
#开启上报到zipkin服务，如果不开启的话，tracing信息还是会在不同服务中传递，只是不会上报的zipkin服务端
spring.zipkin.enabled=false
#使用默认 http 方式收集 span 需要配置此项
spring.zipkin.sender.type=web
#采样率，默认是0.1, 如果是1的话，代表百分之百采样
spring.sleuth.sampler.probability=1
```

#### 3、配置dubbo

```properties
#add tracing filter
dubbo.consumer.filter = tracing
#add tracing filter
dubbo.provider.filter = tracing
```

#### 4、日志配置

```
在日志的输出格式中加入  %X{traceId}
```



## 二、mysql + zipkin



#### 1、导入依赖

```xml
mysql8
<dependency>
	<groupId>io.zipkin.brave</groupId>
	<artifactId>brave-instrumentation-mysql8</artifactId>
</dependency>


mysql6
<dependency>
	<groupId>io.zipkin.brave</groupId>
	<artifactId>brave-instrumentation-mysql6</artifactId>
</dependency>


mysql5
<dependency>
	<groupId>io.zipkin.brave</groupId>
	<artifactId>brave-instrumentation-mysql</artifactId>
</dependency>
```





#### 2、添加mysql连接参数 

```java
?queryInterceptors=brave.mysql8.TracingQueryInterceptor&exceptionInterceptors=brave.mysql8.TracingExceptionInterceptor
```



<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-19 15.00.33.png" style="zoom:50%;" />



```
Collector：收集器组件，它主要用于处理从外部系统发送过来的跟踪信息，将这些信息转换为 Zipkin 内部处理的 Span 格式，以支持后续的存储、分析、展示等功能。
Storage：存储组件，它主要对处理收集器接收到的跟踪信息，默认会将这些信息存储在内存中，我们也可以修改此存储策略，通过使用其他存储组件将跟踪信息存储到数据库中。
RESTful API：API 组件，它主要用来提供外部访问接口。比如给客户端展示跟踪信息，或是外接系统访问以实现监控等。
Web UI：UI 组件，基于 API 组件实现的上层应用。通过 UI 组件用户可以方便而有直观地查询和分析跟踪信息。

```

