



![image-20241019160718210](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410191607656.png)



只有选择  **https://start.aliyun.com/** 才能设置 java 8 





# springboot

## 一、入门

### a.POM文件

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starters</artifactId>
    <version>1.5.10.RELEASE</version>
</parent>
```

表明父工程依赖关系，在父工程中有约定的版本，因此导入之后就不需在常用依赖的版本

```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>2.3.7.RELEASE</version>
    <configuration>
        <mainClass>com.huochai.SpringbootDbApplication</mainClass>
    </configuration>
    <executions>
        <execution>
            <id>repackage</id>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

便于将应用程序打包



### b. 主程序

```java
@SpringBootApplication
public class SpringbootDbApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootDbApplication.class, args);
    }
}
```



@Configuration :Spring的配置注解

表示一个类提供 Spring Boot 应用程序@Configuration 。可以用作 Spring 标准@Configuration注释的替代品，以便可以自动找到配置（例如在测试中）。
应用程序应该只包含一个@SpringBootConfiguration并且大多数惯用的 Spring Boot 应用程序将从@SpringBootApplication继承它。



#### @EnableAutoConfiguration：

启用 Spring Application Context 的自动配置，尝试猜测和配置您可能需要的 bean（提供一些默认配置）

 有了这个注解就可以自动配置一些类,会装载主程序类所在的包以及子包下所有Bean,与@ComponentScan所扫描的对象不一样，会扫描@Entity类，是@ComponentScan无法支持的

使用@EnableAutoConfiguration注释的类的包，通常通过@SpringBootApplication ，具有特定的意义，通常用作“默认值”

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration
```



##### @AutoConfigurationPackage：

指示包含注释类的包应注册到AutoConfigurationPackages （用于存储自动配置包以供以后参考的类）

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage
```

​		@Import(AutoConfigurationPackages.Registrar.class) 

​		@Import 是Spring的底层注解，表示引用这个类，将这个类加入到容器中

```java
@Order(Ordered.HIGHEST_PRECEDENCE)
// ImportBeanDefinitionRegistrar用于存储来自导入配置的基本包。
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

   @Override
   public void registerBeanDefinitions(AnnotationMetadata metadata,
         BeanDefinitionRegistry registry) {
      register(registry, new PackageImport(metadata).getPackageName());
      // 将主配置类所在的包里面的所有组件加载到容器中
   }

   @Override
   public Set<Object> determineImports(AnnotationMetadata metadata) {
      return Collections.<Object>singleton(new PackageImport(metadata));
   }
}
```



```java
@Import(AutoConfigurationImportSelector.class)
```

##### AutoConfigurationImportSelector

```java
@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
				annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}


protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
			AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 返回应考虑的自动配置类名称，此方法将使用SpringFactoriesLoader和getSpringFactoriesLoaderFactoryClass()加载候选人
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = filter(configurations, autoConfigurationMetadata);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}


// 使用给定的类加载器从"META-INF/spring.factories"加载给定类型的工厂实现的完全限定类名
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
		String factoryTypeName = factoryType.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
	}
//loadSpringFactories（）会从META-INF/spring.factories  获取所有配置类，返回的Map,再从Map中获取EnableAutoConfiguration相关的累

```

会给容器中导入很多的自动配置类（xxxxAutoConfiguration）

自动配置类可以免去大量的编写配置来加载功能组件

**Spring启动的时候会扫描所有jar路径下的`META-INF/spring.factories`，将其文件包装成Properties对象**

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220501192814709.png" alt="image-20220501192814709" style="zoom:50%;" />



# 二、配置文件

springBoot 使用全局配置文件 ，全局配置文件是固定的的

Application.properties

Application.yaml

## yaml

### 1、基本语法

k:（空格）v 键值对表示，必须要有空格

用空格的缩进来表示层级关系，只要左对齐就是同一层

属性和值是大小写敏感

### 2、值的写法

k： v 字面值直接写

‘ ’ 单引号会转移特殊字符

“ ”双引号不会转移字符

```yaml
friend: 
	lastname: xiaoming
	age: 20


pets:
	- :cat
	- :dog
```





```
@ConfigurationProperties(prefix = "person")
告诉SpringBoot将本类中的属性与配置文件中的属性像映射
只对容器内的组件起作用，需要将本类加入到容器中
@Comptent
```

# 三、数据访问

### 1.使用原生的数据源操作数据库

#### a.引入pom文件

```java
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.23</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```



#### b.配置数据源

```yaml
spring:
  profiles:
    active: test
  datasource:
    password: Admin123@
    username: huochai
    url: jdbc:mysql://47.96.235.128:3306/student?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&characterEncoding=utf-8&useSSL=false
    driver-class-name: com.mysql.cj.jdbc.Driver
```

#### c.测试

```java
@Test
public void testDataSource(){
    try {
        // 数据源
        System.out.println("dataSource.getClass() = " + dataSource.getClass());
        //测试连接
        final Connection connection = dataSource.getConnection();
        System.out.println("connection = " + connection);
    } catch (SQLException throwables) {
        throwables.printStackTrace();
    }
}
```

```java
显示的结果：
dataSource.getClass() = class com.zaxxer.hikari.HikariDataSource
2021-03-07 14:06:50.220  INFO 28753 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2021-03-07 14:06:51.183  INFO 28753 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
connection = HikariProxyConnection@307046074 wrapping com.mysql.cj.jdbc.ConnectionImpl@415d88de
```

springboot默认使用数据源是**HikariDataSource**



自动配置原理：一些自动配置包都在spring-boot-actuator-autoconfigure包中

<img src="/Users/mac/Desktop/屏幕快照 2021-03-07 14.13.19.png" style="zoom:50%;" />

<img src="/Users/mac/Desktop/屏幕快照 2021-03-07 14.17.31.png" style="zoom:50%;" />

jdbc包中配置了数据源信息



```
DataSourceConfiguration类中定义了一些常用的数据源，默认使用的HikariDataSource

spring.datasource.type指定了数据源的类型
```

```java
@Configuration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ DataSourcePoolMetadataProvidersConfiguration.class,
      DataSourceInitializationConfiguration.class })
public class DataSourceAutoConfiguration
```



```java
/**
 * Generic DataSource configuration.
 自定义数据源
 */
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type")
static class Generic {

   @Bean
   public DataSource dataSource(DataSourceProperties properties) {
     // 根据配置文件，来初始化数据源
     // 使用DataSourceBuilder来创建数据源，利用反射创建响应的type的数据源，之后绑定各个属性
     
      return properties.initializeDataSourceBuilder().build();
   }
}
```

```
public DataSourceBuilder<?> initializeDataSourceBuilder() {
		// 反射
   return DataSourceBuilder.create(getClassLoader()).type(getType())
   //  绑定属性
         .driverClassName(determineDriverClassName()).url(determineUrl())
         .username(determineUsername()).password(determinePassword());
}
```

#### d.操作数据库

使用jdbcTemplate

```java
@Test
public void testJdbc(){
    final List<Map<String, Object>> maps = jdbcTemplate.queryForList("select * from sys_user");
    System.out.println("maps = " + maps);
}
```

### 2.整合Druid数据源	

#### a.引入依赖

```xml
<!--引入druid数据源-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.16</version>
</dependency>
```

#### b.修改配置

