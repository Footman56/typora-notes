# 破解（2021.3.3）

1. 下载插件

​	https://pan.baidu.com/s/1K5uqbUysFWXSKIBUmbGyBQ 提取码: 452s，

2. 移动文件到/Users/用户名下

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202401041612042.png" alt="image-20220430171952837" style="zoom:50%;" />

3. 修改idea.vmoptions

   进入应用程序，选择idea->右键包管理-> 进入bin包下->修改 idea.vmoptions

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202401130054963.png" alt="image-20220430172226315" style="zoom:50%;" />

4. 启动idea

5. 查看激活日期

6. 修改激活日期(默认到2099年)

   进入/Users/peilizhi/ja-netfilter-all/config-jetbrains 下编辑 mymap.conf

   ```tex
   [MyMap]
   EQUAL,licenseeName->MyMap
   EQUAL,gracePeriodDays->100000
   EQUAL,paidUpTo->2099-12-31%
   ```

   7. 如果激活一段时间之后自动失效的话，可以修改 激活日期，之后选择输入激活码
   
      ```
      4W9NP3KV9E-eyJsaWNlbnNlSWQiOiI0VzlOUDNLVjlFIiwibGljZW5zZWVOYW1lIjoic2NyaXAgd2FuZSIsImFzc2lnbmVlTmFtZSI6IiIsImFzc2lnbmVlRW1haWwiOiIiLCJsaWNlbnNlUmVzdHJpY3Rpb24iOiIiLCJjaGVja0NvbmN1cnJlbnRVc2UiOmZhbHNlLCJwcm9kdWN0cyI6W3siY29kZSI6IklJIiwiZmFsbGJhY2tEYXRlIjoiMjAyMy0wMS0yNCIsInBhaWRVcFRvIjoiMjAyMy0wMS0yNCIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiUERCIiwiZmFsbGJhY2tEYXRlIjoiMjAyMy0wMS0yNCIsInBhaWRVcFRvIjoiMjAyMy0wMS0yNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQV1MiLCJmYWxsYmFja0RhdGUiOiIyMDIzLTAxLTI0IiwicGFpZFVwVG8iOiIyMDIzLTAxLTI0IiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBHTyIsImZhbGxiYWNrRGF0ZSI6IjIwMjMtMDEtMjQiLCJwYWlkVXBUbyI6IjIwMjMtMDEtMjQiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUFBTIiwiZmFsbGJhY2tEYXRlIjoiMjAyMy0wMS0yNCIsInBhaWRVcFRvIjoiMjAyMy0wMS0yNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQUEMiLCJmYWxsYmFja0RhdGUiOiIyMDIzLTAxLTI0IiwicGFpZFVwVG8iOiIyMDIzLTAxLTI0IiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBSQiIsImZhbGxiYWNrRGF0ZSI6IjIwMjMtMDEtMjQiLCJwYWlkVXBUbyI6IjIwMjMtMDEtMjQiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUFNXIiwiZmFsbGJhY2tEYXRlIjoiMjAyMy0wMS0yNCIsInBhaWRVcFRvIjoiMjAyMy0wMS0yNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQU0kiLCJmYWxsYmFja0RhdGUiOiIyMDIzLTAxLTI0IiwicGFpZFVwVG8iOiIyMDIzLTAxLTI0IiwiZXh0ZW5kZWQiOnRydWV9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
      ```
   
      
      
      # 破解 （2023.3）
      
      1. 获取破解包，存入没有中文的路径下
      
         ```
         /Users/peilizhi/IdeaProjects/jetbra
         ```
      
      2. 进入 /jetbra/scripts
      
      3. 执行下面命令，需要授予执行权限
      
         ```
         sudo bash install.sh
         ```
      
         **此时需要关闭电脑的 SIP**
      
         提示done 说明激活成功
      
         ```
         终端输入 csrutil status 即可看到 SIP 的状态是 disable 还是 enable 。
         
         
         关闭或者开启sip
         重启 Mac ，按住 Command + R 直到屏幕上出现苹果的标志和进度条 ，进入 Recovery 模式 ；
         在屏幕上方的工具栏找到并打开终端，输入命令 csrutil disable
         关掉终端，重启 Mac ；
         重启以后可以在终端中查看状态确认 。
         ```
      
      4. 重启idea 或者需要重启电脑
      5. 填入如下激活码
      
      ```
      6G5NXCPJZB-eyJsaWNlbnNlSWQiOiI2RzVOWENQSlpCIiwibGljZW5zZWVOYW1lIjoic2lnbnVwIHNjb290ZXIiLCJhc3NpZ25lZU5hbWUiOiIiLCJhc3NpZ25lZUVtYWlsIjoiIiwibGljZW5zZVJlc3RyaWN0aW9uIjoiIiwiY2hlY2tDb25jdXJyZW50VXNlIjpmYWxzZSwicHJvZHVjdHMiOlt7ImNvZGUiOiJQU0kiLCJmYWxsYmFja0RhdGUiOiIyMDI1LTA4LTAxIiwicGFpZFVwVG8iOiIyMDI1LTA4LTAxIiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBEQiIsImZhbGxiYWNrRGF0ZSI6IjIwMjUtMDgtMDEiLCJwYWlkVXBUbyI6IjIwMjUtMDgtMDEiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiSUkiLCJmYWxsYmFja0RhdGUiOiIyMDI1LTA4LTAxIiwicGFpZFVwVG8iOiIyMDI1LTA4LTAxIiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJQUEMiLCJmYWxsYmFja0RhdGUiOiIyMDI1LTA4LTAxIiwicGFpZFVwVG8iOiIyMDI1LTA4LTAxIiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBHTyIsImZhbGxiYWNrRGF0ZSI6IjIwMjUtMDgtMDEiLCJwYWlkVXBUbyI6IjIwMjUtMDgtMDEiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUFNXIiwiZmFsbGJhY2tEYXRlIjoiMjAyNS0wOC0wMSIsInBhaWRVcFRvIjoiMjAyNS0wOC0wMSIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQV1MiLCJmYWxsYmFja0RhdGUiOiIyMDI1LTA4LTAxIiwicGFpZFVwVG8iOiIyMDI1LTA4LTAxIiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBQUyIsImZhbGxiYWNrRGF0ZSI6IjIwMjUtMDgtMDEiLCJwYWlkVXBUbyI6IjIwMjUtMDgtMDEiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUFJCIiwiZmFsbGJhY2tEYXRlIjoiMjAyNS0wOC0wMSIsInBhaWRVcFRvIjoiMjAyNS0wOC0wMSIsImV4dGVuZGVkIjp0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
      ```



