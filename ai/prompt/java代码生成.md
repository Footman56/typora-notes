# 生成通用的PO、DO、Service、ServiceImpl、Manager、Mapper

```
ALTER TABLE `kpi_plan_flow_setting_info`
    ADD COLUMN `allow_deliver` tinyint(1) NULL COMMENT '是否允许转交 1-允许 0-不允许' AFTER `start_generate_electronic_sign`
;

# 转交说明
ALTER TABLE `kpi_todo_emp_status_explanation`
    ADD COLUMN `round_group_id` char(32) NULL COMMENT '分组id' AFTER `type`,
    ADD COLUMN `process_id`     char(32) NULL COMMENT '流程id' AFTER `round_group_id`
;


ALTER TABLE `kpi_plan_process_performance_calibration_setting`
    ADD COLUMN `allow_deliver` tinyint(4) NULL COMMENT '是否允许转交' AFTER `reject_control`
;


ALTER TABLE `kpi_todo_emp_status_explanation`
    ADD KEY `idx_com_plan` (`company_id`, `plan_id`) USING BTREE
;


# 环节抄送配置表
# 需要plan_flow的索引
CREATE TABLE `kpi_plan_flow_carbon_copy_setting`
(
    `id`                        bigint(20)   NOT NULL AUTO_INCREMENT,
    `head_id`                   char(32)     NOT NULL COMMENT '总公司HeadID',
    `company_id`                char(32)     NOT NULL COMMENT '公司id',
    `plan_id`                   char(32)     NOT NULL COMMENT '方案id',
    `flow_id`                   char(32)     NOT NULL COMMENT '环节id',
    `inspection_status`         tinyint(1)   NOT NULL COMMENT '环节类型',
    `carbon_copy_time`          tinyint(1)   NULL COMMENT '抄送时机',
    `review_person_type`        tinyint(4)   NULL COMMENT '考评人类型',
    `review_person_type_id`     varchar(32)  NULL COMMENT '考评人类型对应id，eg:自选分组id或者自建角色id，根据review_person_type区分',
    `review_person_object_name` varchar(255) NULL COMMENT '考评人名称',
    `employee_id`               char(32)     NULL COMMENT '指定人',
    `employee_name`             char(255)    NULL COMMENT '指定人姓名',
    `status`                    tinyint(4)   NULL COMMENT '指定人状态',
    `is_del`                    tinyint(1)   NOT NULL COMMENT '是否删除',
    `addtime`                   int(11)      NOT NULL COMMENT '创建时间',
    `modtime`                   int(11)      NOT NULL COMMENT '修改时间',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARACTER SET = utf8mb4
  COLLATE = utf8mb4_unicode_ci
    COMMENT ='环节抄送配置表';

ALTER TABLE `kpi_plan_flow_carbon_copy_setting`
    ADD KEY `idx_plan_flow` (`company_id`, `plan_id`, `flow_id`) USING BTREE
;


# 被考核人抄送待办表
CREATE TABLE `kpi_assess_carbon_copy_message_todo`
(
    `id`                 bigint(20)   NOT NULL AUTO_INCREMENT COMMENT '主键id',
    `head_id`            char(32)     NOT NULL COMMENT '总公司HeadID',
    `company_id`         char(32)     NOT NULL COMMENT '公司id',
    `plan_id`            char(32)     NOT NULL COMMENT '方案id',
    `flow_id`            char(32)     NULL COMMENT '环节id',
    `flow_name`          varchar(200) NULL COMMENT '环节名称',
    `inspection_status`  tinyint(1)   NOT NULL COMMENT '环节类型',
    `carbon_copy_emp_id` char(32)     NULL DEFAULT '' COMMENT '抄送人员工id',
    `assessee_emp_id`    char(32)     NOT NULL COMMENT '被考核人员工id',
    `is_read`            tinyint(1)   NOT NULL COMMENT '是否查阅',
    `is_del`             tinyint(1)   NOT NULL COMMENT '是否删除',
    `addtime`            int(11)      NOT NULL COMMENT '创建时间',
    `modtime`            int(11)      NOT NULL COMMENT '修改时间',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARACTER SET = utf8mb4
  COLLATE = utf8mb4_unicode_ci
    COMMENT ='被考核人抄送待办表';



ALTER TABLE `kpi_assess_carbon_copy_message_todo`
    ADD KEY `idx_com_plan_flow` (`company_id`, `plan_id`, `flow_id`) USING BTREE,
    ADD KEY `idx_com_plan_copy_assess` (`company_id`, `plan_id`, `carbon_copy_emp_id`)
;
```

