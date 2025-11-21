# 一、配置xml

```xml
<spring-boot.version>2.6.13</spring-boot.version>
<mybatis-plus.version>3.5.10.1</mybatis-plus.version>
<dependency>
  <groupId>com.mysql</groupId>
  <artifactId>mysql-connector-j</artifactId>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>com.taobao.arthas</groupId>
  <artifactId>arthas-spring-boot-starter</artifactId>
  <version>3.6.7</version>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <optional>true</optional>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-boot-starter</artifactId>
  <version>${mybatis-plus.version}</version>
</dependency>
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-generator</artifactId>
  <version>${mybatis-plus.version}</version>
</dependency>

<!-- MySQL JDBC Driver -->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
  <groupId>org.freemarker</groupId>
  <artifactId>freemarker</artifactId>
  <version>2.3.28</version>
  <scope>compile</scope>
</dependency>
```

# 二、配置application.properties

```properties
# 应用服务 WEB 访问端口
server.port=8080
# application.properties
spring.datasource.url=jdbc:mysql://123.57.235.173:3307/test_db?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&remarks=true&useInformationSchema=true
spring.datasource.username=huochai
spring.datasource.password=asdf1234
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# MyBatis-Plus Configuration
mybatis-plus.mapper-locations=classpath:mapper/*.xml
mybatis-plus.type-aliases-package=com.example.model
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

# 三、Configuration

```
@Configuration
@MapperScan("com.huochai.mybatisplus.mapper")
public class DataBaseConfig {
}
```

# 四、自动生成类

```java
public class MysqlGenerator {
    
    public static void main(String[] args) {

        FastAutoGenerator.create("jdbc:mysql://123.57.235.173:3307/test_db?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&remarks=true&useInformationSchema=true",
                        "huochai",
                        "asdf1234")
                .globalConfig(builder -> builder
                        .author("huochai")
                        .outputDir(Paths.get(System.getProperty("user.dir")) + "/src/main/java") // 输出目录
                        .commentDate("yyyy-MM-dd")
                )
                .dataSourceConfig(builder ->
                        builder.typeConvertHandler((globalConfig, typeRegistry, metaInfo) -> {
                            // 兼容旧版本转换成Integer
                            if (JdbcType.TINYINT == metaInfo.getJdbcType()) {
                                return DbColumnType.INTEGER;
                            }
                            return typeRegistry.getColumnType(metaInfo);
                        }))
                .packageConfig(builder -> builder
                        .parent("com.huochai.mybatisplus") // 父包名
                        .entity("po")  // 设置 Entity 包名
                        .mapper("mapper") // 设置 Mapper 包名
                        .service("service") // 设置 Service 包名
                        .serviceImpl("service.impl") //设置 Service Impl 包名
                        .pathInfo(Collections.singletonMap(OutputFile.xml, Paths.get(System.getProperty("user.dir")) + "/src/main/resources/mapper")) // 设置路径配置信息
                        .controller("controller") // 设置 Controller 包名
                )
                .strategyConfig(builder -> builder
                        .addInclude("kpi_calibration_assessee_relation") // 设置需要生成的表名
                        .entityBuilder()
                        .enableLombok() // 启用 Lombok
                        .enableTableFieldAnnotation() // 启用字段注解
                        .formatFileName("%sPO")
                        .controllerBuilder()
                        .enableRestStyle() // 启用 REST 风格
                        .mapperBuilder()
                        .enableBaseResultMap()
                        .enableFileOverride()

                )
                .templateEngine(new FreemarkerTemplateEngine())
                .execute();
    }
}
```



# 五、查询接口

```java
@Repository
public class KpiCalibrationAssesseeRelationManager {
    @Resource
    private KpiCalibrationAssesseeRelationMapper assesseeRelationMapper;
    public boolean insert(KpiCalibrationAssesseeRelationPO po) {
        return assesseeRelationMapper.insert(po) > 0;
    }
    public KpiCalibrationAssesseeRelationPO selectById(Long id) {
        QueryWrapper<KpiCalibrationAssesseeRelationPO> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("id", id);
        return assesseeRelationMapper.selectOne(queryWrapper);
    }
}
```



踩坑：

提示java.lang.NoClassDefFoundError: freemarker/template/Configuration

需要引入

```xml
 <dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.28</version>
            <scope>compile</scope>
        </dependency>
```

