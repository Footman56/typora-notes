# 一、gateway网关

```
门户作用。
网关的角色是作为一个 API 架构，用来保护、增强和控制对于 API 服务的访问。
API 网关是一个处于应用程序或服务（提供 REST API 接口服务）之前的系统，用来管理授权、访问控制和流量限制
```

## 1、职能

```
请求接入：作为所有API接口服务请求的接入点
业务聚合：作为后端业务服务的聚合点
中介策略：实现安全、验证、路由、过滤、流控
统一管理：对所有API服务和策略进行统一管理
```

## 2、分类

```
流量网关：全局流量控制、日志统计、防止	sql注入、防止Web攻击、屏蔽工具扫描、证书/加密处理
业务网关：服务级别流控、服务降级与熔断、路由与负载均衡、灰度策略、服务过滤、聚合与发现、权限验证与用户等级策略、业务规则与参数校验、多级缓存策略
```

## 3、概念

```
Route: 由一个ID、目标URI、一组断言和一组过滤器，如果断言为真，则路由匹配，目标URI会被访问。
Predicate：输入类型是ServerWebExchange.用它来匹配来自HTTP请求的内容
Filter:过滤器

如果有新的对外暴露的服务的时候，需要添加一个新的路由。来对新的URI放行
```

#  二、Oauth2

```
Oauth2的作用就是让第三方应用在用户（资源持有者）授权的情况下，通过认证服务器的认证，从而安全的在资源服务器上获得对应的用户资源的流程指导。

Third-party application: 第三方应用
Resource Owner: 资源持有者，一般就是用户自身
Authorization server: 认证服务器
Resource server: 资源服务器，即具体资源的存储方。与认证服务器是不同的逻辑节点，但是在物理上，双方是可以在一起的
User Agent: 用户代理，一般就是指的浏览器
Http Service: 服务提供者，也就是持有Resource Server的存在方。可以理解为类似QQ，或者微信这样具备用户信息的服务者。
```

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-20 18.20.38.png" style="zoom:50%;" />

```
1、第三方应用向资源持有者请求获取资源
2、资源持有者授权给予第三方应用一个许可
3、第三方应用将该许可给予认证服务器进行认证，如果认证成功，返回一个Access Token
4、第三方应用使用该access token到资源服务器处获取该access token对应的资源（也就是第一步中资源持有者自身的资源）
```

# 三、Eureka

```
服务注册与服务发现
```

## 1、背景

```
现在有A、B、C、D四个服务，它们之间会互相调用(而且IP地址很可能会发生变化)，一旦某个服务的IP地址变了，那服务中的代码要跟着变，手动维护这些静态配置(IP)非常麻烦！
```

## 2、方案

```
创建一个E服务，将A、B、C、D四个服务的信息都注册到E服务上，E服务维护这些已经注册进来的信息
A、B、C、D四个服务都可以拿到Eureka(服务E)那份注册清单。A、B、C、D四个服务互相调用不再通过具体的IP地址，而是通过服务名来调用！
```

## 3、细节

```
Eureka专门用于给其他服务注册的称为Eureka Server(服务注册中心)，其余注册到Eureka Server的服务称为Eureka Client。

服务提供者

服务注册：启动的时候会通过发送REST请求的方式将自己注册到Eureka Server上，同时带上了自身服务的一些元数据信息。
**服务续约：**在注册完服务之后，服务提供者会维护一个心跳用来持续告诉Eureka Server:  "我还活着 ” 、
服务下线：当服务实例进行正常的关闭操作时，它会触发一个服务下线的REST请求给Eureka Server, 告诉服务注册中心：“我要下线了 ”。


服务消费者

获取服务：当我们启动服务消费者的时候，它会发送一个REST请求给服务注册中心，来获取上面注册的服务清单
服务调用：服务消费者在获取服务清单后，通过服务名可以获得具体提供服务的实例名和该实例的元数据信息。在进行服务调用的时候，优先访问同处一个Zone中的服务提供方。


Eureka Server(服务注册中心)：

**失效剔除：**默认每隔一段时间（默认为60秒） 将当前清单中超时（默认为90秒）没有续约的服务剔除出去。
自我保护：。EurekaServer 在运行期间，会统计心跳失败的比例在15分钟之内是否低于85%(通常由于网络不稳定导致)。 Eureka Server会将当前的实例注册信息保护起来， 让这些实例不会过期，尽可能保护这些注册信息。

```

# 四 、流程分析

## 1、商务沟通，开通账号

>最终会给对接方: appKey,appSecret
>
>作用：在请求时携带用于验证身份