```
根据sql文件中的创建表相关的语句生成对应的PO、DO、Service、Manger、Mapper层数据。要求生成的代码必须满足规则模版中设置的规则，必须满足现有代码的包结构，生成的代码功能必须是完整的，不需要进行省略。要求Manger 实现通过mybatis的Example来实现
具体生成在com.qjyd.appraisal.domain.kpiPlan下。
```

```
根据修改语句中 mysql表名 在现有代码结构中对应的PO、DO 、XML文件，在现在对象中创建新的属性，并且完成PO与DO之间的属性负责。要求修改对象必须在现有代码中
```



# 生成通用的rpc实现

```
请参考KpiEmployeeDataServiceImpl中的pageMyTeamAppraisal中的接口实现和上层调用路径，生成调用IKpiEmployeeDataRpcService中的getMyCarbonCopyEmpPeriodList接口的http方法
```

<img src="https://raw.githubusercontent.com/Footman56/images/master/img202502241429675.png" style="zoom:50%;" />

上下文：需要生成的controller 、service、serviceImpl 、 参考方法的位置

```
#  多行数据转换成符合sql查询的条件
将 {{string}} 以换行符为分隔分成多组，每一组用引号包围，用逗号将每一组拼接起来


# 拼接sql--in
{{string}} 将这些数据填充到如下sql 的in 条件中
{{sql}}


# 
读取excel 中所有的数据,将内容拼成sql 中的in条件
SELECT * FROM `kpi_assessee_info` WHERE `plan_id` in () and `employee_id` in () and `is_del` = 0;
读取excel 中的plan_id 列下所有的数据作为 plan_id`的 in 条件
读取excel 中的assess_emp_id 列下所有的数据作为 employee_id的 in 条件。
请写出完整的sql 


#
请用java 的正则表达式 去匹配下面的逻辑:
 规则一:
 “完成值 <= 目标值 < 挑战值” 这种形式的可以匹配到,其中完成值、目标值、挑战值 代表的含义是变量,包括中文、英文、字母、数字、下划线，但是下划线不能开头
 <= ,<  这种代表符号，可能符号有 <= ,<,>,>= 这四种
规则二：
“完成值 <= 目标值 && 目标值 < 挑战值” 这种格式的不能匹配到, 中间连接符号可以是: && 、||
规则三:
"if(完成值<= 目标值){} else if(目标值< 挑战值) {}" 这样的格式也不能匹配
要匹配的话需要完全匹配规则。
规则四：
“IF(完成值 <= 目标值 ,20,30)” 这样的格式也不能匹配
只有文本中匹配到规则一、规则二、规则三的内容就算符合条件
```



# 逻辑删除sql

```
请使用mybatis中的Example来实现，根据接口的参数来逻辑删除数据，可以参考KpiPlanFlowTriggerManagerImpl中的delByPlan()
```

<img src="https://raw.githubusercontent.com/Footman56/images/master/img202502191732811.png" alt="image-20250219173247621" style="zoom:50%;" />

上下文分成三部分： PO、参考实例KpiPlanFlowTriggerManagerImpl、具体代码要生成的位置



# 查询语句

```
请补全KpiPlanFlowCarbonCopySettingServiceImpl中的listByPlanFlow方法实现，具体方法实现需要参考KpiPlanFlowCarbonCopySettingServiceImpl的getByPlan
```

<img src="https://raw.githubusercontent.com/Footman56/images/master/img202502191933743.png" alt="image-20250219193348561" style="zoom:50%;" />

上下文分成三部分： 接口、Manager、对应的ManagerImpl



修改接口

![image-20250317112838368](https://raw.githubusercontent.com/Footman56/images/master/img202503171128037.png)
