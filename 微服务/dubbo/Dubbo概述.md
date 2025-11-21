# Dubbo

## 一、架构演进

<img src="/Users/mac/Desktop/屏幕快照 2021-03-07 21.00.27.png" style="zoom:50%;" />

```markdown
#### 单一应用架构

当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。

#### 垂直应用架构

当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，提升效率的方法之一是将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键。

#### 分布式服务架构

当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。

#### 流动计算架构

当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。
```

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-06 15.22.41.png" style="zoom:50%;" />

| 节点        | 角色说明                               |
| ----------- | -------------------------------------- |
| `Provider`  | 暴露服务的服务提供方                   |
| `Consumer`  | 调用远程服务的服务消费方               |
| `Registry`  | 服务注册与发现的注册中心               |
| `Monitor`   | 统计服务的调用次数和调用时间的监控中心 |
| `Container` | 服务运行容器                           |

```
0、服务容器负责启动，加载，运行服务提供者。
1、服务提供者在启动时，向注册中心注册自己提供的服务。
2、服务消费者在启动时，向注册中心订阅自己所需的服务。
3、注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
4、服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
5、服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。
```

## 二、使用

### 1、配制Zookeeper

### 2、安装Dubbo-admin

```
 要修改配置文件中的admin.registry.address=ip:port        **port=2181**
```

#### 3、项目使用（XML）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="hello-world-app"  />
 
    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234" />
 
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />
 
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" />
 
    <!-- 和本地bean一样实现服务 -->
    <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl" />
