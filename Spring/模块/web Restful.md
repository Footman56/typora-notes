# 一、pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
   父项目
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.5.2</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	
	
	<groupId>com.example</groupId>
	<artifactId>rest-service-initial</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>rest-service-initial</name>
	<description>Demo project for Spring Boot</description>
  
  
  
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
    
    web 模块
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

```java
// restful风格
@RestController
public class GreetingController {

    private final AtomicInteger atomicInteger = new AtomicInteger();
    private static final String TEMPLATE = "Hello,%s!";

  // get 请求
  
    @GetMapping("/greeting")
    public GreetingDO greetingDO(@RequestParam(value = "name", defaultValue = "world") String name) {
        
        // 每次请求atomicInteger 都增加1 
        return new GreetingDO(atomicInteger.incrementAndGet(), String.format(TEMPLATE, name));
    }
}

```