# 2024.3 

1. 下载插件

   https://pan.baidu.com/s/1jD6H2SPXJ00khP18K1Tp9w?pwd=cmvb  提取码: cmvb 

2. 在`/Users/peilizhi/jebra-2024-3/scripts` 下执行install.sh. 
3. 激活码

```
FV8EM46DQYC5AW9-eyJsaWNlbnNlSWQiOiJGVjhFTTQ2RFFZQzVBVzkiLCJsaWNlbnNlZU5hbWUiOiJtZW5vcmFoIHBhcmFwZXQiLCJsaWNlbnNlZVR5cGUiOiJQRVJTT05BTCIsImFzc2lnbmVlTmFtZSI6IiIsImFzc2lnbmVlRW1haWwiOiIiLCJsaWNlbnNlUmVzdHJpY3Rpb24iOiIiLCJjaGVja0NvbmN1cnJlbnRVc2UiOmZhbHNlLCJwcm9kdWN0cyI6W3siY29kZSI6IlBDV01QIiwiZmFsbGJhY2tEYXRlIjoiMjAyNi0wOS0xNCIsInBhaWRVcFRvIjoiMjAyNi0wOS0xNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQUlIiLCJmYWxsYmFja0RhdGUiOiIyMDI2LTA5LTE0IiwicGFpZFVwVG8iOiIyMDI2LTA5LTE0IiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBEQiIsImZhbGxiYWNrRGF0ZSI6IjIwMjYtMDktMTQiLCJwYWlkVXBUbyI6IjIwMjYtMDktMTQiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUFNJIiwiZmFsbGJhY2tEYXRlIjoiMjAyNi0wOS0xNCIsInBhaWRVcFRvIjoiMjAyNi0wOS0xNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJJSSIsImZhbGxiYWNrRGF0ZSI6IjIwMjYtMDktMTQiLCJwYWlkVXBUbyI6IjIwMjYtMDktMTQiLCJleHRlbmRlZCI6ZmFsc2V9XSwibWV0YWRhdGEiOiIwMjIwMjQwNzAyUFNBWDAwMDAwNVgiLCJoYXNoIjoiMTIzNDU2NzgvMC01NDE4MTY2MjkiLCJncmFjZVBlcmlvZERheXMiOjcsImF1dG9Qcm9sb25nYXRlZCI6ZmFsc2UsImlzQXV0b1Byb2xvbmdhdGVkIjpmYWxzZSwidHJpYWwiOmZhbHNlLCJhaUFsbG93ZWQiOnRydWV9-cH8qBniG31nF8954hthJJuzF6Fk4RQ9T03IfNxsFkuxUcwaAGHKOcRudvBZIAbLwDDFw63q2QZsnpwthBb/6IqBYnJrjRC83a8wkYKGN8HqAyDtbqdLOxLjcaiAiSKzektfAXn6nGNfDeygcFr/WzMfI0on/43ByuwxmSrjwYc4M8SCR0nkDAi0XwXNnFp3vSp0gJQd+lJtkSHO2KR7gUyNDZOPVduljJGbdLJUK6UcUjrlAd6NrTNqpu5P7hcYRaNzjoJ0KeIx5k9KmMCdcfQBia/zSHUbwZiecFsyjxqtIU0C3TDaX1OM4siJVDpgrXi+ocY86hiiYE79ygJf2IA==-MIIETDCCAjSgAwIBAgIBDTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTIwMTAxOTA5MDU1M1oXDTIyMTAyMTA5MDU1M1owHzEdMBsGA1UEAwwUcHJvZDJ5LWZyb20tMjAyMDEwMTkwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCUlaUFc1wf+CfY9wzFWEL2euKQ5nswqb57V8QZG7d7RoR6rwYUIXseTOAFq210oMEe++LCjzKDuqwDfsyhgDNTgZBPAaC4vUU2oy+XR+Fq8nBixWIsH668HeOnRK6RRhsr0rJzRB95aZ3EAPzBuQ2qPaNGm17pAX0Rd6MPRgjp75IWwI9eA6aMEdPQEVN7uyOtM5zSsjoj79Lbu1fjShOnQZuJcsV8tqnayeFkNzv2LTOlofU/Tbx502Ro073gGjoeRzNvrynAP03pL486P3KCAyiNPhDs2z8/COMrxRlZW5mfzo0xsK0dQGNH3UoG/9RVwHG4eS8LFpMTR9oetHZBAgMBAAGjgZkwgZYwCQYDVR0TBAIwADAdBgNVHQ4EFgQUJNoRIpb1hUHAk0foMSNM9MCEAv8wSAYDVR0jBEEwP4AUo562SGdCEjZBvW3gubSgUouX8bOhHKQaMBgxFjAUBgNVBAMMDUpldFByb2ZpbGUgQ0GCCQDSbLGDsoN54TATBgNVHSUEDDAKBggrBgEFBQcDATALBgNVHQ8EBAMCBaAwDQYJKoZIhvcNAQELBQADggIBABKaDfYJk51mtYwUFK8xqhiZaYPd30TlmCmSAaGJ0eBpvkVeqA2jGYhAQRqFiAlFC63JKvWvRZO1iRuWCEfUMkdqQ9VQPXziE/BlsOIgrL6RlJfuFcEZ8TK3syIfIGQZNCxYhLLUuet2HE6LJYPQ5c0jH4kDooRpcVZ4rBxNwddpctUO2te9UU5/FjhioZQsPvd92qOTsV+8Cyl2fvNhNKD1Uu9ff5AkVIQn4JU23ozdB/R5oUlebwaTE6WZNBs+TA/qPj+5/we9NH71WRB0hqUoLI2AKKyiPw++FtN4Su1vsdDlrAzDj9ILjpjJKA1ImuVcG329/WTYIKysZ1CWK3zATg9BeCUPAV1pQy8ToXOq+RSYen6winZ2OO93eyHv2Iw5kbn1dqfBw1BuTE29V2FJKicJSu8iEOpfoafwJISXmz1wnnWL3V/0NxTulfWsXugOoLfv0ZIBP1xH9kmf22jjQ2JiHhQZP7ZDsreRrOeIQ/c4yR8IQvMLfC0WKQqrHu5ZzXTH4NO3CwGWSlTY74kE91zXB5mwWAx1jig+UXYc2w4RkVhy0//lOmVya/PEepuuTTI4+UJwC7qbVlh5zfhj8oTNUXgN0AOc+Q0/WFPl1aw5VV/VrO8FCoB15lFVlpKaQ1Yh+DVU8ke+rt9Th0BCHXe0uZOEmH0nOnH/0onD
```