</beans>
```

```
在非分布式系统中，将服务的提供者和消费者是在同一容器内，可以直接注入。就是像引用jar一样
```

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-06 15.31.19.png" style="zoom:50%;" />

##### a、将服务提供者注册到注册中心

###### 1、导入Dubbo依赖，zookeeper的注册中心

```xml
<properties>
    <dubbo.version>2.7.9</dubbo.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
        <version>${dubbo.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <version>${dubbo.version}</version>
        <type>pom</type>
    </dependency>
</dependencies>
```

###### 2、配置服务提供者

```xml
provider.xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系，当前应用的名称，不要重名 -->
    <dubbo:application name="user"  />

    <!-- 使用multicast广播注册中心暴露服务地址  指定注册中心的位置-->
    <dubbo:registry protocol="zookeeper" address="47.96.235.128:2181" />


    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />

    <!-- 声明需要暴露的服务接口
    interface: 服务端接口
    ref:指向服务的真正实现对象（容器内）
    -->
    <dubbo:service interface="com.huochai.service.UserService" ref="userService" />

    <!-- 和本地bean一样实现服务 -->
    <bean id="userService" class="com.huochai.service.impl.UserServiceImpl" />
</beans>
```

```
需要先运行dubbo-admin-server
之后在运行dubbo-admin-ui  输入 npm run dev
```



**返回的结果必须可序列化。**

修改一个结果之后就需要重新将提供者注册到注册中心上

##### b、让服务消费者去注册中心订阅服务提供者的服务地址

##### 1、导入Dubbo，zookeeper依赖

##### 2、编写消费者

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo" xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="order" />

    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
    <dubbo:registry protocol="zookeeper" address="47.96.235.128:2181" />

    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="userService" interface="com.huochai.service.UserService" />

  // 提供包扫描，便于测试
    <context:component-scan base-package="com.huochai.service" />

</beans>
```

在消费者端也需要知道调用的接口，只是不需知道具体的调用内容（需要引用只有接口定义的jar）

#### 4、SpringBoot 整合Dubbo(注解版)

##### a、引入Bubbo 的start 

```xml
 			<dubbo.version>2.7.4.1</dubbo.version>

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
```

##### b、配置文件

默认的通讯协议是dubbo， 端口号是20880. **如果本地启两个项目的话，需要指定不同的dubbo端口**

```
根据配置文件的内容完全转换成properties,yaml

dubbo:
  application:
    name: test-consumer
  protocol:
    port: 20881
  registry:
    address: zookeeper://127.0.0.1:2181
```

##### c、注解

```markdown
提供者使用import com.alibaba.dubbo.config.annotation.Service;
@Service
** 在提供者方的主程序类上加入@EnableDubbo 或者也可以在配置文件中使用dubbo.scan.base-package来替代   @EnableDubbo **

可以指定版本，group。
*@Service(interfaceClass = IHardwareRpcService.class, version = "1.0.0", timeout = 3000)*

消费者使用@Reference 来依赖

* @Reference(check = false,group = "${attendance.clock.group}")*
 private IClockService clockService;



消费者需要导入提供者的jar包？
需要
```

```markdown
dubbo的默认超时时间是1s,设置timeout来设置超时时间

方法级优先，接口级次之，全局配置再次之。
如果级别一样，则消费方优先，提供方次之。

配置覆盖原则：
	精确优先 
	消费者设置优先
	![屏幕快照 2021-03-08 17.15.09](/Users/mac/Desktop/屏幕快照 2021-03-08 17.15.09.png)
```

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-06 15.32.55.png" style="zoom:50%;" />

[https://dubbo.apache.org/zh/docs/v2.7/user/configuration/](https://dubbo.apache.org/zh/docs/v2.7/user/configuration/)

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-06 15.26.27.png" style="zoom:50%;" />



### 5、属性配置

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-06 15.36.49.png" style="zoom:50%;" />

## 三、注意点

```
无论是编写消费者还是服务提供者，都需要注意一下几点：
1、相同的注册中心
2、相同的group
3、服务提供者使用的协议，暴露的端口号
4、服务提供者需要提供出去，用于服务消费者计算路径
```

## 四、配置

### 1、在 Provider 端尽量多配置 Consumer 端属性

```markdown
**Consumer**
timeout：方法调用的超时时间
retries：失败重试次数，缺省是 2 2
loadbalance：负载均衡算法 3，缺省是随机 random。还可以配置轮询 roundrobin、最不活跃优先 4 leastactive 和一致性哈希 consistenthash 等
actives：消费者端的最大并发调用限制，即当 Consumer 对一个服务的并发调用到上限后，新调用会阻塞直到超时，在方法上配置 dubbo:method 则针对该方法进行并发限制，在接口上配置 dubbo:service，则针对该服务进行并发限制
```

```markdown
**Provider**
threads：服务线程池大小
executes：一个服务提供者并行执行请求上限，即当 Provider 对一个服务的并发调用达到上限后，新调用会阻塞，此时 Consumer 可能会超时。在方法上配置 dubbo:method 则针对该方法进行并发限制，在接口上配置 dubbo:service，则针对该服务进行并发限制
```

## 五、实践

### 1、分包

```
服务接口、服务模型、服务异常等均放在 API 包
```

### 2、粒度

```
服务接口尽可能大粒度，每个服务方法应代表一个功能，而不是某功能的一个步骤
不建议使用过于抽象的通用接口，如：Map query(Map)
```

### 3、版本

```
每个接口都应定义版本号

建议使用两位版本号，因为第三位版本号通常表示兼容升级，只有不兼容时才需要变更服务版本。
```

### 4、兼容性

```
服务接口增加方法，或服务模型增加字段，可向后兼容，删除方法或删除字段，将不兼容，
枚举类型新增字段也不兼容，需通过变更版本号升级。
```

### 5、枚举值

```markdown
如果是完备集，可以用 Enum，比如：ENABLE, DISABLE。

**如果是业务种类，以后明显会有类型增加，不建议用 Enum，可以用 String 代替。**

如果是在返回值中用了 Enum，并新增了 Enum 值，建议先升级服务消费方，这样服务提供方不会返回新值。

如果是在传入参数中用了 Enum，并新增了 Enum 值，建议先升级服务提供方，这样服务消费方不会传入新值。
```

### 6、序列化

```
服务参数及返回值建议使用 POJO 对象，即通过 setter, getter 方法表示属性的对象。

服务参数及返回值都必须是传值调用，而不能是传引用调用，消费方和提供方的参数或返回值引用并不是同一个，只是值相同，Dubbo 不支持引用远程对象。
```







Invalid name="org.apache.dubbo.config.ProtocolConfig#0" contains illegal character, only digit, letter, '-', '_' or '.' is legal.

必须指定 id或者name ,否则会报错
