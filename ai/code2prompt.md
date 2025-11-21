# 安装

需要git、rust 、cargo

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```



```
#  Rust 和 Cargo 已正确安装：
rustc --version
cargo --version
```



```
code2prompt /Users/peilizhi/javaProjects/xrxs-appraisal-springboot/appraisal-webfront --tokens --encoding cl100k_base --output ai_input.md
```







```
老数据-转交-plz-进行中:20fc4d245c744ed2a6ff186e7171b240
老数据-转交-plz-归档:818b4b196a40436586b3cffe87b0fdbe
老数据-转交-plz-未开启:95070dc54163435d84f871c0b324313a



UPDATE `kpi_plan_flow_setting_info`  as flow inner join `kpi_plan_flow_setting_customize`  as cus on flow.`plan_id`  = cus.`plan_id`  
  and flow.`flow_id` = cus.`flow_id`  and flow.`inspection_status` = 9 
SET
  flow.allow_deliver = cus.`allow_deliver` 
WHERE
  flow.`is_del` =0 and cus.`is_del` =0 and flow.`plan_id` = '20fc4d245c744ed2a6ff186e7171b240'
;


curl -X GET "http://localhost:9101/test/flow/wash-single-allowDeliverEval-control?companyId=839fe5f3310d48b5a6dcb65cbf925f62&planId=20fc4d245c744ed2a6ff186e7171b240&flowId=ff18e9ceae884f03ac67db9499fc9389"



UPDATE `kpi_plan_flow_setting_info`  as flow inner join `kpi_plan_flow_setting_customize`  as cus on flow.`plan_id`  = cus.`plan_id`  
  and flow.`flow_id` = cus.`flow_id`  and flow.`inspection_status` = 9 
SET
  flow.allow_deliver = cus.`allow_deliver` 
WHERE
  flow.`is_del` =0 and cus.`is_del` =0 and flow..`company_id` ='839fe5f3310d48b5a6dcb65cbf925f62'
;



curl -X GET "http://localhost:9101/test/flow/wash-company-allowDeliverEval-control?companyId=839fe5f3310d48b5a6dcb65cbf925f62"



curl -X GET "http://localhost:9101/test/flow/wash-all-allowDeliverEval-control"
```