```yaml
spring:
  profiles:
    active: test
  datasource:
    password: Admin123@
    username: huochai
    url: jdbc:mysql://47.96.235.128:3306/student?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&characterEncoding=utf-8&useSSL=false
    driver-class-name: com.mysq
    l.cj.jdbc.Driver
    #   指明数据源的类型
    type: com.alibaba.druid.pool.DruidDataSource

    # 后面的配置都是Druid特有属性，不是底层数据源的属性，建议添加组件都容器中

    #    配置初始化大小、最小、最大
    druid:
      initialSize: 5
      minIdle: 10
      maxActive: 20
      #    获取连接等待超时的时间
      maxWait: 6000
      #    间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
      timeBetweenEvictionRunsMillis: 2000
      #    一个连接在池中最小生存的时间，单位是毫秒
      minEvictableIdleTimeMillis: 600000
      maxEvictableIdleTimeMillis: 900000
  
      validationQuery: select 1
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
  
      keepAlive: true
      phyMaxUseCount: 500
      #    监控统计拦截的filters
      filters: stat
```

#### c. 测试数据源中的配置是否生效

<img src="/Users/mac/Desktop/屏幕快照 2021-03-07 15.00.53.png" style="zoom:50%;" />

#### d. 异常

```
Springboot配置druid报错Failed to bind properties under 'spring.datasource' to javax.sql.DataSource
```

**需要在pom中加入log4j**

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

#### e.监控配置

```java
// 配置Druid监控
// 1.配置一个管理后台的Servlet
@Bean
public ServletRegistrationBean servletRegistrationBean(){
    // 现在要进行druid监控的配置处理操作
    ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(
            new StatViewServlet(), "/druid/*");
    // 白名单,多个用逗号分割， 如果allow没有配置或者为空，则允许所有访问
    servletRegistrationBean.addInitParameter("allow", "127.0.0.1,172.29.32.54");
    // 黑名单,多个用逗号分割 (共同存在时，deny优先于allow)
    servletRegistrationBean.addInitParameter("deny", "192.168.1.110");
    // 控制台管理用户名
    servletRegistrationBean.addInitParameter("loginUsername", "admin");
    // 控制台管理密码
    servletRegistrationBean.addInitParameter("loginPassword", "admin123");
    // 是否可以重置数据源，禁用HTML页面上的“Reset All”功能
    servletRegistrationBean.addInitParameter("resetEnable", "false");

    return servletRegistrationBean;
}

// 2.配置一个web监控的filter
@Bean
public FilterRegistrationBean filterRegistrationBean() {
    FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean() ;
    filterRegistrationBean.setFilter(new WebStatFilter());
    //所有请求进行监控处理
    filterRegistrationBean.addUrlPatterns("/*");
    //添加不需要忽略的格式信息
    filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.css,/druid/*");
    return filterRegistrationBean ;
}
```

### 3.整合Mybatis

#### a. 引入Pom依赖

```xml
<!--引入mybatis依赖-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
```

所有这种与springboot整合的jar 中，META-INF 里面的spring.factories里面都定义了需要自动装配的配置类，配置类里面定义了一些配置，与配置文件作用相同

#### b. 创建模型

#### c.编写Mapper

```java
@Mapper
public interface StudentMapper {

    @Select("select * from sys_student where id=${id}")
    Student queryStudentById(Integer id);

    @Insert("insert into sys_student(name,age,sex) values(#{student.name},#{student.age},#{student.sex})")
    Integer insertStudent(Student student);

    @Delete("delete from sys_student where id =${id}")
    Insert deleteStudentById(Integer id);

    @Update("update sys_student set name= #{student.name} where id=#{student.id}")
    void updeteStudentById(Student student);
}
```

```
需要在Mapper中加入@Mapper 注解
或者在启动类上加入 @MapperScan("com.houchai.respository.mapper")，使用包扫描的策略
```

```java
@SpringBootApplication
@MapperScan(basePackages = "com.huochai.domain.**.respository.mapper")
public class ConsumerWebappApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerWebappApplication.class, args);
    }
}

```



#### d 自动配置

```java
@Configuration
public class MybatisConfig {
    
    // 给Mybatis添加个性化配置
    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return configuration -> {
            // 开启驼峰转换
            configuration.setMapUnderscoreToCamelCase(true);
        };
    }
}
```



```java
源文件配置
@org.springframework.context.annotation.Configuration
@ConditionalOnClass({ SqlSessionFactory.class, SqlSessionFactoryBean.class })
@ConditionalOnSingleCandidate(DataSource.class)
@EnableConfigurationProperties(MybatisProperties.class)
@AutoConfigureAfter({ DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class })
public class MybatisAutoConfiguration implements InitializingBean 


  
@Bean
@ConditionalOnMissingBean
public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
  SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
  factory.setDataSource(dataSource);
  factory.setVfs(SpringBootVFS.class);
  if (StringUtils.hasText(this.properties.getConfigLocation())) {
    factory.setConfigLocation(this.resourceLoader.getResource(this.properties.getConfigLocation()));
  }
  applyConfiguration(factory);
  if (this.properties.getConfigurationProperties() != null) {
    factory.setConfigurationProperties(this.properties.getConfigurationProperties());
  }
  if (!ObjectUtils.isEmpty(this.interceptors)) {
    factory.setPlugins(this.interceptors);
  }
  if (this.databaseIdProvider != null) {
    factory.setDatabaseIdProvider(this.databaseIdProvider);
  }
  if (StringUtils.hasLength(this.properties.getTypeAliasesPackage())) {
    factory.setTypeAliasesPackage(this.properties.getTypeAliasesPackage());
  }
  if (this.properties.getTypeAliasesSuperType() != null) {
    factory.setTypeAliasesSuperType(this.properties.getTypeAliasesSuperType());
  }
  if (StringUtils.hasLength(this.properties.getTypeHandlersPackage())) {
    factory.setTypeHandlersPackage(this.properties.getTypeHandlersPackage());
  }
  if (!ObjectUtils.isEmpty(this.typeHandlers)) {
    factory.setTypeHandlers(this.typeHandlers);
  }
  if (!ObjectUtils.isEmpty(this.properties.resolveMapperLocations())) {
    factory.setMapperLocations(this.properties.resolveMapperLocations());
  }
  Set<String> factoryPropertyNames = Stream
      .of(new BeanWrapperImpl(SqlSessionFactoryBean.class).getPropertyDescriptors()).map(PropertyDescriptor::getName)
      .collect(Collectors.toSet());
  Class<? extends LanguageDriver> defaultLanguageDriver = this.properties.getDefaultScriptingLanguageDriver();
  if (factoryPropertyNames.contains("scriptingLanguageDrivers") && !ObjectUtils.isEmpty(this.languageDrivers)) {
    // Need to mybatis-spring 2.0.2+
    factory.setScriptingLanguageDrivers(this.languageDrivers);
    if (defaultLanguageDriver == null && this.languageDrivers.length == 1) {
      defaultLanguageDriver = this.languageDrivers[0].getClass();
    }
  }
  if (factoryPropertyNames.contains("defaultScriptingLanguageDriver")) {
    // Need to mybatis-spring 2.0.2+
    factory.setDefaultScriptingLanguageDriver(defaultLanguageDriver);
  }

  return factory.getObject();
}
```

