# 零、基础

**每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的**

```
SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先配置的 Configuration 实例来构建出 SqlSessionFactory 实例。
```

从xml 中获取SqlSessionFactory

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  
  <!---environment 元素体中包含了事务管理和连接池的配置。--->
  <environments default="development">
    
    <environment id="development">
      <transactionManager type="JDBC"/>
      
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
      
    </environment>
  </environments>
  
  <!---映射器 这些映射器的 XML 映射文件包含了 SQL 代码和映射定义信息。-->
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
  
  
</configuration>
```

从Configuration 中获取

```java
// 数据源
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
// 事务
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
// 配置
Configuration configuration = new Configuration(environment);
// 添加映射器类
configuration.addMapper(BlogMapper.class);
// 创建
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```

```
映射器类是 Java 类，它们包含 SQL映射注解 从而避免依赖 XML 文件。不过，由于 Java 注解的一些限制以及某些 MyBatis 映射的复杂性，要使用大多数高级映射（比如：嵌套联合映射），仍然需要使用 XML 配置。有鉴于此，如果存在一个同名 XML 配置文件，MyBatis 会自动查找并加载它（在这个例子中，基于类路径和 BlogMapper.class 的类名，会加载 BlogMapper.xml）
```

 SqlSession 实例来直接执行已映射的 SQL 语句

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
}
```

**在配置文件中需要指明名称空间 **

```
命名解析：为了减少输入量，MyBatis 对所有具有名称的配置元素（包括语句，结果映射，缓存等）使用了如下的命名解析规则。

全限定名（比如 “com.mypackage.MyMapper.selectAllThings）将被直接用于查找及使用。
短名称（比如 “selectAllThings”）如果全局唯一也可以作为一个单独的引用。 如果不唯一，有两个或两个以上的相同名称（比如 “com.foo.selectAllThings” 和 “com.bar.selectAllThings”），那么使用时就会产生“短名称不唯一”的错误，这种情况下就必须使用全限定名。
```

生命周期

```
SqlSessionFactoryBuilder: 一旦创建了 SqlSessionFactory，就不再需要它了. SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）,

SqlSessionFactory: 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例.SqlSessionFactory 的最佳作用域是应用作用域.最简单的就是使用单例模式或者静态单例模式。

SqlSession:每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。 绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 如果你现在正在使用一种 Web 框架，每次收到 HTTP 请求，就可以打开一个 SqlSession，返回一个响应后，就关闭它。 这个关闭操作很重要，为了确保每次都能执行关闭操作，你应该把这个关闭操作放到 finally 块中。

映射器实例: 方法作用域才是映射器实例的最合适的作用域。 也就是说，映射器实例应该在调用它们的方法中被获取，使用完毕之后即可丢弃。 映射器实例并不需要被显式地关闭
```

**映射器是一些绑定映射语句的接口。映射器接口的实例是从 SqlSession 中获得的**



#  一、xml 配置









# 一、映射

```
1、在dao层使用@Mapper注解，或者使用配置扫描包 @MapperScan("com.huochai.dao"),需要加入到启动类上，并且要指定到dao层，范围过大时会引入其他包
2、mybatis:
 # 指定Xml文件的位置
  mapper-locations: classpath:mapper/*Mapper.xml
  configuration:
  # 指定将数据库中的字段映射到PO对象的规则， card-num:cardNum
    map-underscore-to-camel-case: true
```

```
org.apache.ibatis.binding.BindingException原因总结

1、mapper接口和mapper.xml是否在同一个包（package)下？名字是否一样（仅后缀不同）？ 可以通过配置文件来指定

2、mapper.xml的命名空间（namespace）是否跟mapper接口的包名一致？

3、接口的方法名，与xml中的一条sql标签的id一致

4、如果接口中的返回值List集合（不知道其他集合也是），那么xml里面的配置，尽量用resultMap（保证resultMap配置正确）,不
```

# 一、动态sql

# 二、批量操作

在xml文件中执行批量操作的时候使用 **foreach** 标签。