# 安装

# 配置

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426204553320.png" alt="image-20220426204553320" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426204607540.png" alt="image-20220426204607540" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426204855985.png" alt="image-20220426204855985" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426205014191.png" alt="image-20220426205014191" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426205027389.png" alt="image-20220426205027389" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426205046491.png" alt="image-20220426205046491" style="zoom:50%;" />



<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426205334230.png" alt="image-20220426205334230" style="zoom:50%;" />

​	class

```
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end

/**
 *
 *@author peilizhi 
 *@date ${DATE} ${TIME}
 **/
public class ${NAME} {
}

```

interface

```
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end

/**
 *
 *@author peilizhi 
 *@date ${DATE} ${TIME}
 **/
public interface ${NAME} {
}

```

enum

```
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end

/**
 *
 *@author peilizhi 
 *@date ${DATE} ${TIME}
 **/
public enum ${NAME} {
}

```

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426205524898.png" alt="image-20220426205524898" style="zoom:50%;" />

Maven

```
https://repo.maven.apache.org/maven2	Remote	Never	UPDATING
http://maven.qjyd.com:8081/nexus/content/groups/public	Remote	Never	IDLE
/Users/peilizhi/apache-maven-3.6.3/repository	Local	2021/6/21	IDLE
```



<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426210439241.png" alt="image-20220426210439241" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426210504029.png" alt="image-20220426210504029" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426210524681.png" alt="image-20220426210524681" style="zoom:50%;" />

