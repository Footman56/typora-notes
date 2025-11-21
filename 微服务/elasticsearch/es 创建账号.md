1. 停到 es 、kibana

2. 在elastic search.yaml 中添加配置

   ```yaml
   xpack.security.enabled: true
   discovery.type: single-node
   
   ```

3. 重启es

   ```sh
   ./bin/elasticsearch 
   ```

4. 在官网申请license 证书

   [https://register.elastic.co](https://register.elastic.co/)  ，填写后，在邮箱中获取json 证书文件

5. 修改证书内容

   ```
   "type":"basic" 替换为 "type":"platinum"    # 基础版变更为铂金版
   
   "expiry_date_in_millis":1561420799999 替换为  "expiry_date_in_millis":3107746200000# 1年变为50年
   ```

6.   获取破解版x-pack-core-6.7.0.jar

   网盘地址： https://pan.baidu.com/s/1qI6DaUKsF-ydgFYzUEwHxw   a2zv

7. 替换/Users/peilizhi/elasticsearch-6.7.0/modules/x-pack-core 中的 x-pack-core-6.7.0.jar

8. 启动x-pack 安全插件

   ```
   curl -H "Content-Type:application/json" -XPOST  http://localhost:9200/_xpack/license/start_trial\?acknowledge\=true
   ```

9. 设置默认用户密码

   ```sh
   bin/elasticsearch-setup-passwords interactive
   ```

   默认都是 asdf123

10. 生成证书

```shell
./bin/elasticsearch-certgen
```

直接一路回车就可以，

11. 导入证书

    ```
    curl -XPUT -u elastic  'http://localhost:9200/_license?acknowledge=true' -H "Content-Type: application/json" -d @peilizhi-license.json
    ```

10. kibana配置文件中添加账号密码

    ```shell
    elasticsearch.username: "elastic"
    elasticsearch.password: "123456"
    ```

    

    