```
属性：

index：在list和数组中,index是元素的序号，在map中，index是元素的key，该参数可选
item：集合中元素迭代时的别名，该参数为必选
open：foreach代码的开始符号，一般是(和close=")"合用。常用在in(),values()时。该参数可选
separator：元素之间的分隔符，该参数可选。
close：foreach代码的关闭符号，一般是)和open="("合用。常用在in(),values()时。该参数可选。
collection：要做foreach的对象（必选）
	作为入参时，List对象默认用"list"代替作为键，
	数组对象有"array"代替作为键，
	传人map ，可遍历： map.keys;map.values;map.entrySet()
	Map对象没有默认的键。
	当然在作为入参时可以使用@Param("keyName")来设置键，设置keyName后，list,array将会失效。
	除了入参这种情况外，还有一种作为参数对象的某个字段的时候。如果User有属性List  ids。入参是User对	象，那么这个collection = "ids".  如果User有属性Ids ids;其中Ids是个对象，Ids有个属性List id;   	入参是User对象，那么collection = "ids.id"
```

 ```
 不管是多参数还是单参数的list,array类型，都可以封装为map进行传递。如果传递的是一个List，则mybatis会封装为一个list为key，list值为object的map，如果是array，则封装成一个array为key，array的值为object的map.
 ```

List 类型的参数，会默认封装成 key="list" value=List<> 的map

Array 类型的参数，会默认封装成 key="array" value=array[] 的map

```xml
自己封装成Map示例：
<select id="dynamicForeach3Test" resultType="Blog">
    select * from t_blog where title like "%"#{title}"%" and id in
          <foreach collection="ids" index="index" item="item" open="(" separator="," close=")">
               #{item}
          </foreach>
 </select>


 params.put("ids", ids);
 params.put("title", "中国");
 List blogs = blogMapper.dynamicForeach3Test(params);
```

## 2.1 批量插入

对 values() 进行遍历

values (),(),()

```xml
void batchSaveUser(List<SysUser> userList);
<insert id="batchSaveUser">
       insert into sys_user (ding_user_id, username, nickname, password, email, 
       mobile, avatar, creator_id, create_time, updator_id, update_time, is_delete)
       values
       <foreach collection="list" item="user" separator=",">
           (
           #{user.dingUserId}, #{user.username}, #{user.nickname}, #{user.password}, #{user.email},
           #{user.mobile}, #{user.avatar}, #{user.creatorId}, now(), #{user.updatorId}, now(), 0
           )
       </foreach>
   </insert>
```

```xml
void batchSaveGroupAndUser(@Param("map") Map<Long, List<Long>> groupUserMap);

<insert id="batchSaveGroupAndUser" parameterType="java.util.Map">
        insert into sys_group_member (group_id, user_id, creator_id, create_time)
        values
        <foreach collection="map.keys" item="groupId" separator=",">
            <foreach collection="map[groupId]" item="userId" separator=",">
                (
                #{groupId}, #{userId}, 'admin', now()
                )
            </foreach>
        </foreach>
    </insert>
```

```xml
void batchInsert(@Param("map") Map<String, String> map);


<insert id="batchInsert" parameterType="java.util.Map">
        insert into brand_info (code, `name`, is_delete, create_time)
        values
        <foreach collection="map.entrySet()" index="key" item="value" open="(" close=")" separator=",">
            #{key}, #{value}, 0, now()
        </foreach>
    </insert>
```

## 2.2 批量更新

**在执行批量Update的时候，数据库的url配置需要添加一项参数：&allowMultiQueries=true**

```xml
<update id="batchUpdateCorporation" parameterType="java.util.List">
        <foreach collection="list" item="item" index="index" separator=";">
            update sys_corporation set
            <if test="item.name != null and item.name !=''">
                `name` = #{item.name},
            </if>
            <if test="item.code != null and item.code !=''">
                code = #{item.code},
            </if>
            <if test="item.parentCode != null and item.parentCode !=''">
                parent_code = #{item.parentCode},
            </if>
            updater = 'system',
            update_time = now()
            where id = #{item.id}
        </foreach>
    </update>
```

