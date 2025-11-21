# 一、引入依赖

## 父工程

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.5</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>


 <properties>
        <java.version>1.8</java.version>
        <dubbo.version>2.7.8</dubbo.version>
        <spring.boot.version>2.7.5</spring.boot.version>
    </properties>


<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter</artifactId>
                <version>${spring.boot.version}</version>
            </dependency>

            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <version>${spring.boot.version}</version>
                <exclusions>
                    <exclusion>
                        <artifactId>jackson-databind</artifactId>
                        <groupId>com.fasterxml.jackson.core</groupId>
                    </exclusion>
                </exclusions>
            </dependency>

            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <version>${spring.boot.version}</version>
                <scope>test</scope>
                <!--默认使用junit5-->
                <exclusions>
                    <exclusion>
                        <groupId>org.junit.vintage</groupId>
                        <artifactId>junit-vintage-engine</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
          
            <!--dubbo-->
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-spring-boot-starter</artifactId>
                <version>${dubbo.version}</version>
            </dependency>

            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-dependencies-zookeeper</artifactId>
                <version>${dubbo.version}</version>
            </dependency>
        </dependencies>

    </dependencyManagement>
```

## 子工程-提供者

```xml
<parent>
        <groupId>com.huochai</groupId>
        <version>0.0.1-SNAPSHOT</version>
        <artifactId>demo</artifactId>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>


<dependencies>

        <dependency>
            <groupId>com.huochai</groupId>
            <artifactId>test-interface</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <!--dubbo-->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-reload4j</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>

    </dependencies>
```

## 子工程-消费者

```xml
<parent>
        <groupId>com.huochai</groupId>
        <version>0.0.1-SNAPSHOT</version>
        <artifactId>demo</artifactId>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>


        <!--dubbo-->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-reload4j</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
```



# 二、服务提供者

```java

import org.apache.dubbo.config.annotation.DubboService;

/**
 *
 *@author peilizhi
 *@date 2023/11/2 16:14
 **/
@DubboService
public class HelloServiceImpl implements HelloService {

    @Override
    public String sayHello() {
        return "sayHello";
    }
}
```

# 三、服务提供者配置

```yaml
server:
  port: 9101
dubbo:
  # 服务名称
  application:
    name: test.client
  protocol:
    # 协议及协议端口
    name: dubbo
    port: 20881
  registry:
    # 注册中心类型、地址、端口
    address: zookeeper://127.0.0.1:2181
```

# 四、服务消费者启动类

```java
import org.apache.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import tk.mybatis.spring.annotation.MapperScan;

@SpringBootApplication
@EnableInvoker
@EnableDubbo
@MapperScan(basePackages = "com.huochai.domain.testDomain.mapper")
public class TestConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(TestConsumerApplication.class, args);
    }

}
```

# 五、服务消费者

```java
import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.stereotype.Service;

/**
 *
 *@author peilizhi
 *@date 2023/11/2 16:16
 **/
@Service
public class ConsumerServiceImpl implements ConsumerService {

    @DubboReference
    private HelloService helloService;

    @Override
    public String consumer() {
        return helloService.sayHello();
    }
}

```

#  六、服务消费者配置

注意：如果在同一台机器上配置消费者和提供者的话，**dubbo.protocol.port 不能相同**

```yaml
dubbo:
  application:
    name: test-consumer
  protocol:
    port: 20882
    # 启用@EnableDubbo 注解必须指定protocol.id 或者protocol.name
    id: test-consumer
  registry:
    address: zookeeper://127.0.0.1:2181
```

***服务消费者和服务提供者的dubbo.registry.address 必须一致***



启动之后发现，说明服务提供者注册成功啦

![image-20231103001033699](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202311030010053.png)

# 七、直连

直连是针对

## 7.1 服务提供者

只改配置文件即可

```yaml
server:
  port: 9101
dubbo:
  # 服务名称
  application:
    name: test.client
  protocol:
    # 协议及协议端口
    name: dubbo
    port: 20881
  registry:
    # 直连,不需要注册中心
    address: N/A
    # 注册中心类型、地址、端口
#    address: dubbo://127.0.0.1:2181
```



## 7.2 服务消费者

```yaml
dubbo:
  application:
    name: test-consumer
  protocol:
    port: 20882
    # 启用@EnableDubbo 注解必须指定protocol.id 或者protocol.name
    id: test-consumer
  registry:
    address: N/A
```

```java
@Service
public class ConsumerServiceImpl implements ConsumerService {

  
    @DubboReference(check = false, url = "dubbo://127.0.0.1:20881/com.huochai.HelloService")
    private HelloService helloService;

    @Override
    public String consumer() {
        return helloService.sayHello();
    }
}
```