#### e 常见问题

1. 注解失效

   ```xml
   添加jar包
   <!--缺少此jar包，导致@Mapper注解无效-->
           <dependency>
               <groupId>org.mybatis.spring.boot</groupId>
               <artifactId>mybatis-spring-boot-starter</artifactId>
               <version>1.2.0</version>
           </dependency>
   
   
   注意版本问题
   ```

   

2. Invalid bound statement (not found)

   ```
   添加如下配置
   mybatis.mapper-locations=classpath:mapper/*.xml
   ```

   
   
   SpringBoot 将Mapper 加入到容器中有两个方法：
   
   - @Mapper 注解
   
   - **包扫描 @MapperScan()**
   
     

# 四、日志

### 1.常见日志框架	

| 日志门面（抽象层）          | 日志实现                 |
| --------------------------- | ------------------------ |
| JCL,**SLF4J**,jboss-logging | Log4j,Log4j2,**Logback** |

SpringBoot 使用SLF4J +Logback

### 2.使用

#### a.导入jar

```
先导入Slf4j的jar包，之后根据实际情况导入实现jar包，可能需要适配的jar
```

#### b.编程应用

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```



<img src="/Users/mac/Desktop/屏幕快照 2021-03-07 16.51.12.png" style="zoom:50%;" />



```
每个日志实现的框架有自己的配资文件，使用Slf4j之后，配置文件还是做成日志实现框架本身的配置文件
```

#### c.问题

```
每一个框架都有自己的日志框架实现，就会导致日志框架更加不好控制。需要统一日志框架
解决思路就是将各个框架中的日志依赖经过多个适配的jar转换成slf4j，（适配器模式）
```



<img src="/Users/mac/Desktop/屏幕快照 2021-03-07 17.02.21.png" style="zoom:50%;" />

```
实现方法：
1.将其他框架中的日志框架排除
2.用中间包来替换原来的日志框架
3，导入Slf4j，及其实现
```

#### d. SpringBoot如何使用日志框架

引入spring-boot-starter 或者spring-boot-starter-web的时候会默认引入spring-boot-starter-logging

```xml
引入jar
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-logging</artifactId>
  <version>2.3.7.RELEASE</version>
  <scope>compile</scope>
</dependency>
```

<img src="/Users/mac/Desktop/屏幕快照 2021-03-07 17.29.04.png" style="zoom:50%;" />

```markdown
总结：
## SpringBoot底层也是使用slf4j+logback 
## SpringBoot 也把其他日志都替换成Slf4j
## 中间替换包
## 如果引入其他框架，一定要把这个框架的日志排除
```

<img src="/Users/mac/Desktop/屏幕快照 2021-03-07 17.34.49.png" style="zoom:50%;" />



#### e. 日志使用与配置

```java
Logger logger = LoggerFactory.getLogger(getClass());

    @Test
    public void testLogger(){
        logger.trace("这是trace消息");
        logger.debug("这是debug消息");
//        SpringBoot的默认级别是Info
        logger.info("这是info消息");
        logger.warn("这是warn消息");
        logger.error("这是error消息");
    }
```



| Logger.file | Logger.path | Example  | Description                    |
| ----------- | ----------- | -------- | ------------------------------ |
| None        | None        |          | 在控制台输出                   |
| 指定文件名  | None        | My.log   | 在当前目录下My.log中输出       |
| None        | 指定目录    | /var/log | 输出到指定目录的spring.log文件 |

```yaml
logging:
  level:
#    写成map形式
    root: debug
    org.springframework.web: DEBUG
#    日志输出的文件名
  file: log
#  日志输出的位置
  path: ./log
#   logback.xml路径，默认为classpath:logback.xml
  config: ./logback.xml
```



#### f.日志配置

```
给类路径下放上每个人日志框架的配置文件即可，
```

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

```
logback.xml就直接被日志框架识别啦
logback-spring.xml：日志框架不直接加载日志配置，由SpringBoot解析日志配置，
```

```
logback-spring.xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>
```

#### g.日志切换

##### 使用logf4j2

###### 1、引入jar包

```xml
如果是父子工程需要在子工程中将logback排除干净。

<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
    <exclusions><!-- 去掉springboot默认配置 -->  
        <exclusion>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-logging</artifactId>  
        </exclusion>  
    </exclusions>  
</dependency> 

<dependency> <!-- 引入log4j2依赖 -->  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-log4j2</artifactId>  
</dependency>
```

```xml
在maven的父子工程中建议子工程不引入 spring-boot-starter
			<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                    <exclusion>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-starter-logging</artifactId>
                    </exclusion>
            </exclusions>
        </dependency>
```



###### 2、配置文件

```
logging:
  config: xxxx.xml
  level:
    cn.jay.repository: trace


默认名是log4j2-spring.xml
```

###### 3、配置文件模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出-->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
<configuration monitorInterval="5">
  <!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->

  <!--变量配置-->
  <Properties>
    <!-- 格式化输出：%date表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度 %msg：日志消息，%n是换行符-->
    <!-- %logger{36} 表示 Logger 名字最长36个字符 -->
    <property name="LOG_PATTERN" value="%date{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n" />
    <!-- 定义日志存储的路径 -->
    <property name="FILE_PATH" value="更换为你的日志路径" />
    <property name="FILE_NAME" value="更换为你的项目名" />
  </Properties>

  <appenders>

    <console name="Console" target="SYSTEM_OUT">
      <!--输出日志的格式-->
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <!--控制台只输出level及其以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
    </console>

    <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，适合临时测试用-->
    <File name="Filelog" fileName="${FILE_PATH}/test.log" append="false">
      <PatternLayout pattern="${LOG_PATTERN}"/>
    </File>

    <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
    <RollingFile name="RollingFileInfo" fileName="${FILE_PATH}/info.log" filePattern="${FILE_PATH}/${FILE_NAME}-INFO-%d{yyyy-MM-dd}_%i.log.gz">
      <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <Policies>
        <!--interval属性用来指定多久滚动一次，默认是1 hour-->
        <TimeBasedTriggeringPolicy interval="1"/>
        <SizeBasedTriggeringPolicy size="10MB"/>
      </Policies>
      <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
      <DefaultRolloverStrategy max="15"/>
    </RollingFile>

    <!-- 这个会打印出所有的warn及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
    <RollingFile name="RollingFileWarn" fileName="${FILE_PATH}/warn.log" filePattern="${FILE_PATH}/${FILE_NAME}-WARN-%d{yyyy-MM-dd}_%i.log.gz">
      <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <Policies>
        <!--interval属性用来指定多久滚动一次，默认是1 hour-->
        <TimeBasedTriggeringPolicy interval="1"/>
        <SizeBasedTriggeringPolicy size="10MB"/>
      </Policies>
      <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
      <DefaultRolloverStrategy max="15"/>
    </RollingFile>

    <!-- 这个会打印出所有的error及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
    <RollingFile name="RollingFileError" fileName="${FILE_PATH}/error.log" filePattern="${FILE_PATH}/${FILE_NAME}-ERROR-%d{yyyy-MM-dd}_%i.log.gz">
      <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <Policies>
        <!--interval属性用来指定多久滚动一次，默认是1 hour-->
        <TimeBasedTriggeringPolicy interval="1"/>
        <SizeBasedTriggeringPolicy size="10MB"/>
      </Policies>
      <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
      <DefaultRolloverStrategy max="15"/>
    </RollingFile>

  </appenders>

  <!--Logger节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。-->
  <!--然后定义loggers，只有定义了logger并引入的appender，appender才会生效-->
  <loggers>

    <!--过滤掉spring和mybatis的一些无用的DEBUG信息-->
    <logger name="org.mybatis" level="info" additivity="false">
      <AppenderRef ref="Console"/>
    </logger>
    <!--监控系统信息-->
    <!--若是additivity设为false，则 子Logger 只会在自己的appender里输出，而不会在 父Logger 的appender里输出。-->
    <Logger name="org.springframework" level="info" additivity="false">
      <AppenderRef ref="Console"/>
    </Logger>

    <root level="info">
      <appender-ref ref="Console"/>
      <appender-ref ref="Filelog"/>
      <appender-ref ref="RollingFileInfo"/>
      <appender-ref ref="RollingFileWarn"/>
      <appender-ref ref="RollingFileError"/>
    </root>
  </loggers>

</configuration>

```