插入对象并且将新插入的对象id 回显出来

```xml

<insert id="insertStudent" useGeneratedKeys="true" keyProperty="id">
    insert into sys_student(`name`, age, sex)
    values (#{name}, #{age}, #{sex})
 </insert>
```

**keyProperty：表示指定的属性作为主键**

**useGeneratedKeys：如果为true，会使mybatis使用Jdbc的getGeneratedKeys()的方法来获取数据库内部生成得到主键。**

此时不需要更改对象的默认命名



执行流程

```
1、先获取SqlSessionFactory 里面封装了Configuration ，Configuration 里面有 Mapper对象
2、打开连接
3、执行sql:
```



# Mybits快速生成代码

## 1、引入jar包

```xml
<properties>
        指定数据库版本
        <mysql.version>8.0.22</mysql.version>
   		 <tkmybatis.version>2.1.5</tkmybatis.version>
</properties>

 <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>


        <!--        mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

 <!--tk mybatis-->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
            <version>${tkmybatis.version}</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.6</version>
        </dependency>


 <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.7</version>
                <configuration>
                    配置文件的位置
                    <configurationFile>${basedir}/src/main/resources/generator/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        要制定版本（与引入的版本相同）
                        <version>${mysql.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>tk.mybatis</groupId>
                        <artifactId>mapper-generator</artifactId>
                        <version>1.0.5</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
```

## 2、配置文件

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
            <property name="mappers" value="com.huochai.dao.BaseMapper"/>
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

        <!-- tinyInt转换为Integer    -->
<!--        <javaTypeResolver type="com.huochai.dao.MyJavaTypeResovler"/>-->


        <!-- 生成模型的包名和位置 -->
        <javaModelGenerator targetPackage="${mybatis.modelPackage}" targetProject="${mybatis.javaProject}"/>
        <!-- 生成的映射文件包名和位置 -->
        <sqlMapGenerator targetPackage="${mybatis.mapperPackage}" targetProject="${mybatis.resources}"/>
        <!-- 生成service的包名和位置 -->
        <javaClientGenerator targetPackage="${mybatis.clientPackage}" targetProject="${mybatis.javaProject}"
                             type="XMLMAPPER"/>

        <!--数据库表-->
        <table tableName="wind_condition" domainObjectName="WindConditionPO1" mapperName="WindConditionMapper1">
            <generatedKey column="id" sqlStatement="Mysql" identity="true"/>
        </table>

        <table tableName="wind_power_equipment_active" domainObjectName="WindPowerEquipmentActivePO1"
               mapperName="WindPowerEquipmentActiveMapper1">
            <generatedKey column="id" sqlStatement="Mysql" identity="true"/>
        </table>
    </context>
</generatorConfiguration>
```

```properties
mysql.driverClass=com.mysql.cj.jdbc.Driver
mysql.username=huochai
mysql.password=Admin123@
mysql.url=jdbc:mysql://47.96.235.128:3306/machine_monitor?useSSL=false&serverTimezone=UTC
mybatis.modelPackage=com.huochai.bean.po
mybatis.mapperPackage=sqlmap.kpi
mybatis.clientPackage=com.huochai.mapper.machine
mybatis.javaProject=src/main/java
mybatis.resources=src/main/resources

# 方便xml文件引入依赖项
```

## 3、启动插件

<img src="/Users/mac/Library/Application Support/typora-user-images/image-20210529182129932.png" alt="image-20210529182129932" style="zoom:50%;" />

## 4、引用

```
生成的Mapper类中有各种SQL方法，也可以自己写[正常的定义接口，之后再XML文件中书写实现]。


想要依赖入驻Mapper对象，需要加入包扫描。

import tk.mybatis.spring.annotation.MapperScan;
@MapperScan(basePackages = {"com.huochai.mapper.machine"})
```



