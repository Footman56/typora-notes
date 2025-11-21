BOM全称是Bill Of Materials，译作材料清单。BOM本身并不是一种特殊的文件格式，而是一个普通的POM文件，只是在这个POM中，我们罗列的是一个工程的所有依赖和其对应的版本。该文件一般被其它工程使用，当其它工程引用BOM中罗列的jar包时，不用显示指定具体的版本，会自动使用BOM对应的jar版本。



# 一、用法

定义一个基础pom 文件，在pom文件中指定版本，项目依赖。（这里是定义了基础的项目版本，后续都使用这个同一的版本）

**打包方式为pom**

后续维护版本都在properties 里面定义

test-dependency / pom.xml

```xml
 <artifactId>test-dependency</artifactId>
    <groupId>com.huochai</groupId>
    <version>1.0-SNAPSHOT</version>
    <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging>

 <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <mysql.version>8.0.16</mysql.version>
</properties>

 <dependencyManagement>
        <dependencies>

            <!-- 统一依赖管理 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
   </dependencies>
</dependencyManagement>
```

项目中只有一个pom.xml文件

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202401031645750.png" alt="image-20240103164518325" style="zoom:50%;" />



在根项目下引入 

type一定要制定为pom形式，

```xml
<dependencyManagement>
    <!--dependencyManagement 方式依赖bom -->
    <dependencies>
        <dependency>
            <groupId>com.huochai</groupId>
            <artifactId>test-dependency</artifactId>
            <version>1.0-SNAPSHOT</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
    </dependencies>

</dependencyManagemen
```



 **如果项目的打包方式是pom的话，引入的时候一定要 <type>pom</type>**  



Maven的版本依赖优先顺序

1. 直接在当前工程中显示指定的版本

2. <parent/>中配置的父工程使用的版本
3.  在当前工程中通过<dependencyManagement/>引入的BOM清单中的版本，当引入的多个BOM都有对应jar包时，先引入的BOM生效
4. 上述三个地方都没配置，则启用依赖调解机制 ( **依赖的路径最短**)

