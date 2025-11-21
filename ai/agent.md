1. 能够本地调用 类似raycast
2. **使用免费的模型**
   + 探索ai 免费的api接口
   + ~~搭建本地模型~~（本地机器内存有限）
3. 提供一些基础知识（比如数据库表，代码）能够根据自然语言描述去生成解决方案
4. 





# sql 快速生成

```
你是mysql专家，根据条件书写sql, 兼顾性能
形式：mybatis的xml映射文件
表：kpi_assessee_info
类型：查询
入参：List<KpiPlanFlowParamDO> ,多个KpiPlanFlowParamDO 之间是或的关系
入参： bool isDel 
条件一： KpiPlanFlowParamDO 中有两个属性，flowName匹配kpi_assessee_info#flow_name ,inspectionStatus 匹配kpi_assessee_info#inspection_status。
条件二：isDel 匹配kpi_assessee_info#is_del
多个条件之间是and 关系
```



# java 问题解决

```
你是java 工程师，请根据问题给出通用的解决方法，需注意拓展性,并且兼顾性能，并举例代码说明 

现有数据List<LinkedMap> 请转换成List<KpiPlanFlowParamDO> 

KpiPlanFlowParamDO {


    private String flowId;


    private String flowName;


    private Integer inspectionStatus;
}
```



# sql 快速填充

```
todo "bf710cc9929343919de03269ec1ad1ce"  "b1474a60134443ca9395a6159b6e0e2b"

=> SELECT * FROM `kpi_process_todo` WHERE `plan_id` = 'bf710cc9929343919de03269ec1ad1ce' and `assessee_emp_id` = 'b1474a60134443ca9395a6159b6e0e2b'

```



# 智能客服

+ 技术客服 -> 生产
+ 产品逻辑客服

# 总结工具

+ 文档维护-构建模版 每完成一个事都用文档维护，总结的时候直接上传文档