## 2、生成Token

```
在哪里生成？
		在oauth2
```

## 3、openapi接受请求

```
接受到请求后如何处理?
	通过Eureka 找到服务
```

## 4、openapi调用服务资源

# 五、问题

```apl
openapi是我们对外暴露服务的。
什么人？
	有权限的人：在请求时需要确定好是否具有权限

如何调用自身服务？
	dubbo调用，（gateway中使用dubbo泛型）


AppInfo  代表的是客户可访问的模块吗？

功能：消息推送，访问其他业务模块（使用dubbo），拦截机制（成功之后才能访问Controller）。
data-filer：处理数据权限相关的
openapi-api:dubbo中服务提供者
openapi-core:
消息推送：通过RabbitMQ接受变化的消息，使用http请求发送过去
	接受的MQ消息-》获取消息处理器-》消息处理器处理消息-》获取推送配置-》封装消息（不同的消息处理器封装不同的格式）-》生成签名-》发送消息

推送消息配置：公司的uri,appKey，消息类型
数据库：openapi
```

```apl
openapi-application（权限管理,集成oauth2）
数据库：app_management
用户，角色，权限
```

```
gateway 网关转发，统一拦截，
应用管理：具体实现
openapi:整合业务
开发：签名在param ,与考勤那边的参数有所有不同
接口设计规范（参考之前）
定义好接口（参数、响应头）校验【去掉不合理】

开发
body -requestBody
封装rpc
要有openapi自己的对象（维持不变）


分析

看v5 规范
vaild
v5/report/detail
确定一些业务校验（石飞飞）
测试用例 更新sdk @Test （黑盒）
sdk:是我们把方法写好，供第三方调用。就类似黑盒测试。

需求-》接口-》openapi开发->后台配置（路由）-》sdk-》测试
```

# 六、openApi接口规范

```api
1、请求方式 HTTPS(POST)
2、请求地址
	https://api.xinrenxinshi.com/{版本号}/{应用名}/{XXX}
3、请求Header
4、请求参数
	请求参数是openAPI自己定义的，不与rpc中定义的相关
	必含有：access_token,sign,timestamp（请求的时间）
5、body参数
6、分页处理
7、返回值
	错误码有固定格式，
8、拦截器机制就检查用户登录时权限问题

请求参数中的 appkey,sign,timestamp 经过GlobalParamFilter都被封装到ThreadLocalValue 中


6、实现
（1）、校验参数
（2）、打印参数
（3）、获取GlobalParam[经过过滤器、拦截器中，对请求的url进行了整合]
（4）、构造rpc参数
（5）、调用rpc

校验参数：
	字面值是否合理
	是否有意义：对应的对象是否存在
```

# 七、报表需求

```
openapi 需要对外提供查询报表的需求，且此报表是已经归档过的报表，分页展示
1、需要的参数
2、如何获取参数
3、如何展示结果
	每一张报表有很多字段，需要做筛选，选择最关键的
	初步设计每张表都是一个类
```

# 八、开发流程

```
1、确定好接口，参考 https://api.xinrenxinshi.com/doc/v3/page/attendance/cancelTravel_v5.html 格式
2、审核接口文档
3、在Xone/openapi开发
4、编写sdk代码 项目是：xrxs-openapi-sdk-java，更新版本打包
5、发起项目，在backend配置 项目有：openapi、gateway、eureka、nobility
6、后台添加权限（请求的路径）、添加角色、应用开通（根据超管手机号查询公司）

7、获取appSecret ，在应用管理发送短信，在booster-notify项目接受短信，
8、




```



# 九、测试账号

```
115环境
	手机号：19400270922
	公式id:294d9f20fd4b4a25a2d943c6bb5f6b9d
	appKey：appBhkiB3m4iNE0VYABux9vruNVsrEV3
	appSecret：Zo_jI_Zqs2V$FD9CQCW6
	
	
	
public void queryReportDetail(String yearmo, Integer reportType) throws ApiException{
        AttendanceReportSearchRequest reportSearchRequest=new AttendanceReportSearchRequest(access_token);
        reportSearchRequest.setReportType(reportType);
        reportSearchRequest.setYearmo(yearmo);
        reportSearchRequest.setPageNo(0);
        reportSearchRequest.setPageSize(100);
        XrxsAttendanceService.queryReportDetail(reportSearchRequest);
    }

openapi:
  appKey: appvipjBlFNcmaEl1ZnAiZDJ6tZ9dSU7
  appSecret: YMzTx0L@LyeTB2Vq0g*q
  
  url: http://api115.devtest.vip:8886
```





