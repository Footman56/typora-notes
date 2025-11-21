# 一、状态码

```
400: 服务器无法理解客户的请求，  [请求书写的问题]
403: 服务器拒绝地址请求
404：无法找到文件
405：资源被禁用[权限不足]
408:请求超时
305：要通过代理才能访问
500：服务器内部错误
502： 服务器上的错误网关，出现502错误的时候，可以清除缓存，或者刷新浏览器[项目没有启动]
302:请求的参数形式与接口中的参数形式不一致。比如：前端使用json数据传递整体，后端使用对象来获取[获取失败]。
204:该状态码代表服务器接收的请求已成功处理，但在返回的响应报文中不含实体的主体部分。
```

**有时候需要清空浏览器的缓存**



**3XX: 请求重定向问题**

```
检查接口是否正确，相符
```









# Body 

Body 的四种格式：

from-data、x-www-form-urlencoded、raw 、binary

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301091422475.png" alt="image-20220427173918543" style="zoom:50%;" />

## from-data

就是 Http 协议中的 `multipart/form-data`，表示以表单形式提交，它会将表单的数据处理为一条消息，以标签为单元，用分隔符分开。既可以上传键值对，也可以上传文件，也支持同时传 文件和键值对

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301091422641.png" alt="image-20220427174707781" style="zoom:50%;" />

## x-www-from-urlencoded

除了content-type 有所区别外，其余的相差不大，

理论上，表单提交的参数以 `key-value` 键值对的形式被封装到请求体中：name=111&useWay=222

![image-20220427174902831](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301091422650.png)

## raw

可以选择 不同的类型,不同的类型对应不同的请求头

+ text :  text/plain
+ javascript: application/javascript
+ json:application/json
+ html: text/html
+ application/xml :application/xml

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301091131209.png" alt="image-20220427182754010" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220427183134047.png" alt="image-20220427183134047" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220427183109865.png" alt="image-20220427183109865" style="zoom:50%;" />



## binary

`binary` 通常用来上传文件，由于没有键值，所以，一次只能上传一个文件（一般用的不多）

