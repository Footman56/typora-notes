一、老版常用sql

```sql
# 根据方案名称获取方案设置
SELECT * FROM appraisal_setting WHERE name  LIKE '%xxxx%';

# 查询方案信息根据名称
SELECT * FROM appraisal WHERE setting_id in (SELECT id FROM appraisal_setting WHERE name like '%xxx%');

# 根据设置ID获取方案信息 参数是appraisal_setting的id 字段
SELECT * FROM appraisal WHERE setting_id ='0'

# 获取被考核人明细
SELECT * FROM appraisal_employee WHERE appraisal_id ='xx' and employee_id ='xx'；

# 查看方案流程设置
SELECT * FROM appraisal_setting_node WHERE setting_id ='0'；

# 查看员工待办
setting_node_id:appraisal_setting_node表中的id
record_id: appraisal_employee表中的id 
SELECT * FROM appraisal_employee_node WHERE setting_node_id ='xx' and record_id ='xx'
```

二、查询没有走到结果确认环节的人，（汇报对象）

```sql
SELECT employee_id  FROM employee_job_new WHERE hrbp_id ='XX';

之后导出employee_id
SELECT * FROM kpi_assessee_info WHERE plan_id ='AA' and employee_id  in (XX,XX) and inspection_status != 4;
```





```
a1 = getDimenScore(1,"业绩维度")
a2 = getDimenScore(2,"价值观维度")
a3 =IF(a1<0.6,"没有达成",IF(a1<0.8,"部分达成",IF(a1<1.1,"达成目标",IF(a1<1.4,"超过","远远超过"))))

a4 =IF(a2<2,"几乎不",IF(a2<3,"很少",IF(a2<4,"一般",IF(a2<5,"经常","始终如一"))))

if (((a4 == "始终如一") || (a4 == "经常" ) || (a4 == "一般")) && (a3 == "没有达成") || ((a4 == "经常") || (a4 == "一般" ) || (a4 == "很少")) && (a3 == "部分达成")  || ((a4 == "很少") || (a4 == "几乎不" ) ) && (a3 == "达成目标")   || ((a4 == "很少") || (a4 == "几乎不")) && (a3 == "超过") || ((a4 == "几乎不")) && (a3 == "远远超过")) {
考核总分 = 4
} if (((a4 == "始终如一") || (a4 == "经常" )) && (a3 == "达成目标") || ((a4 == "经常")) && (a3 == "超过")){
考核总分 = 2
} if (((a4 == "始终如一") || (a4 == "经常" )) && (a3 == "远远超过") || ((a4 == "始终如一")) && (a3 == "超过")){
考核总分 = 1
} if (((a4 == "很少") || (a4 == "几乎不" )) && (a3 == "没有达成") || ((a4 == "几乎不")) && (a3 == "部分达成")){
考核总分 = 5
} else{
考核总分 = 3
}
```



```
a1 =  1.035  
a2 =  4
a3 = 达成目标
a4 = 经常
```





```
(((a4 == "始终如一") || (a4 == "经常" )) && (a3 == "达成目标") || ((a4 == "经常")) && (a3 == "超过"))
```



```
curl 'http://localhost:9650/temp/appraisal360/autoStopAppraisal?appraisalId=5de83f303cea4f55ae95e827ca299cdc&companyId=827ca94aec3347df87836ef9f24bc4ed'
```





```
The last packet successfully received from the server was 30,224 milliseconds ago.  The last packet sent successfully to the server was 30,226 milliseconds ago.
```

是sql 执行时间过长，可以分批次去执行