```
trace：追踪，就是程序推进一下，可以写个trace输出
debug：调试，一般作为最低级别，trace基本不用。
info：输出重要的信息，使用较多
warn：警告，有些信息不是错误信息，但也要给程序员一些提示。
error：错误信息。用的也很多。
fatal：致命错误
```

##### 使用slf4j+log4j

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>logback-classic</groupId>
            <artifactId>ch.qos.logback</artifactId>
        </exclusion>
        <exclusion>
            <groupId>log4j-over-slf4j</groupId>
            <artifactId>org.slf4j</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
</dependency>
```

# 五、web



**SpringBoot 的自动配置原理很重要**

```
1、自动配置原理帮我们配置了哪些？   XXXAutoConfiguration帮我们自动配置组件，XXXXProperties：配置类来封装配置文件内容
2、能不能修改配置？
3、修改哪些配置
```



### 1、静态资源

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
  这个类指明了关于资源的配置项：
```



```java
		// 添加静态资源映射
		@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
			if (!registry.hasMappingForPattern("/webjars/**")) {
        // 配置访问的地址
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
                                             // 文件实际的位置
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
      // 以指定文件路径的方法来映射资源位置：根据路径配置的位置来找资源
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}


	 //配置欢迎页
		@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
			return welcomePageHandlerMapping;
		}
```

1） 如果请求以/webjar/** 方式，就会映射到这个路径下的文件 classpath:/META-INF/resources/webjars/ 

​	webjar :以jar包的方式引入静态资源

​	目标连接：http://localhost:8080/webjars/jquery/3.6.0/jquery.min.js **在连接一定要加入webjars**

```xml
<!--        引入jquery的webjar-->
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.6.0</version>
        </dependency>
```

2)    /** 访问当前项目的任何资源（静态资源文件夹）   /A = classpath:/META-INF/resources/A

classpath 指的是main包下的 Java,resource包

```java
"classpath:/META-INF/resources/",
"classpath:/resources/", 
"classpath:/static/", 
"classpath:/public/"
"/"  当前项目的根路径
```



 3) 欢迎页：静态资源文件夹下的index.html 都被/**映射 

4）配置图标 所有的**/favicon.ico 都在静态资源文件夹下找

```html
springboot在2.x版本之后需要在再静态资源文件上添加图标，并且在每一页都引入图标
<link rel="icon" href="favicon.ico" >
<!-- <link rel="icon" th:href="@{favicon.ico}" >  <!-- thymeleaf方式-->
```

5)

```yaml
spring:
  application:
    name: restful-web
  resources:
    static-locations: classpath:/hello
    
用static-locations 来指定静态资源文件夹的位置
```



### 2 、模板引擎

``` 
jsp ，Volocity,Freemarker,Thymeleaf（SpringBoot推荐）.
在页面中有动态表达式，模板引擎将数据填充到到动态表达式中
```

#### 1、引入



```xml
<!--引入thymeleaf 模板引擎-->
这个引用的组件的基本方法，但是如果组件还引入了其他的组件。需要注意适配
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
thymeleaf 的配置文件
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

  // 前缀
	public static final String DEFAULT_PREFIX = "classpath:/templates/";

  // 后缀
	public static final String DEFAULT_SUFFIX = ".html";

只要我们把页面放在templates文件夹下并且以".html" 结尾,thymelea引擎就会自动解析成静态html
```

#### 2、语法规则

```html
引入名称空间，后续会有提示
<html xmlns:th="http://www.thymeleaf.org">
```

##### 1、基本表达式

```properties
Variable Expressions: ${...} 
	获取值 ${person.father.name}
	使用函数 ${person.createCompleteNameWithSeparator('-')}
	使用内置的对象
    #ctx : the context object. 
    #vars: the context variables. 
    #locale : the context locale. 
    #request : (only in Web Contexts) the HttpServletRequest object. 
    #response : (only in Web Contexts) the HttpServletResponse object. 
    #session : (only in Web Contexts) the HttpSession object. 
    #servletContext : (only in Web Contexts) the ServletContext object.
	例：${#request.hello}
	使用简单的工具对象
	#execInfo : information about the template being processed. 
	#messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax. 
	#uris : methods for escaping parts of URLs/URIs Page 20 of 106
	#conversions : methods for executing the configured conversion service (if any). 
	#dates : methods for java.util.Date objects: formatting, component extraction, etc. 
	#calendars : analogous to #dates , but for java.util.Calendar objects. 
	#numbers : methods for formatting numeric objects.
  #strings : methods for String objects: contains, startsWith, prepending/appending, etc. 
	#objects : methods for objects in general. #bools : methods for boolean evaluation. #arrays : methods for arrays. 
	#lists : methods for lists. 
	#sets : methods for sets. 
	#maps : methods for maps.
  #aggregates : methods for creating aggregates on arrays or collections.
  #ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).
	
	
	 
Selection Variable Expressions: *{...} 
	等同于${...},
	简便获取对象的属性，*{}来代替对象获取属性
	<div th:object="${session.user}"> 
		<p>Name: <span th:text="*{firstName}">Sebastian</span>.</p> 
		<p>Surname: <span th:text="*{lastName}">Pepper</span>.</p> 
		<p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p> 
	</div>
Message Expressions: #{...} 
	获取国际化内容
Link URL Expressions: @{...} 
	标识连接：在括号内输入参数，以map形式输入
	<!-- Will produce 'http://localhost:8080/gtvg/order/details?orderId=3' (plus rewriting) --> 
	<a href="details.html" th:href="@{http://localhost:8080/gtvg/order/details(orderId=${o.id})}">view</a>
	
Fragment Expressions: ~{...}
	插入片段
	<div th:insert="~{commons :: main}">...</div>
	
	<div th:with="frag=~{footer :: #main/text()}"> 
		<p th:insert="${frag}">
  </div>
```

##### 2、数据结构

