# 一、查询指标信息

1. sql 并导出excel

   ```sql
   SELECT `target_name` ,`target_id`,`target_items`   FROM `kpi_assessee_target_detail` 
   WHERE `plan_id` ='417c4544af484b7c9b196a37edb2d78d' and `assessee_emp_id` ='3e8b06c6bee34bf7acde975697971d81'
   ;
   ```

2. 将数据复制到google -excel

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408051730453.png" alt="image-20240805173010101" style="zoom:50%;" />

3. 输入指令到chatgpt

   ```
   请根据下面要求在google excel 中完成
   第一行的数据作为表头,表头有:target_name,target_id,target_items
   需要将target_items 同一个列的数据，进行json解析成一个集合,将解析后的数据作为新的表头
   解析规则为:  headerName 的值作为表头,fieldValue 的值作为内容,
   最终返回处理后的excel,excel 样式需要便与查看。需要获取当前sheet，并且生成的数据也在当前sheet中
   需要将上述功能做成自定义菜单
   ```

   给出的结果：

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408051742661.png" alt="image-20240805174211616" style="zoom:50%;" />

4. 复制生成的指令到脚本中

5. 保存->运行 -> 设置权限

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408051743490.png" alt="image-20240805174344444" style="zoom: 33%;" />

6. 下载

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202408051746710.png" alt="image-20240805174649655" style="zoom: 33%;" />