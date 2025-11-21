# 零 文档

**https://mp.baomidou.com/guide/#%E7%89%B9%E6%80%A7**

# 一、引入jar

```xml
<mybatis-plus.version>3.5.3.1</mybatis-plus.version>


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
```



# 二、配置

```yaml
mybatis-plus:
  global-config:
    db-config:
      ## 逻辑删除已删除值
      logic-delete-value: 1
      ## 逻辑删除未删除值
      logic-not-delete-value: 0
```



# 三、数据源配置

```java
import org.mybatis.spring.annotation.MapperScan;
import com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean;
/**
 * 动态数据源
 *@author peilizhi
 *@date 2023/11/8 17:56
 **/
@Configuration
@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})
// 扫描的是mapper接口
@ConditionalOnProperty(name = "custom.database", havingValue  = "dynamic")
@MapperScan(basePackages = "com.huochai.domain.**.mapper", sqlSessionFactoryRef = "dynamicSqlSessionFactory")
public class DynamicDataSourceConfig {

    @Resource
    private MasterDataSourceConfig masterDataSourceConfig;
    @Resource
    private SlaveDataSourceConfig slaveDataSourceConfig;

    @Primary
    @Bean("dynamicDataSource")
    public DataSource dynamicDataSource() throws SQLException {
        DynamicDataSource dataSource = new DynamicDataSource();
        // 备选数据源,后续根据参数从备选数据源中获取
        Map<Object, Object> targetDataSources = new HashMap<>(2);
        targetDataSources.put(BaseConstant.MASTER_DATABASE, masterDataSourceConfig.masterDataBase());
        targetDataSources.put(BaseConstant.SLAVE_DATABASE, slaveDataSourceConfig.slaveDataSource());
        dataSource.setTargetDataSources(targetDataSources);
        // 默认数据源
        dataSource.setDefaultTargetDataSource(masterDataSourceConfig.masterDataBase());
        return dataSource;
    }

    @Primary
    @Bean("dynamicSqlSessionFactory")
    public SqlSessionFactory dynamicSqlSessionFactory(@Qualifier("dynamicDataSource") DataSource dataSource) throws Exception {
        // 设置数据源
        MybatisSqlSessionFactoryBean masterSqlFactory = new MybatisSqlSessionFactoryBean();
        masterSqlFactory.setDataSource(dataSource);
        // mapper的xml文件位置
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
      // 扫描的是xml文件
        String locationPattern = "classpath*:/mapper/**/*.xml";
        masterSqlFactory.setMapperLocations(resolver.getResources(locationPattern));
        return masterSqlFactory.getObject();
    }

    /**
     * 配置事务管理器
     */
    @Primary
    @Bean("dynamicPlatformTransaction")
    public DataSourceTransactionManager dynamicPlatformTransactionManager(@Qualifier("dynamicDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}

```



```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
public interface StudentMapper extends BaseMapper<StudentPO> {


    @Select("select * from database.student where is_del=0")
    List<StudentPO> selectAll();
}
```







# 自动生成代码

适用于版本在3.5.1 以上 

```xml
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


        <!-- 模板引擎 -->
        <dependency>
            <groupId>org.apache.velocity</groupId>
            <artifactId>velocity-engine-core</artifactId>
            <version>2.0</version>
        </dependency>
```

```java
public class CodeGeneratorUtil {


    private static final String PACKAGE_NAME = "com.huochai.domain.security";
    private static final String OUT_PATH = System.getProperty("user.dir") + "/test-consumer";
    private static final String URL = "jdbc:mysql://localhost:3306/test?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&characterEncoding=utf-8&useSSL=false&allowPublicKeyRetrieval=true&tinyInt1isBit=false";
    private static final String USERNAME = "huochai";
    private static final String PASSWORD = "asdf1234";
    private static final String MODULE_NAME = "security";
    public static final String TABLE_NAME = "sys_user,sys_role,sys_user_role,sys_menu,sys_role_menu";

    public static void main(String[] args) {

      
        String projectPath = OUT_PATH;
        // 代码生成器
        FastAutoGenerator.create(URL, USERNAME, PASSWORD)
                .globalConfig(gc ->
                        gc.outputDir(projectPath + "/src/main/java")
                                .author("huochai")
                                .disableOpenDir()
                )
                .dataSourceConfig(dsc ->
                        dsc.typeConvertHandler(new ITypeConvertHandler() {
                            /**
                             * 转换字段类型
                             *
                             * @param globalConfig 全局配置
                             * @param typeRegistry 类型注册信息
                             * @param metaInfo     字段元数据信息
                             * @return 子类类型
                             */
                            @Override
                            public @NotNull
                            IColumnType convert(GlobalConfig globalConfig, TypeRegistry typeRegistry, TableField.MetaInfo metaInfo) {
                                // 兼容旧版本转换成Integer
                                int typeCode = metaInfo.getJdbcType().TYPE_CODE;
                                if (typeCode == Types.SMALLINT) {
                                    return DbColumnType.INTEGER;
                                } else if (typeCode == Types.TINYINT) {
                                    return DbColumnType.INTEGER;
                                }
                                return typeRegistry.getColumnType(metaInfo);
                            }
                        }))
                .packageConfig(pc -> {
                    String xmlPath = projectPath + "/src/main/resources/mapper/" + MODULE_NAME;
                    pc.parent(PACKAGE_NAME)
                            .entity("bean.po")
                            .pathInfo(Collections.singletonMap(OutputFile.xml, xmlPath)); // xml位置（还可自定义配置entity，service等位置）
                })
                .templateConfig(tc -> {
                    // 不配置controller
                    tc.controller("");
                })
                .strategyConfig(sc ->
                        sc.
                                addTablePrefix(MODULE_NAME + "_")
                                .addInclude(TABLE_NAME.split(","))


                                .entityBuilder()
                                .enableTableFieldAnnotation() //开启生成实体时生成的字段注解
                                .naming(NamingStrategy.underline_to_camel)
                                .columnNaming(NamingStrategy.underline_to_camel)
                                .enableLombok()
                                .formatFileName("%sPO")
                                .enableFileOverride()

                                .serviceBuilder()
                                .formatServiceFileName("%sService") // Service 文件名称
                                .formatServiceImplFileName("%sServiceImpl") // ServiceImpl 文件名称
                                //.superServiceClass("")
                                .enableFileOverride()

                                .mapperBuilder()
                                .enableBaseColumnList()
                                .enableBaseResultMap()
                                .formatMapperFileName("%sMapper")
                                .formatXmlFileName("%sMapper")
                                .enableFileOverride()// 文件覆盖
                )

                .execute();

    }

}
```

解决 tinting 转换成int形式

```
1. url 上加上 tinyInt1isBit=false
2. 重写typeConvertHandler 方法
```





```
spring.database.master.url=jdbc:mysql://localhost:3306/test?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&characterEncoding=utf-8&useSSL=false&allowPublicKeyRetrieval=true
spring.database.master.username=huochai
spring.database.master.password=asdf1234
spring.database.master.driver-class-name=com.mysql.cj.jdbc.Driver
```