```properties
Literals 
  Text literals: 'one text' , 'Another one!' ,… 
  Number literals: 0 , 34 , 3.0 , 12.3 ,… 
  Boolean literals: true , false 
  Null literal: null 
  Literal tokens: one , sometext , main ,… 
Text operations: 
	String concatenation: + 
	Literal substitutions: |The name is ${name}| 
Arithmetic operations: 
  Binary operators: + , - , * , / , % 
  Minus sign (unary operator): - 
  Boolean operations: 
  Binary operators: and , or 
  Boolean negation (unary operator): ! , not 
  Comparisons and equality: Comparators: > , < , >= , <= ( gt , lt , ge , le ) 
  Equality operators: == , != ( eq , ne ) 
Conditional operators: 
	If-then: (if) ? (then) 
	If-then-else: (if) ? (then) : (else)
  Default: (value) ?: (defaultvalue) 
Special tokens: 
	No-Operation: _
```

##### 3、优先级

<img src="/Users/mac/Pictures/thymeleaf优先级.png" style="zoom:50%;" />

## 3、Filter

```
它主要用于对用户请求进行预处理和后处理，拥有一个典型的处理链。
使用Filter完整的流程是：Filter对用户请求进行预处理，接着将请求交给Servlet进行预处理并生成响应，最后Filter再对服务器响应进行后处理。

作用：
 * 1) Authentication Filters, 即用户访问权限过滤
 * 2) Logging and Auditing Filters, 日志过滤，可以记录特殊用户的特殊请求的记录等
 * 3) Image conversion Filters
 * 4) Data compression Filters <br>
 * 5) Encryption Filters <br>
 * 6) Tokenizing Filters <br>
 * 7) Filters that trigger resource access events <br>
 * 8) XSL/T filters <br>
 * 9) Mime-type chain Filter <br>
 
自定义的过滤器都必须实现javax.Servlet.Filter接口，并重写接口中定义的三个方法：

	void init(FilterConfig config)
	用于完成Filter的初始化。
    void destory()
    用于Filter销毁前，完成某些资源的回收。
    void doFilter(ServletRequest request,ServletResponse response,FilterChain chain)
    实现过滤功能，即对每个请求及响应增加的额外的预处理和后处理。,执行该方法之前，即对用户请求进行预处理；执行该方法之后，即对服务器响应进行后处理。值得注意的是，chain.doFilter()方法执行之前为预处理阶段，该方法执行结束即代表用户的请求已经得到控制器处理。因此，如果再doFilter中忘记调用chain.doFilter()方法，则用户的请求将得不到处理。

此时需要在Config中注册Bean
@Configuration
public class WebConfig {
    /**
     * 注册第三方过滤器
     * 功能与spring mvc中通过配置web.xml相同
     * @return
     */
    @Bean
    public FilterRegistrationBean thirdFilter(){
        ThirdPartFilter thirdPartFilter = new ThirdPartFilter();
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean() ;

        FilterRegistrationBean<MultipartFilter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(thirdPartFilter);
        // 设置优先级别
        filterRegistrationBean.setOrder(MULTIPART_FILTER_ORDER);
        // 设置匹配的路径 “/*”表示匹配全部路径
        filterRegistrationBean.addUrlPatterns("/*");
        // name
        filterRegistrationBean.setName("multipartFilter");
        
        return filterRegistrationBean;
    }
}

```

## 4、Interceptor

```
preHandler(HttpServletRequest request, HttpServletResponse response, Object handler)
方法将在请求处理之前进行调用。SpringMVC中的Interceptor同Filter一样都是链式调用。
每个Interceptor的调用会依据它的声明顺序依次执行，而且最先执行的都是Interceptor中的preHandle方法，所以可以在这个方法中进行一些前置初始化操作或者是对当前请求的一个预处理，也可以在这个方法中进行一些判断来决定请求是否要继续进行下去。
该方法的返回值是布尔值Boolean 类型的，当它返回为false时，表示请求结束，后续的Interceptor和Controller都不会再执行；当返回值为true时就会继续调用下一个Interceptor 的preHandle 方法，
如果已经是最后一个Interceptor 的时候就会是调用当前请求的Controller 方法。

postHandler(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
在当前请求进行处理之后，也就是Controller 方法调用之后执行，但是它会在DispatcherServlet 进行视图返回渲染之前被调用，
所以我们可以在这个方法中对Controller 处理之后的ModelAndView 对象进行操作。

afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handle, Exception ex)
该方法也是需要当前对应的Interceptor的preHandle方法的返回值为true时才会执行。
该方法将在整个请求结束之后，也就是在DispatcherServlet 渲染了对应的视图之后执行。这个方法的主要作用是用于进行资源清理工作的。


```

```
三者的调用顺序Filter->Intercepto->Aspect->Controller
当Controller抛出的异常的处理顺序则是从内到外的。因此我们总是定义一个注解@ControllerAdvice去统一处理控制器抛出的异常。

这些方法适合对请求做统一处理，意味着返回的结果是相同的，如果多种类型的消息就不方便整理输出
```

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-23 15.20.47.png" style="zoom:50%;" />



# 六、校验

```java
@Validated：作用在类、方法和参数上
```

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-20 16.28.34.png" style="zoom:50%;" />

```xml
引入web模块后会自定引入
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

# 七、注解积累

##  @ControllerAdvice

```
对Controller进行增强

使用这个 Controller ，可以实现三个方面的功能：
1、全局异常处理
2、全局数据绑定
3、全局数据预处理
```

### 1、全局异常处理

```java
@ControllerAdvice
public class MyGlobalExceptionHandler {
  // 专门处理的异常类
  // 参数：异常
    @ExceptionHandler(Exception.class)
    public ModelAndView customException(Exception e) {
        ModelAndView mv = new ModelAndView();
        mv.addObject("message", e.getMessage());
        mv.setViewName("myerror");
        return mv;
    }
}
```

### 2、全局数据绑定

```java
全局数据绑定功能可以用来做一些初始化的数据操作，我们可以将一些公共的数据定义在添加了 @ControllerAdvice 注解的类中，这样，
在每一个 Controller 的接口中，就都能够访问导致这些数据。
  
@ControllerAdvice
public class MyGlobalExceptionHandler {
    @ModelAttribute(name = "md")
    public Map<String,Object> mydata() {
        HashMap<String, Object> map = new HashMap<>();
        map.put("age", 99);
        map.put("gender", "男");
        return map;
    }
}

md:Map
使用 @ModelAttribute 注解标记该方法的返回数据是一个全局数据，
  默认情况下，这个全局数据的 key 就是返回的变量名，value 就是方法返回值，
  当然开发者可以通过 @ModelAttribute 注解的 name 属性去重新指定 key。
```

### 3、全局数据预处理

## @ConditionalOnProperty

```java
@ConditionalOnProperty注解来控制@Configuration是否生效.
  
@Configuration
@ConditionalOnProperty(prefix = "filter",name = "loginFilter",havingValue = "true")
public class FilterConfig {
	//prefix为配置文件中的前缀,
	//name为配置的名字
	//havingValue是与配置的值对比值,当两个值相同返回true,配置类生效.
  // matchIfMissing 指定如果未设置属性，条件是否应匹配
    @Bean
    public FilterRegistrationBean getFilterRegistration() {
        FilterRegistrationBean filterRegistration  = new FilterRegistrationBean(new LoginFilter());
        filterRegistration.addUrlPatterns("/*");
        return filterRegistration;
    }
}
```

##  @Scheduled

```
定时执行任务，需要配置Cron表达式
```

## @RequestBody

```markdown
SpringMVC中处理请求参数有好几种不同的方式，如我们常见的下面几种

