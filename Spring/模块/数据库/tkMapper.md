```
tkMapper 可以快速编写sql,并能够根据数据库中的表结构来快速生成代码，不需要进行复杂的编写。
```

# 一、pom.xml

```xml
<properties>
        <java.version>1.8</java.version>
        <mysql.version>8.0.16</mysql.version>
        <mybatis-spring-boot-starter.version>1.3.2</mybatis-spring-boot-starter.version>
        <tkmybatis.version>2.1.5</tkmybatis.version>
        <pagehelper.version>1.2.7</pagehelper.version>
        <druid.version>1.1.16</druid.version>
        <persistence-api.version>1.0</persistence-api.version>
        <mybatis-generator-core.version>1.3.7</mybatis-generator-core.version>
    </properties>

<dependencies>

       <!--包含自定义字段映射关系 处理mysql与java字段-->
        <dependency>
            <groupId>com.huochai</groupId>
            <artifactId>mybatis-generator-self</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>

        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>

        <!--druid数据源-->
  			<dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid.version}</version>
        </dependency>
        <!-- Spring Boot Mybatis 依赖 -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <!--tk mybatis-->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
            <version>${tkmybatis.version}</version>
        </dependency>
        <!--pagehelper-->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>${pagehelper.version}</version>
        </dependency>
  
</dependencies>


<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
          
          
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.7</version>
                <configuration>
                    <configurationFile>
                        ${basedir}/src/main/resources/generatorConfig.xml
                    </configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>${mysql.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>tk.mybatis</groupId>
                        <artifactId>mapper-generator</artifactId>
                        <version>1.0.5</version>
                    </dependency>
                  <!-- 包含定义MyJavaTypeResolver-->
                    <dependency>
                        <groupId>com.huochai</groupId>
                        <artifactId>mybatis-generator-self</artifactId>
                        <version>0.0.1-SNAPSHOT</version>
                        <exclusions>
                            <exclusion>
                                <groupId>org.mybatis.generator</groupId>
                                <artifactId>mybatis-generator-core</artifactId>
                            </exclusion>
                        </exclusions>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>


```

# 二、数据库连接配置

```yaml
spring:
  profiles:
    active: test
  datasource:
    password: Admin123@
    username: huochai
    url: jdbc:mysql://47.96.235.128:3306/student?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&characterEncoding=utf-8&useSSL=false
    driver-class-name: com.mysql.cj.jdbc.Driver
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
# mybatis扫描 路径
mybatis:
  mapper-locations: classpath:mapper/*Mapper.xml

```

## 三、配置文件

文件名为generator.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >
<generatorConfiguration>

    <!--加载外部配置项或者配置文件,   不要引入.yaml格式，mybatis解析不了
    resource 属性，从classpath开始查找
    -->
    <properties resource="mysql.properties"/>

    <!--
    classPathEntry: 引入项目需要的依赖包 -->
    <!-- <classPathEntry
             location="D:\程序\mysql-5.6.26-winx64\mysql-connector-java-5.1.17.jar" />-->
    <!--
    id:上下文id
    targetRuntime:MyBatis3Simple 不生成dao方法
    defaultModelType: flat 所有字段（主键、blob 等）全部生成在一个对象中。
    -->
    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <!-- TKmybatis配置 -->

        <!--property用于为代码生成指定属性-->
        <property name="javaFileEncoding" value="UTF-8"/>
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
        <!--  <plugin type="${mapper.plugin}">
              <property name="mappers" value="${mapper.Mapper}"/>
          </plugin>-->

        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="com.huochai.repository.BaseMapper"/>
        </plugin>
        <!-- 使生成的 Model 实现 Serializable 接口  -->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <!--  为生成的 Model 覆写 toString() 方法 -->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>
        <!--  为生成的 Model 覆写 equals() 和 hashCode() 方法 -->
        <plugin type="org.mybatis.generator.plugins.EqualsHashCodePlugin"/>


        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="false"/>
            <!--  生成的注释中不包含时间信息，默认为 false -->
            <property name="suppressDate" value="true"/>

        </commentGenerator>
        <!-- 数据库链接URL、用户名、密码 -->
        <jdbcConnection driverClass="${mysql.driverClass}"
                        connectionURL="${mysql.url}" userId="${mysql.username}"
                        password="${mysql.password}"/>

        <!-- tinyInt转换为Integer   不会扫描当前项目,需要在别的jar中定义,再引入 -->
                <javaTypeResolver type="com.huochai.mybatis.MyJavaTypeResolver"/>



        <!-- 生成模型的包名和位置 -->
        <javaModelGenerator targetPackage="${mybatis.poPackage}" targetProject="${mybatis.javaProject}"/>
        <!-- 生成的映射文件包名和位置 -->
        <sqlMapGenerator targetPackage="${mybatis.mapperPackage}" targetProject="${mybatis.resources}"/>
        <!-- 生成service的包名和位置 -->
        <javaClientGenerator targetPackage="${mybatis.servicePackage}" targetProject="${mybatis.javaProject}"
                             type="XMLMAPPER"/>

        <!--数据库表-->
        <table tableName="sys_student" domainObjectName="StudentPO" mapperName="StudentMapper">
            <generatedKey column="id" sqlStatement="Mysql" identity="true"/>
        </table>

        <table tableName="sys_user" domainObjectName="UserPO"
               mapperName="UserMapper">
            <generatedKey column="id" sqlStatement="Mysql" identity="true"/>
        </table>

        <table tableName="user_info" domainObjectName="UserInfoPO"
               mapperName="UserInfoMapper">
            <generatedKey column="id" sqlStatement="Mysql" identity="true"/>
        </table>


    </context>
</generatorConfiguration>
```

```properties
文件名为mysql.properties
mysql.driverClass=com.mysql.cj.jdbc.Driver
mysql.username=huochai
mysql.password=Admin123@
mysql.url=jdbc:mysql://47.96.235.128:3306/student?useSSL=false&serverTimezone=UTC
# 生成对象的包名
mybatis.poPackage=com.huochai.bean.po
# 生成的映射文件包名
mybatis.mapperPackage=mapper
# 生成service的包名
mybatis.servicePackage=com.huochai.repository.mapper
# 生成对象位置
mybatis.javaProject=src/main/java
# 生成的映射文件位置
mybatis.resources=src/main/resources

# 方便xml文件引入依赖项
```

## 3、启动插件





如果想要快速使用的话，需要将mapper 文件扫描进来

使用

```java
import tk.mybatis.spring.annotation.MapperScan;
@MapperScan(basePackages = "com.huochai.domain.testDomain.mapper")
```