```
https://repo1.maven.org/maven2
https://repo1.maven.org/maven2
https://repository.jboss.org/nexus/content/repositories/public/
http://maven.qjyd.com:8081/nexus/content/groups/public


https://oss.sonatype.org/service/local/
https://repo.jfrog.org/artifactory/api/
https://repository.jboss.org/nexus/service/local/
https://jcenter.bintray.com
```



<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426210544499.png" alt="image-20220426210544499" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426210603122.png" alt="image-20220426210603122" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426210721065.png" alt="image-20220426210721065" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426210749450.png" alt="image-20220426210749450" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426210819064.png" alt="image-20220426210819064" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426210833737.png" alt="image-20220426210833737" style="zoom:50%;" />



Yapi

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426211034854.png" alt="image-20220426211034854" style="zoom:50%;" />

```
http://yapi.xinrenxinshi.vip
007c92a485e99ffe2a703c388361efc2864ee255382dc548d3ed6429be0e8519
depart_relation
```



Application

```
vm:-Dspring.config.location=classpath:application-local120.properties -Ddubbo.resolve.file=/Users/peilizhi/IdeaProjects/appraisal-points/appraisal-webapp/target/classes/dubbo-direct.properties

```

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426212030300.png" alt="image-20220426212030300" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220426212056300.png" alt="image-20220426212056300" style="zoom:50%;" />

```
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8879
```

# 热部署

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220429120327291.png" alt="image-20220429120327291" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220429120349115.png" alt="image-20220429120349115" style="zoom:50%;" />

Frame : 页面失去焦点的时候部署

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220429120444222.png" alt="image-20220429120444222" style="zoom:50%;" />

更新事件

热部署只能对方法内修改的有所感知，不能感知到方法上注解的变化





# 插件

Alibaba Java Coding Guidelines plugin 

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202401122317203.png" alt="image-20240112231703508" style="zoom:50%;" />

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202401122318626.png" alt="image-20240112231811582" style="zoom:50%;" />

## Maven Helper

作用：检查pom 文件的冲突。

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20211220144151664.png" alt="image-20211220144151664" style="zoom:50%;" />

使用：下载后，打开pom 文件，在下栏会展示 

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20211220144540092.png" alt="image-20211220144540092" style="zoom:50%;" />

排除有冲突的依赖：

1. 点击j a r ,右键标红的版本，点击排查

![image-20211220144610459](/Users/peilizhi/Library/Application Support/typora-user-images/image-20211220144610459.png)

# 父子Maven 工程

1、先创建项目， file->new -> project -> multi-module

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220514154919872.png" alt="image-20220514154919872" style="zoom:50%;" /> 

2、依次创建根模块、子模块，子模块中指明父模块，父模块中有子模块的定义

父模块

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.7</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.huochai</groupId>
    <artifactId>consumer</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
  
     <!--子模块-->
    <modules>
        <module>consumer-client</module>
        <module>consumer-webapp</module>
    </modules>

```

子模块

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!--    声明父工程-->
    <parent>
        <artifactId>consumer</artifactId>
        <groupId>com.huochai</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>


    <artifactId>consumer-webapp</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>consumer-webapp</name>
    <description>consumer-webapp</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
```



3、jar包原则

在父模块中定义，在子模块中使用

父模块通过 <properties> 标签来统一声明jar的版本, <dependencyManagement> 来定义依赖信息，子模块就只需要描述group ID和 artifact  ID信息，其余信息从父模块这里获取默认值



4、 书签使用

1. 添加书签

   fn + option + f3 添加助记标签

2. 定位书签 control +  助记标签 标签数字