根据 HttpServletRequest 对象获取
根据 @PathVariable 注解获取url参数
根据 @RequestParam 注解获取请求参数 
根据Bean的方式获取请求参数
根据 @ModelAttribute 注解获取请求参数
@RequestBody注解常用来处理content-type不是默认的application/x-www-form-urlcoded编码的内容，获取请求体中的数据
比如说：application/json或者是application/xml等。

## content-type 
MediaType，即是Internet Media Type，互联网媒体类型；也叫做MIME类型，在Http协议消息头中，使用Content-Type来表示具体请求中的媒体类型信息。

常见媒体格式如下:

text/html ： HTML格式
text/plain ：纯文本格式
text/xml ：  XML格式
image/gif ：gif图片格式
image/jpeg ：jpg图片格式
image/png：png图片格式

以application开头的媒体格式类型：

application/xhtml+xml ：XHTML格式
application/xml     ： XML数据格式
application/atom+xml  ：Atom XML聚合格式
application/json    ： JSON数据格式
application/pdf       ：pdf格式
application/msword  ： Word文档格式
application/octet-stream ： 二进制流数据（如常见的文件下载）
application/x-www-form-urlencoded ： 中默认的encType，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）,标准的编码格式


对于前端使用而言，form表单的enctype属性为编码方式，
常用有两种：application/x-www-form-urlencoded和multipart/form-data，默认为application/x-www-form-urlencoded。


如果后端使用@RequestBody，需要在前端设置 content-type 来指定媒体类型（通常为json）格式，通常使用POST请求


如果后端接收参数是一个对象，且该参数是用@RequestBody修饰的，那么前端json传数据，要满足：

后端将http输入流装配到目标类时，会根据json字符串中的key来匹配对应实体类的属性，如果匹配一致且json中的key对应符合，那么后端能成功接收
json字符串中，如果value为""的话，后端对应属性如果是String类型的，那么接受到的就是""，如果后端对应的是引用类型Integer、Double等的话，那么就收的就是null
json字符串中，如果value为null的话，后端接收到的就是null
如果某个参数没有value，在传json给后端的时候，要么干脆不该把盖子端写到json中，要么就给value赋值""或null

***如果参数放在请求体中，传入后台需要用@RequestBody进行接收；如果不是放在请求体中的话，那么后台接收前台传过来的参数时，要用@RequestParam进行对应接收。***
```

## @ResponseBody

```
@ResponseBody的作用其实是将java对象转为json格式的数据。

@responseBody注解的作用是将controller的方法返回的对象通过适当的转换器转换为指定的格式之后，写入到response对象的body区，通常用来返回JSON数据或者是XML数据。
注意：在使用此注解之后不会再走视图处理器，而是直接将数据写入到输入流中，他的效果等同于通过response对象输出指定格式的数据。

@ResponseBody是作用在方法上的，@ResponseBody 表示该方法的返回结果直接写入 HTTP response body 中，一般在异步获取数据时使用【也就是AJAX】。
```

## @JsonInclude

```
返回前端的实体类中如果某个字段为空的话那么就不返回这个字段了
在实体类序列化成json的时候在使用不同的策略
JsonJsonInclude.Include.NON_NULL这个最常用，即如果加该注解的字段为null,那么就不序列化这个字段了。
```

## @Autowired

```
@Autowired注入是按照类型注入的，只要配置文件中的bean类型和需要的bean类型是一致的，这时候注入就没问题。但是如果相同类型的bean不止一个，此时注入就会出现问题，Spring容器无法启动。 

如果容器中没有一个和标注变量类型匹配的Bean，Spring容器启动时将报NoSuchBeanDefinitionException的异常。如果希望Spring即使找不到匹配的Bean完成注入也不用抛出异常，那么可以使用@Autowired(required=false)进行标注：

可以放在方法的参数上表示对方法的参数进行注入依赖

```

## @Resource

```
@Resourced标签是按照bean的名字来进行注入的，如果我们没有在使用@Resource时指定bean的名字，同时Spring容器中又没有该名字的bean,这时候@Resource就会退化为@Autowired即按照类型注入，这样就有可能违背了使用@Resource的初衷。所以建议在使用@Resource时都显示指定一下bean的名字@Resource(name="xxx") 
```















































# 八、创建Bean

```java
@Bean
    public DefaultMqListener flowProcessListener() {
        return new DefaultMqListener("flow_process");
    }

    @Bean
    public DefaultMqListener employeeListener() {
        return new DefaultMqListener("employee");
    }

这个是属于同类型，不同实例名。
使用@Resource(name="")俩实现依赖注入
```

```
bean在容器外部创建，对于这种生命周期不完全受容器控制的bean,其中的@Autowired/@Value注解缺省是不生效的，换句话讲，对于这种bean,容器不会对它们进行自动注入。


SpringBeanAutowiringSupport就是框架提供的一个这样的工具类，用于手工触发一个bean的依赖注入
	public static void processInjectionBasedOnCurrentContext(Object target)
	该方法会使用ContextLoader.getCurrentWebApplicationContext()获取当前Web应用上下文，然后对目标bean target触发依赖注入。

	public static void processInjectionBasedOnServletContext(Object target, ServletContext servletContext)
	该方法会使用指定的servletContext,获取它对应的Web应用上下文，然后对目标bean target触发依赖注入。
```



# 九、Order

```
Ordered接口，顾名思义，就是用来排序的。
Spring是一个大量使用策略设计模式的框架，这意味着有很多相同接口的实现类，那么必定会有优先级的问题。
PriorityOrdered是个接口，继承自Ordered接口，未定义任何方法。

   1. 若对象o1是Ordered接口类型，o2是PriorityOrdered接口类型，那么o2的优先级高于o1

　　2. 若对象o1是PriorityOrdered接口类型，o2是Ordered接口类型，那么o1的优先级高于o2

　　3. 其他情况，若两者都是Ordered接口类型或两者都是PriorityOrdered接口类型，调用Ordered接口的getOrder方法得到order值，order值越大，优先级越小

```

# 十、 整合Mybatis

```
如果想产看SQL语句的执行情况，可以在配置文件中控制mapper接口的日志输出级别


logging.level.com.secbro.mapper=debug

“logging.level.”为前缀，
“com.secbro.mapper”为Mapper接口所在的包路径。对应的value值为日志的级别。
```

# 十一、上传下载文件

## 11.1 上传文件

```
上传文件：
对于文件上传，控制器中对应的上传方法中要包含MultipartFile对象，MultipartFile对象可以是一个数组对象，也可以是单个对象，如果是一个数组对象，则可以进行多文件上传

方法一：从request流中获取对象。需要指明对象名称 获取到的对象是 MultipartFile，之后转化成File对象。
方法二：直接传递MultipartFile对象，此时需要指定请求的类型   

整体上就是借助流的操作来处理文件对象。
```

```
文件上传的时候需要编写一个页面，通过from 表单提交post 请求，并制定上报的数据是文件格式。
想要通过http请求 获取的页面。首先不能指定返回的数据是json 格式。不要加    @ResponseBody 注解。
并且需要在appliaction.yaml 文件中指定 模版的位置。并且在pom.xml文件中指定thymeleaf 作为模版引擎
```

1、pom.xml

```xml
<!--页面模版引擎-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
```

2、在resource目录下创建templates 文件夹来存储模版

3、指定模版位置

```yaml
spring:
  thymeleaf:
  # 需要带“/” 指明文件内
      prefix: classpath:/templates/
      suffix: .html
