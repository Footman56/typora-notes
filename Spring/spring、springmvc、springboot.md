## **一、概念**

### **1、Spring**

Spring是一个开源[容器](https://cloud.tencent.com/product/tke?from=10680)框架，可以接管web层，业务层，dao层，持久层的组件，并且可以配置各种bean,和维护bean与bean之间的关系。其核心就是控制反转(IOC),和面向切面(AOP),简单的说就是一个分层的轻量级开源框架。重点功能在于bean的处理

### **2、SpringMVC**

SpringMVC属于SpringFrameWork的后续产品，已经融合在Spring Web Flow里面。SpringMVC是一种web层mvc框架，用于替代servlet（处理|响应请求，获取表单参数，表单校验等。SpringMVC是一个MVC的开源框架，SpringMVC=struts2+spring，springMVC就相当于是Struts2加上Spring的整合。**主要处理的是web请求，接受网络请求，处理，返回对应的结果**

### **3、SpringBoot**

Springboot是一个微服务框架，延续了spring框架的核心思想IOC和AOP，简化了应用的开发和部署。Spring Boot是为了简化Spring应用的创建、运行、调试、部署等而出现的，使用它可以做到专注于Spring应用的开发，而无需过多关注XML的配置。提供了一堆依赖打包，并已经按照使用习惯解决了依赖问题--->习惯大于约定。



# spring

![Spring Boot架构_产品简介_算法服务敏捷平台_企业版](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301171539964.png)

Spring为简化我们的开发工作，封装了一系列的开箱即用的组件功能模块，包括：Spring JDBC 、Spring MVC 、Spring Security、 Spring AOP 、Spring ORM 、Spring Test等。

springmvc  只是spring 架构中web 模块的拓展

```javascript
spring mvc < spring < springboot
```

# springmvc

![在这里插入图片描述](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301171541563.jpeg)

Springmvc 重点在于处理web请求（包括http、https）

M（model）: 业务对象数据

V （view）： 页面视图

C（controller） : 控制器，业务处理

# springboot

在spring 的基础之上做了些拓展优化，减少了一些客户手动配置，使用默认的配置。并且整合一些starter 插件，并于快速使用功能

`Spring Boot`基本上是`Spring`框架的扩展，它消除了设置`Spring`应用程序所需的`XML配置`，为更快，更高效的开发生态系统铺平了道路。

`Spring Boot`中的一些特点：

1.  创建独立的`spring`应用。
2.  嵌入`Tomcat`, `Jetty` `Undertow` 而且不需要部署他们。
3.  提供的“starters” poms来简化`Maven`配置
4.  尽可能自动配置`spring`应用。
5.  提供生产指标,健壮检查和外部化配置
6.  绝对没有代码生成和`XML`配置要求。



**`Spring Boot` 对比`Spring`的一些优点包括**：

- 提供嵌入式容器支持
- 使用命令*java -jar*独立运行jar
- 在外部容器中部署时，可以选择排除依赖关系以避免潜在的jar冲突
- 部署时灵活指定配置文件的选项
- 用于集成测试的随机端口生成