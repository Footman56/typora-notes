# 一、明确版本

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202303161022243.png" alt="image-20230316102158439" style="zoom:50%;" />

需要明确springboot 与es 的版本对应关系

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>jackson-databind</artifactId>
                    <groupId>com.fasterxml.jackson.core</groupId>
                </exclusion>
            </exclusions>
        </dependency>
```

# 二、新增配置

```yaml
spring:
	elasticsearch:
		uris: localhost:9200
    connection-timeout: 10s
    password:
    username:
```



```java
/**
 * @author peilizhi
 * @date 2023/3/29 15:43
 **/
@Configuration
public class EsConfig extends AbstractElasticsearchConfiguration {


    /**
     * 链接
     */
    @Value("${spring.data.elasticsearch.client.reactive.endpoints}")
    private String endPoint;


    /**
     *  用户名
     */
    @Value("${spring.data.elasticsearch.client.reactive.username}")
    private String username;

    /**
     * 密码
     */
    @Value("${spring.data.elasticsearch.client.reactive.password}")
    private String password;


    @Bean("elasticsearchTemplate")
    @Primary
    public ElasticsearchRestTemplate elasticsearchTemplate() {
        return new ElasticsearchRestTemplate(elasticsearchClient());
    }


    @Bean("restHighLevelClient")
    @Primary
    @Override
    public RestHighLevelClient elasticsearchClient() {
        HttpHeaders defaultHeaders = new HttpHeaders();

        ClientConfiguration.TerminalClientConfigurationBuilder builder = ClientConfiguration.builder()
                .connectedTo(endPoint)
                .withDefaultHeaders(defaultHeaders);

        if (StringUtil.isNotBlank(username)) {
            defaultHeaders.setBasicAuth(username, password);
            builder.withDefaultHeaders(defaultHeaders);
            builder.withBasicAuth(username, password);
        }
        ClientConfiguration clientConfiguration = builder.build();

        return RestClients.create(clientConfiguration).rest();
    }

    @Bean
    @Override
    public EntityMapper entityMapper() {
        ElasticsearchEntityMapper entityMapper = new ElasticsearchEntityMapper(elasticsearchMappingContext(),
                new DefaultConversionService());
        entityMapper.setConversions(elasticsearchCustomConversions());
        return entityMapper;
    }
}
```

# 三、创建索引对象

```
```









```
1. 部门查询（一级部门、部门类型） 之后再聚合的时候，返回的数据可能比查询条件多
2.  搜索的时候 不带下级部门数据。所在部门
3.  指标维度 后续可能会计算同部门最大值 ，设计mysql 字段存储、设计es 存储
告诉他怎么取字段，取哪些字段，怎么算，什么时候取，取完之后计算结果放在哪里（数仓放或者我们取）
4. 区别指标唯一（目前根据指标名称），并且每个方案上的指标冗余同部门平均值
5. 指标索引如何设置 自定义字段（直接从es中查询出来） 
6. 
```







- 数据清洗（12天）
  - mysql 清洗维度分和类型 - 并且同步es （2天）
  - mysql 清洗被考核人基本信息并迁移新表（1天）  （待定，迁移新表涉及点多）
  - mysql  被考核人最终分、指标分信息，同步es-供智数使用（1.5 天）
  - mysql 获取同部门平均最终分、平均指标分，同步es -智数计算结果 （1.5天）
  - es 被考核人信息 索引 创建、新增文档、更新文档 （1天）
  - es 被考核人指标信息 索引创建、新增文档、删除文档、更新文档 （3天）
  - 监听员工MQ 维护被考核人基本信息 （1天）
  - es 和 mysql 双数据源数据完整性、一致性校验定时任务(1天)
- 考核结果分布报表（7.5天）
  - 筛选方案保存、删除（0.25天）
  - 筛选条件下拉框 - 考核方案（0.25天）
  - 初始化筛选条件列表（0.5天）
  - 表头自定义控制（0.5天）
  - 获取报表数据 （4.5天）
    - 按部门分组-摸索代码如果实现聚合（1.5天）
      - 默认直属部门聚合
      - 一级部门聚合
    - 按岗位分组（1天）
    - 按职级分组（1天） 未排职级分组
    - 按司龄分组- 入职时间转换成司龄（1天）
  - 组装成页面（0.5天）
  - 全量导出- excel 中单元格合并（1天）
- 员工绩效对比（5.5天）
  - 筛选方案保存、删除（0.25天）
  - 筛选条件下拉框 - 考核方案（0.25天）
  - 获取报表数据（4.5天）
    - 按考核结果分组（1.5天）
      - 聚合分页（0.5天）
      - 获取智数同部门平均分（0.25天）
      - 组装成页面（0.5 天）
      - 字段管理（0.25）
    - 按考核维度分组（1.5天）
      - 聚合分页（0.5天）
      - 获取智数同部门平均分（0.25天）
      - 组装成页面（0.5 天）
      - 字段管理（0.25天）
    - 按指标维度分组（1.5天）
      - 聚合分页（0.5天）
      - 获取智数同部门平均分（0.25天）
      - 组装成页面（0.5 天）
      - 字段管理（0.25天）
  - 多个页面报表全量导出- excel 中单元格合并（1 天）
- 指标统计分析（4天）
  - 筛选方案保存、删除（0.25天）
  - 筛选条件下拉框 - 考核维度 + 考核指标下拉框（0.5天）
  - 字段管理（1天）
  - 获取报表数据 （2天）
    - 聚合分页（0.5天）
    - 记录字段计算方式（0.25天）
    - 组成页面（0.5天）
  - 导出 （0.5天）
- 指标达成明细（1.5天）
  - 筛选方案保存、删除（0.25天）
  - 筛选条件下拉框 - 考核方案（0.25天）
  - 字段管理（0.25天）
  - 数据查询（0.5天）
  - 导出（0.5天）
- 历史绩效统计（3天）
  - 筛选方案保存、删除（0.25天）
  - 筛选条件下拉框 - 考核方案（0.25天）
  - 字段管理特殊处理（0.5天）
  - 查询页面数据（1天）
  - 导出（1天）
  - 总分计算（0.5天）