```

4、上传页面

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>Spring Boot file upload example</h1>
  <!--post 请求，enctype文件格式-->
    <form method="POST" action="/upload" enctype="multipart/form-data">
        <input type="file" name="file"/><br/><br/>
        <input type="submit" value="Submit"/>
    </form>
</body>
</html>
```

5、controller

```java
@Controller
// 在控制台输出
@Slf4j
public class UploadFileController {

    public static final String UPLOADED_FOLDER = "/Users/peilizhi/file/";

    @GetMapping("/")
    public String index() {
        return "upload";
    }


    /**
     *
     * @param multipartFile 上传的文件对象
     * @param redirectAttributes 重定向
     * @return
     */
    @PostMapping("/upload")
    @ResponseBody
    public String singleFileUpload(@RequestParam("file") MultipartFile multipartFile,
                                   RedirectAttributes redirectAttributes) {

        if (multipartFile.isEmpty()) {
            redirectAttributes.addFlashAttribute("message", "Please select a file to upload");
            log.warn("文件数据为空");
            return "uploadStatus";
        }

        try {

            // Get the file and save it somewhere
            byte[] bytes = multipartFile.getBytes();
            // 存储数据
            Path path = Paths.get(UPLOADED_FOLDER + multipartFile.getOriginalFilename());
          	// 文件存储
            Files.write(path, bytes);

            log.info("成功保存文件,文件名:{}",multipartFile.getOriginalFilename());

            redirectAttributes.addFlashAttribute("message",
                    "You successfully uploaded '" + multipartFile.getOriginalFilename() + "'");

        } catch (IOException e) {
            e.printStackTrace();
        }

        return "uploadStatus";
    }
}
```

**有时间的会出现文件写不进去的，是因为目录没有开启写权限**

chmod 777 xxx   [比较绝对]

## 11.2 下载文件

```java
@GetMapping("/downloadLocal")
    public void downloadLocal(String path, HttpServletResponse response) throws IOException {
        // 将文件读到流中
        InputStream inputStream = new FileInputStream(path);
        // 颠倒
        response.reset();
        response.setContentType("application/octet-stream");
        String fileName = new File(path).getName();
        // 设置响应头
        response.setHeader("Content-Disposition", "attachment; filename=" + URLEncoder.encode(fileName, "UTF-8"));
        // 获取响应的输出
        ServletOutputStream outputStream = response.getOutputStream();
        final byte[] bytes = new byte[1024];
        int len;
        while ((len = inputStream.read(bytes)) > 0) {
            outputStream.write(bytes, 0, len);
        }
        inputStream.close();
    }
```

**将文件通过响应流输出过去就可以。需要配置响应流**

**返回前端的文件名必须进行URL编码**

**文件名有特殊字符都不行的**

# 十二、Test

SpringBoot-web 下默认引入test 包

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

包含的类：<img src="https://ask.qcloudimg.com/http-save/yehe-1305760/n8jrsi1fao.png?imageView2/2/w/1620" alt="img" style="zoom:90%;" />

## Service、Dao 的单元测试

Idea 选择类，之后再选择 Navigate -》Test 会自动针对当前类生成测试类

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220429180500878.png" alt="image-20220429180500878" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220429180836140.png" alt="image-20220429180836140" style="zoom:50%;" />

```java
/**
 * @author by peilizhi
 * @date 2022/4/29 17:11
 */
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.NONE)
// 如果想使用junit4 的话，可以加入@RunWith，
// 想使用junit5 的话，直接@SpringBootTest就可以
@RunWith(SpringRunner.class)
class RedisCommonDAOTest {

    @Resource
    private RedisCommonDAO redisCommonDAO;

    @Test
    void isKeyExist() {
        boolean exist = redisCommonDAO.isKeyExist("xiaohong");
        System.out.println("exist = " + exist);
    }
}
```



## @SpringBootTest

会加载ApplicationContext，启动spring容器

使用@SpringBootTest时并没有像@ContextConfiguration一样显示指定locations或classes属性，原因在于@SpringBootTest注解会自动检索程序的配置文件，检索顺序是从当前包开始，逐级向上查找被@SpringBootApplication或@SpringBootConfiguration注解的类。

- value 指定配置属性
- properties 指定配置属性，和value意义相同
- classes 指定配置类，等同于@ContextConfiguration中的class，若没有显示指定，将查找嵌套的@Configuration类，然后返回到SpringBootConfiguration搜索配置
- webEnvironment 指定web环境，可选值有：MOCK、RANDOM_PORT、DEFINED_PORT、NONE

`webEnvironment`详细说明：

- MOCK 此值为默认值，该类型提供一个mock环境，此时内嵌的服务（servlet容器）并没有真正启动，也不会监听web端口。

- RANDOM_PORT 启动一个真实的web服务，监听一个随机端口。

- DEFINED_PORT 启动一个真实的web服务，监听一个定义好的端口（从配置中读取）。

- NONE 启动一个非web的ApplicationContext，既不提供mock环境，也不提供真是的web服务。

  

@SpringBootTest vs @WebMvcTest(或@*Test)

- 都可以启动Spring的ApplicationContext
- @SpringBootTest自动侦测并加载@SpringBootApplication或@SpringBootConfiguration中的配置，@WebMvcTest不侦测配置，只是默认加载一些自动配置。
- @SpringBootTest测试范围一般比@WebMvcTest大。

```java

/**
* 指定额外的容器来执行测试
 * @author peilizhi
 * @date 2022/5/13 09:10
 **/
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {BeanConfig.class})
public class ConfigTest {
    @Autowired
    Person person;

    @Test
    public void test() {
        System.out.println("person = " + person);
    }
}
```

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@BootstrapWith(SpringBootTestContextBootstrapper.class)
@ExtendWith(SpringExtension.class)
public @interface SpringBootTest {
  @AliasFor("properties")
	String[] value() default {};

	/**
	 */
	@AliasFor("value")
	String[] properties() default {};

	/**
	 */
	String[] args() default {};

	/**
	 */
	Class<?>[] classes() default {};

	/**
	 */
	WebEnvironment webEnvironment() default WebEnvironment.MOCK;

}
```

@SpringBootTest 注解里面就默认带了ExtendWith ，支持junit5

SpringExtension将Spring TestContext Framework集成到 JUnit 5 的Jupiter编程模型中。

## @RunWith

@RunWith是Junit4提供的注解，将Spring和Junit链接了起来。假如使用Junit5，不再需要使用@ExtendWith注解，@SpringBootTest和其它@*Test默认已经包含了该注解。



## 切片测试

切片指一些在特定环境下才能执行的模块，比如MVC中的Controller、JDBC数据库访问、Redis客户端等，这些模块大多脱离特定环境后不能独立运行，假如spring没有为此提供测试支持，开发者只能启动完整服务对这些模块进行测试，

通过@*Test开启具体模块的测试支持，开启后spring仅加载相关的bean，无关内容不会被加载

## MvcTest

```java
@WebMvcTest
class ExcelControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    void exportHssf() throws Exception {
        MvcResult result = mvc.perform(MockMvcRequestBuilders
                        .post("/excel/export-hssf"))
                .andReturn();
    }

    @Test
    void exportXssfData() {
    }

    @Test
    void exportEasyPoi() {
    }

    @Test
    void exportEasyExcel() {
    }
}
```

使用@WebMvcTest和MockMvc搭配使用，可以在不启动web容器的情况下，对Controller进行测试（注意：仅仅只是对controller进行简单的测试，如果Controller中依赖用@Autowired注入的service、dao等则不能这样测试）

 

所有的@*Test注解都被@BootstrapWith注解，它们可以启动ApplicationContext，是测试的入口，所有的测试类必须声明一个@*Test注解。

- @SpringBootTest 自动侦测并加载@SpringBootApplication或@SpringBootConfiguration中的配置，默认web环境为MOCK，不监听任务端口
- @DataRedisTest 测试对Redis操作，自动扫描被@RedisHash描述的类，并配置Spring Data Redis的库
- @DataJpaTest 测试基于JPA的数据库操作，同时提供了TestEntityManager替代JPA的EntityManager 
- @DataJdbcTest 测试基于Spring Data JDBC的数据库操作
- @JsonTest 测试JSON的序列化和反序列化
- @WebMvcTest 测试Spring MVC中的controllers
- @WebFluxTest 测试Spring WebFlux中的controllers
- @RestClientTest 测试对REST客户端的操作
- @DataLdapTest 测试对LDAP的操作
- @DataMongoTest 测试对MongoDB的操作
- @DataNeo4jTest 测试对Neo4j的操作



# 十三 获取配置文件参数

## 13.1 @Value 读取配置文件参数

只适合当前类，不是很通用

```java
@SpringBootTest(classes = TestConsumerApplication.class)
@Slf4j
class TestMangerImplTest {

    @Value("${sys.platform}")
    private String name ;

    @Resource
    private TestService testService;
```

## 13.2 Environment 读取配置文件参数

```java
import org.springframework.core.env.Environment;


@Autowired
private Environment environment;


@Test
    void listAll2()  {
        String s = environment.getProperty("sys.platform");
        System.out.println("s = " + s);
    }
```

# 13.3 @ConfigurationProperties 读取配置文件参数

```java
@Data
@Component
@ConfigurationProperties(prefix = "sys")
@PropertySource("classpath:application.yaml")
public class EnvironmentOperations {

    public String platform;
}
```





# 十四、其他

## 接口多实现类注入

默认情况，springboot注入的时候，是直接找接口的实现类，如果有多个实现类的时候，就不能确实要找的是哪个

1. @Primary 

   ```java
   @Bean("doSomethingExecutor")
       @Primary
       public Executor doSomethingExecutor() {
           ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
           // 核心线程数：线程池创建时候初始化的线程数
           executor.setCorePoolSize(10);
           // 最大线程数：线程池最大的线程数，只有在缓冲队列满了之后才会申请超过核心线程数的线程
           executor.setMaxPoolSize(10);
           // 缓冲队列：用来缓冲执行任务的队列
           executor.setQueueCapacity(500);
           // 允许线程的空闲时间60秒：当超过了核心线程之外的线程在空闲时间到达之后会被销毁
           executor.setKeepAliveSeconds(60);
           // 线程池名的前缀：设置好了之后可以方便我们定位处理任务所在的线程池
           executor.setThreadNamePrefix("do-something-");
           // 设置拒绝策略  如果线程池没有Shutdown，则直接调用Runnable任务的run()方法。
           executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
           executor.initialize();
           return executor;
       }
   
   
       @Bean("annotionExecutor")
       public Executor annotionExecutor() {
           ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
           // 核心线程数：线程池创建时候初始化的线程数
           executor.setCorePoolSize(10);
           // 最大线程数：线程池最大的线程数，只有在缓冲队列满了之后才会申请超过核心线程数的线程
           executor.setMaxPoolSize(10);
           // 缓冲队列：用来缓冲执行任务的队列
           executor.setQueueCapacity(500);
           // 允许线程的空闲时间60秒：当超过了核心线程之外的线程在空闲时间到达之后会被销毁
           executor.setKeepAliveSeconds(60);
           // 线程池名的前缀：设置好了之后可以方便我们定位处理任务所在的线程池
           executor.setThreadNamePrefix("annotion-");
           // 设置拒绝策略  如果线程池没有Shutdown，则直接调用Runnable任务的run()方法。
           executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
           executor.initialize();
           return executor;
       }
   ```

   2. @Resource 

      使用bean的名称来注入，使用的时候需要指定想使用bean的容器内的名称，默认是首字母小写

      ```java
      @Resource
          private Executor doSomethingExecutor;
      ```

   3. @Qualifier

      同@Resource  都是指定名称

      ```java
      @Resource
      @Qualifier("doSomethingExecutor")
      private Executor executor;
      ```

   4. 不同的环境使用不同实现类
   
      ```java
       @Bean("doSomethingExecutor")
       @ConditionalOnProperty(value = "deploy.province", havingValue = "doSomethingExecutor")
          public Executor doSomethingExecutor() {
              ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
              // 核心线程数：线程池创建时候初始化的线程数
              executor.setCorePoolSize(10);
              // 最大线程数：线程池最大的线程数，只有在缓冲队列满了之后才会申请超过核心线程数的线程
              executor.setMaxPoolSize(10);
              // 缓冲队列：用来缓冲执行任务的队列
              executor.setQueueCapacity(500);
              // 允许线程的空闲时间60秒：当超过了核心线程之外的线程在空闲时间到达之后会被销毁
              executor.setKeepAliveSeconds(60);
              // 线程池名的前缀：设置好了之后可以方便我们定位处理任务所在的线程池
              executor.setThreadNamePrefix("do-something-");
              // 设置拒绝策略  如果线程池没有Shutdown，则直接调用Runnable任务的run()方法。
              executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
              executor.initialize();
              return executor;
          }
      
      
          @Bean("annotionExecutor")
          @ConditionalOnProperty(value = "deploy.province", havingValue = "annotionExecutor")
          public Executor annotionExecutor() {
              ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
              // 核心线程数：线程池创建时候初始化的线程数
              executor.setCorePoolSize(10);
              // 最大线程数：线程池最大的线程数，只有在缓冲队列满了之后才会申请超过核心线程数的线程
              executor.setMaxPoolSize(10);
              // 缓冲队列：用来缓冲执行任务的队列
              executor.setQueueCapacity(500);
              // 允许线程的空闲时间60秒：当超过了核心线程之外的线程在空闲时间到达之后会被销毁
              executor.setKeepAliveSeconds(60);
              // 线程池名的前缀：设置好了之后可以方便我们定位处理任务所在的线程池
              executor.setThreadNamePrefix("annotion-");
              // 设置拒绝策略  如果线程池没有Shutdown，则直接调用Runnable任务的run()方法。
              executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
              executor.initialize();
              return executor;
          }
      ```
   
      不同的类指定不同的参数值，在不同的环境下，参数有不同的值
   
      ```application.yaml
      deploy:
        province: doSomethingExecutor
      ```
   
      ```application-test.yaml
      deploy:
        province: annotionExecutor
      ```
   
      在启动参数中指定环境 <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221019114005984.png" alt="image-20221019114005984" style="zoom:50%;" />
